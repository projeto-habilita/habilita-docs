# ADR 0006 — Arquivos no Cloud Storage privado: upload pela API (5 MB, PDF/JPG), compressão no cliente, miniaturas e expurgo em 90 dias

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar em PR até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 · revisado em 26/10/2026 |
| Decisores | Tech Lead, Backend, Frontend |
| Consultados | PO, DevOps (custo), tutor (pergunta sobre o limite exato) |
| Relacionados | H02, H12, AM-04, AM-05, ADR 0003 (região), ADR 0010 (divergências), [Segurança](../../seguranca/index.md), [FinOps](../../finops/index.md) |

## 1. Contexto

As terceirizadas enviam PDFs e fotos de documentos de SST, alguns com dado de saúde (ASO). O upload precisa ser seguro (tipo e tamanho validados, nenhum arquivo malicioso servido a outro usuário) e o download, restrito a quem pode ver. O Firestore tem limite de tamanho por documento e cobra caro para guardar binários.

A ficha do Habilita e o Caderno fixam quatro regras que esta ADR precisa seguir (feedback do Checkpoint 1, item 3):

| Regra do Caderno / ficha | Versão 1.0 desta ADR |
|---|---|
| Upload **PDF/JPG até 5 MB** | PDF/JPEG/PNG até 10 MiB |
| **Lifecycle de 90 dias** no bucket, com expurgo | Sem lifecycle |
| **Compressão de imagem no cliente** | Não previa |
| **Miniaturas** | Não previa |

O **Storage é o principal custo variável do projeto**: cada megabyte a menos por arquivo e cada arquivo expurgado no prazo reduzem o gasto de armazenamento e de rede.

## 2. Opções consideradas

**Onde guardar**

1. Binário no Firestore (base64): limite por documento e custo alto. Descartado.
2. **Cloud Storage em bucket privado**: barato, feito para arquivos, regras próprias.

**Como enviar**

1. Upload direto do navegador ao Storage com URL assinada: não passa pela API, mas a validação do conteúdo só acontece depois de o arquivo estar gravado, exige fluxo em duas etapas e pode se comportar diferente no emulador.
2. **Upload multipart pela API**: a API valida tamanho e assinatura de bytes **antes** de gravar, em um fluxo só, que funciona igual no emulador. Com 5 MB, o tráfego pela API é pequeno (o Cloud Run aceita até 32 MiB por requisição HTTP/1).

**Como baixar**

1. A API repassa o arquivo (streaming): controle total, mas consome CPU e banda da API.
2. **URL assinada de leitura com validade curta**: a API autoriza e gera a URL; o download vai direto ao Storage.

**Como aplicar os 90 dias**

1. Regra de lifecycle por **idade do objeto** (`age: 90`): simples, mas apagaria um ASO **ainda válido** (validade típica de 12 meses) 90 dias depois do envio e quebraria o semáforo. Descartado.
2. **Regra por `daysSinceCustomTime: 90`**: a API marca o `customTime` do objeto quando o arquivo **deixa de ser necessário**, e o Storage apaga 90 dias depois, sem job próprio.

**Onde gerar miniaturas**

1. No navegador, enviando dois arquivos: o servidor teria de confiar numa imagem produzida pelo cliente.
2. **Na API, no upload, com Pillow**: a imagem é decodificada de verdade (validação extra) e a miniatura sai sem metadados.

## 3. Decisão

### 3.1 Upload

- **Tipos aceitos: só PDF e JPEG (JPG)**, validados pela **assinatura de bytes** (`%PDF-` e `FF D8 FF`), nunca pela extensão nem pelo `Content-Type` enviado. Qualquer outro conteúdo → **415**, com mensagem amigável: "Envie um PDF ou uma foto em JPG".
- **Limite: 5 MB = 5.000.000 bytes.** 5.000.000 bytes → 201; 5.000.001 bytes → **413**, com a mensagem "O arquivo passou de 5 MB. Tire a foto de novo ou reduza o PDF". O tamanho é verificado em *streaming*, antes de ler o arquivo inteiro.
  - Usamos o valor decimal (5.000.000), e não 5 MiB (5.242.880), porque atende o Caderno nas duas leituras de "5 MB". **Pergunta enviada ao tutor** (ver ADR 0010); se ele indicar 5 MiB, só muda a constante e os casos de BVA.
- A API calcula o **SHA-256** e grava junto do documento.
- Caminho no bucket: `tenants/{tenantId}/trabalhadores/{trabalhadorId}/documentos/{documentoId}/{uuid}.{pdf|jpg}`. O nome original do arquivo não entra no caminho.

### 3.2 Compressão no cliente (frontend)

- Quando a terceirizada escolhe uma **imagem** (foto da câmera em JPEG, PNG, WebP ou HEIC quando o navegador decodifica), o navegador **redimensiona para no máximo 2000 px no maior lado** e **recodifica em JPEG com qualidade 0,8** antes de enviar. A recodificação também remove os metadados EXIF, inclusive a localização da foto.
- Assim, um print em PNG vira JPG no cliente e é aceito; a API continua recusando PNG que chegue direto.
- **PDF não é recomprimido no cliente.** Se passar de 5 MB, a tela recusa antes do envio, com a mesma mensagem do 413.
- A validação no cliente é só experiência do usuário: **a API valida tudo de novo**.

### 3.3 Miniaturas (API)

- Para **JPEG**, a API abre a imagem com Pillow, com limite de pixels contra *decompression bomb* (`Image.MAX_IMAGE_PIXELS` ≈ 40 MP). Imagem que não decodifica → 415.
- A miniatura tem **320 px no maior lado**, em JPEG qualidade 70, sem metadados, e fica gravada ao lado do original como `{uuid}_mini.jpg`.
- **PDF não tem miniatura no MVP**: a tela mostra um ícone de PDF. Renderizar PDF exigiria uma biblioteca nativa (poppler/pdfium) na imagem da API, o que aumenta tamanho, cold start e superfície de ataque. Reavaliar depois do Demo Day.
- A miniatura é servida pela mesma rota de URL assinada, com `variante: miniatura`.

### 3.4 Retenção e expurgo (lifecycle de 90 dias)

- Bucket com a regra de lifecycle **`Delete` quando `daysSinceCustomTime ≥ 90`**, aplicada ao original e à miniatura.
- A API e o job diário (ADR 0007) gravam o `customTime` do objeto no **dia em que o arquivo deixa de ser necessário**:
  1. documento **rejeitado**;
  2. documento **substituído** por outro aprovado do mesmo tipo e do mesmo trabalhador;
  3. documento **vencido** e não usado em nenhum vínculo ativo;
  4. trabalhador **inativado** ou contrato **encerrado** sem outro vínculo ativo.
- O registro no Firestore **não é apagado**. Ele guarda `arquivo.expurgoPrevistoEm` e, depois do expurgo, `arquivo.expurgadoEm`, para manter a trilha de auditoria sem guardar o arquivo. Pedir o download de um arquivo expurgado retorna **410**.
- **Soft delete** do bucket configurado em 0 dias: o padrão de 7 dias seria custo sem uso no MVP.
- Esse prazo é o padrão da frase de LGPD da Visão até haver orientação jurídica (feedback do CP1, item 5).

### 3.5 Download e regras

- **URL assinada de leitura com validade de 5 minutos**, gerada só depois da autorização, com `Content-Disposition: attachment` para evitar execução no navegador.
- **Security Rules do Storage: deny-all.**

## 4. Consequências

- **Positivas:** validação antes de gravar; nenhum arquivo acessível sem autorização; arquivos menores (5 MB, fotos comprimidas) e expurgo automático, que reduzem o principal custo; miniaturas deixam a fila de revisão leve; fotos sem localização (LGPD).
- **Negativas:** um PDF digitalizado acima de 5 MB exige que a terceirizada reduza o arquivo fora do sistema; PDFs não têm miniatura; o `customTime` precisa ser marcado em todos os pontos listados em 3.4, o que exige teste.
- **Risco aceito:** não há antivírus no MVP. Mitigação: só dois tipos, decodificação real das imagens, download como anexo e aprovação humana. Reavaliar se o produto sair do lab.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Arquivo malicioso sem antivírus no MVP | Só PDF/JPG por assinatura de bytes; imagem decodificada pelo Pillow; download como anexo; aprovação humana |
| *Decompression bomb* em imagem | Limite de pixels no Pillow; tamanho de 5 MB verificado antes |
| `customTime` não marcado e arquivo guardado para sempre | Teste por gatilho (3.4) e relatório semanal de objetos sem `customTime` cujo documento não está em uso |
| Expurgo de arquivo ainda necessário | O `customTime` só é marcado nos gatilhos de 3.4; teste de que um documento aprovado e válido nunca recebe `customTime` |
| URL assinada não funcionar no emulador de Storage | Spike na S3; plano B local: download pela API habilitado só por configuração de desenvolvimento |
| Conta de serviço sem permissão para assinar URL na nuvem | Configurar e testar no CP3 antes do primeiro deploy |

## 6. Validação da decisão

- [ ] BVA de tamanho: 4.999.999 e 5.000.000 bytes → 201; 5.000.001 bytes → 413 com mensagem amigável.
- [ ] EP de tipo: PDF → 201; JPG → 201; PNG enviado direto → 415; `.exe` renomeado para `.pdf` → 415; JPG corrompido → 415.
- [ ] Teste do frontend: imagem de 4000 px é enviada com no máximo 2000 px, em JPEG e sem EXIF.
- [ ] Miniatura gerada para JPG (320 px, sem EXIF); PDF sem miniatura.
- [ ] `customTime` marcado em cada gatilho de 3.4 e nunca em documento aprovado e válido.
- [ ] Regra de lifecycle conferida no bucket (print em [FinOps](../../finops/index.md)) no CP3.
- [ ] Teste que confirma URL expirada após o prazo.
- [ ] Teste das Rules do Storage no emulador (leitura e escrita negadas).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Arquivos no Cloud Storage privado, com download por URL assinada. Status: proposto. |
| 2.0 | 26/10/2026 | @osamuelleite | Feedback do CP1 (itens 3 e 7): limite de 5 MB e só PDF/JPG; compressão de imagem no cliente; miniaturas na API; lifecycle de 90 dias por `customTime`; soft delete em 0. Status: proposto, para aprovar até o CP2. |
