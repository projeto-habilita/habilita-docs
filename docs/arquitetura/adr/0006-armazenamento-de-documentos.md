# ADR 0006 — Arquivos no Supabase Storage privado: upload pela API (5 MB, PDF/JPG), compressão no cliente, miniaturas e expurgo em 90 dias

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar em PR junto com a ADR 0010) |
| Data | 14/10/2026 · revisado em 26/10/2026 e 03/11/2026 |
| Decisores | Tech Lead, Backend, Frontend |
| Consultados | PO, DevOps (custo e plano gratuito), monitor (Plano Custo Zero) |
| Relacionados | H02, H12, AM-04, AM-05, [ADR 0007](0007-recalculo-idempotente.md) (job diário), [ADR 0010](0010-plano-custo-zero.md) (Plano Custo Zero), [ADR 0011](0011-divergencias-caderno.md) (divergências e interpretações), [Segurança](../../seguranca/index.md), [FinOps](../../finops/index.md) |

> **Versão 3.0:** com a [ADR 0010](0010-plano-custo-zero.md), o armazenamento sai do Cloud Storage e vai para o **Supabase Storage** no plano gratuito, como define o documento do monitor "Plano Custo Zero — sem o voucher do GCP" (14/10/2026). As regras de upload, compressão, miniaturas e retenção continuam as da versão 2.0. Muda só **onde** o arquivo fica e **quem** apaga depois de 90 dias (o job diário, porque o Supabase não tem lifecycle por data de expurgo).

## 1. Contexto

As terceirizadas enviam PDFs e fotos de documentos de SST, alguns com dado de saúde (ASO). O upload precisa ser seguro (tipo e tamanho validados, nenhum arquivo malicioso servido a outro usuário) e o download, restrito a quem pode ver. O Firestore tem limite de tamanho por documento e não é feito para guardar binários.

A ficha do Habilita, o Caderno e o Plano Custo Zero do monitor fixam as regras que esta ADR precisa seguir:

| Regra | Fonte |
|---|---|
| Upload **PDF/JPG até 5 MB** | Ficha do Habilita / Caderno; repetido no Plano Custo Zero |
| **Expurgo em 90 dias** | Caderno; no Plano Custo Zero, "o mesmo workflow [do job diário] apaga os arquivos" |
| **Compressão de imagem no cliente** e **miniaturas** | Caderno; Plano Custo Zero ("comprima as imagens no cliente") |
| **Bucket privado** no Supabase Storage; **URL assinada curta** gerada pela API; o navegador **nunca** recebe a chave do Supabase | Plano Custo Zero |

O plano gratuito do Supabase tem **1 GB armazenado** e 5 GB de download por mês (limites conferidos pelo monitor em 14/10/2026). Cada megabyte a menos por arquivo e cada arquivo expurgado no prazo mantêm o projeto dentro da cota.

## 2. Opções consideradas

**Onde guardar**

1. Binário no Firestore (base64): limite por documento e cota de 1 GiB dividida com os dados. Descartado.
2. Cloud Storage do Firebase: desde setembro de 2024 exige o plano Blaze, com cartão. Descartado pela ADR 0010.
3. **Supabase Storage, bucket privado**: gratuito sem cartão, com URL assinada e limite de tamanho e tipo por bucket.

**Como enviar**

1. URL assinada de **envio**, com o navegador subindo direto ao Supabase (o Plano Custo Zero cita "URL assinada curta para envio e para download"): não passa pela API, mas o conteúdo só é validado depois de gravado, exige duas etapas (gerar URL e confirmar) e um job para apagar envios não confirmados.
2. **Upload multipart pela API**: a API valida tamanho e assinatura de bytes **antes** de gravar, em um fluxo só, e grava com a chave de serviço, que fica só no servidor. Com 5 MB, o tráfego pela API é pequeno.

**Como baixar**

1. A API repassa o arquivo (streaming): controle total, mas consome CPU e banda do Render gratuito.
2. **URL assinada de leitura com validade curta**: a API autoriza e gera a URL; o download vai direto ao Supabase.

**Como aplicar os 90 dias**

1. Apagar por **idade do objeto** (90 dias desde o envio): simples, mas apagaria um ASO **ainda válido** (validade típica de 12 meses) e quebraria o semáforo e a cena de reuso da demo. Descartado.
2. **Data de expurgo no Firestore + job diário**: a API grava `arquivo.expurgoPrevistoEm = hoje + 90 dias` quando o arquivo **deixa de ser necessário**, e o job diário (ADR 0007) apaga os objetos cuja data chegou.

**Onde gerar miniaturas**

1. No navegador, enviando dois arquivos: o servidor teria de confiar numa imagem produzida pelo cliente.
2. **Na API, no upload, com Pillow**: a imagem é decodificada de verdade (validação extra) e a miniatura sai sem metadados.

## 3. Decisão

### 3.1 Upload (multipart pela API)

- **Tipos aceitos: só PDF e JPEG (JPG)**, validados pela **assinatura de bytes** (`%PDF-` e `FF D8 FF`), nunca pela extensão nem pelo `Content-Type` enviado. Qualquer outro conteúdo → **415**, com a mensagem "Envie um PDF ou uma foto em JPG".
- **Limite: 5 MB = 5.000.000 bytes** (decisão do Tech Lead de 26/10, confirmada em 03/11). 4.999.999 e 5.000.000 bytes → 201; 5.000.001 bytes → **413**, com a mensagem "O arquivo passou de 5 MB. Tire a foto de novo ou reduza o PDF". O tamanho é verificado em *streaming*, antes de ler o arquivo inteiro. O valor fica numa única constante da API e é repetido no `file_size_limit` do bucket (defesa em profundidade).
- A API calcula o **SHA-256** e grava junto do documento.
- Caminho no bucket `documentos`: `tenants/{tenantId}/trabalhadores/{trabalhadorId}/documentos/{documentoId}/{uuid}.{pdf|jpg}`. O `tenantId` vem **só da claim do token**; o nome original do arquivo não entra no caminho.
- Por que não a URL assinada de envio do Plano Custo Zero: está registrado na [ADR 0011](0011-divergencias-caderno.md) (interpretação I5). O plano é atendido no que importa: bucket privado, URL assinada curta no download e chave do Supabase fora do navegador.

### 3.2 Compressão no cliente (frontend)

- Quando a terceirizada escolhe uma **imagem** (foto da câmera em JPEG, PNG, WebP ou HEIC quando o navegador decodifica), o navegador **redimensiona para no máximo 2000 px no maior lado** e **recodifica em JPEG com qualidade 0,8** antes de enviar. A recodificação também remove os metadados EXIF, inclusive a localização da foto.
- Assim, um print em PNG vira JPG no cliente e é aceito; a API continua recusando PNG que chegue direto.
- **PDF não é recomprimido no cliente.** Se passar de 5.000.000 bytes, a tela recusa antes do envio, com a mesma mensagem do 413.
- A validação no cliente é só experiência do usuário: **a API valida tudo de novo**.

### 3.3 Miniaturas (API)

- Para **JPEG**, a API abre a imagem com Pillow, com limite de pixels contra *decompression bomb* (`Image.MAX_IMAGE_PIXELS` ≈ 40 MP). Imagem que não decodifica → 415.
- A miniatura tem **320 px no maior lado**, em JPEG qualidade 70, sem metadados, e fica gravada ao lado do original como `{uuid}_mini.jpg`.
- **PDF não tem miniatura no MVP**: a tela mostra um ícone de PDF. Renderizar PDF exigiria uma biblioteca nativa (poppler/pdfium) na imagem da API, o que aumenta tamanho, cold start e superfície de ataque. Reavaliar depois do Demo Day.
- A miniatura é servida pela mesma rota de URL assinada, com `variante: miniatura`.

### 3.4 Retenção e expurgo (90 dias depois de deixar de ser necessário)

- A API e o job diário gravam `arquivo.expurgoPrevistoEm` (data, fuso `America/Sao_Paulo`) no **dia em que o arquivo deixa de ser necessário** + 90 dias:
  1. documento **rejeitado**;
  2. documento **substituído** por outro aprovado do mesmo tipo e do mesmo trabalhador;
  3. documento **vencido** e não usado em nenhum vínculo ativo;
  4. trabalhador **inativado** ou contrato **encerrado** sem outro vínculo ativo.
- Se o documento volta a ser necessário antes do prazo (por exemplo, um novo vínculo reaproveita um documento ainda válido), o campo é limpo.
- O **job diário** (ADR 0007) lê os documentos com `expurgoPrevistoEm ≤ hoje` (consulta paginada, no máximo 50 por página), apaga o original e a miniatura no Supabase e grava `arquivo.expurgadoEm`. Apagar um objeto que já não existe conta como sucesso (idempotente).
- O registro no Firestore **não é apagado**: ele mantém a trilha de auditoria sem guardar o arquivo. Pedir o download de um arquivo expurgado retorna **410**.
- Esse prazo é o padrão da frase de LGPD da Visão até haver orientação jurídica (feedback do CP1, item 5). Como o plano do monitor fala em "arquivos com mais de 90 dias", a contagem a partir de quando o arquivo deixa de ser necessário fica registrada como interpretação na [ADR 0011](0011-divergencias-caderno.md) (I2), para confirmação com o monitor.

### 3.5 Download e acesso ao bucket

- **URL assinada de leitura com validade de 300 segundos (5 minutos)**, gerada só depois da autorização (H12.CA3), com download como anexo para evitar execução no navegador.
- Bucket `documentos` **privado** (`public = false`), com `file_size_limit = 5000000` e `allowed_mime_types = [application/pdf, image/jpeg]`.
- **Nenhuma política de acesso para `anon` ou `authenticated`** no schema `storage`: sem a chave de serviço, toda leitura e escrita é negada (equivalente ao deny-all). A chave `service_role` existe **só** nas variáveis do Render e nos secrets do GitHub (ADR 0010, regra 5); o navegador nunca recebe chave do Supabase.
- As **Security Rules do Firestore continuam deny-all**. O arquivo `storage.rules` (deny-all) continua no repositório, para o caso de voltar ao Cloud Storage (ADR 0010, seção 7).
- A API acessa o Storage por uma **porta** (`ArmazenamentoDocumentos`) com um adaptador Supabase e um em memória para testes. Trocar para Cloud Storage é trocar o adaptador.

### 3.6 Ambiente local

- O Supabase Storage roda como serviço do Docker Compose (ADR 0010, seção 3.4), com o mesmo bucket, limite e tipos. Chaves locais são as de demonstração do Supabase self-hosted, sem acesso a nada real.

## 4. Consequências

- **Positivas:** validação antes de gravar; nenhum arquivo acessível sem autorização; custo zero; arquivos menores (5 MB, fotos comprimidas) e expurgo no prazo, que mantêm o projeto no 1 GB do plano gratuito; miniaturas deixam a fila de revisão leve; fotos sem localização (LGPD); troca de provedor isolada num adaptador.
- **Negativas:** um PDF digitalizado acima de 5 MB exige que a terceirizada reduza o arquivo fora do sistema; PDFs não têm miniatura; o expurgo depende do job diário rodar (se o GitHub Actions falhar alguns dias, o arquivo fica um pouco além de 90 dias); a chave `service_role` dá acesso total ao Storage e precisa de cuidado.
- **Risco aceito:** não há antivírus no MVP. Mitigação: só dois tipos, decodificação real das imagens, download como anexo e aprovação humana. Reavaliar se o produto sair do lab.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Arquivo malicioso sem antivírus no MVP | Só PDF/JPG por assinatura de bytes; imagem decodificada pelo Pillow; download como anexo; aprovação humana |
| *Decompression bomb* em imagem | Limite de pixels no Pillow; tamanho de 5.000.000 bytes verificado antes |
| `expurgoPrevistoEm` não gravado e arquivo guardado para sempre | Teste por gatilho (3.4) e relatório semanal de objetos sem data de expurgo cujo documento não está em uso |
| Expurgo de arquivo ainda necessário | A data só é gravada nos gatilhos de 3.4; teste de que um documento aprovado e válido nunca recebe data de expurgo |
| Job diário falhar e o prazo passar | O GitHub avisa por e-mail quando um workflow agendado falha; o job seguinte apaga o atraso (idempotente) |
| Vazamento da chave `service_role` | Chave só em secrets do GitHub e variáveis do Render; gitleaks no CI; troca imediata da chave no painel em caso de suspeita (ADR 0010) |
| Supabase pausar após 7 dias sem uso | O job diário usa o Storage todos os dias (ADR 0010) |
| Cota de 1 GB estourar | Seed com no máximo 3 documentos por trabalhador; compressão no cliente; expurgo; acompanhamento em [FinOps](../../finops/index.md) |

## 6. Validação da decisão

- [ ] BVA de tamanho: 4.999.999 e 5.000.000 bytes → 201; 5.000.001 bytes → 413 com mensagem amigável (na API **e** no bucket).
- [ ] EP de tipo: PDF → 201; JPG → 201; PNG enviado direto → 415; `.exe` renomeado para `.pdf` → 415; JPG corrompido → 415.
- [ ] Teste do frontend: imagem de 4000 px é enviada com no máximo 2000 px, em JPEG e sem EXIF.
- [ ] Miniatura gerada para JPG (320 px, sem EXIF); PDF sem miniatura.
- [ ] `expurgoPrevistoEm` gravado em cada gatilho de 3.4 e nunca em documento aprovado e válido; job apaga só os vencidos e é idempotente.
- [ ] Teste que confirma URL assinada expirada depois de 300 s.
- [ ] Teste de acesso ao bucket sem a chave de serviço (com a chave `anon`): leitura e escrita negadas.
- [ ] Print do projeto Supabase no plano gratuito, sem forma de pagamento, em [FinOps](../../finops/index.md) (CP3).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Arquivos no Cloud Storage privado, com download por URL assinada. Status: proposto. |
| 1.1 | 31/10/2026 | @pedrolmelo | Nota de substituição parcial pela ADR 0010 (Plano Custo Zero). |
| 2.0 | 26/10/2026 | @osamuelleite | Feedback do CP1 (itens 3 e 7): limite de 5 MB e só PDF/JPG; compressão de imagem no cliente; miniaturas na API; lifecycle de 90 dias. |
| 3.0 | 03/11/2026 | @osamuelleite | Plano Custo Zero do monitor (ADR 0010): Supabase Storage privado no lugar do Cloud Storage; expurgo de 90 dias feito pelo job diário a partir de `expurgoPrevistoEm`; URL assinada de 300 s; bucket sem política para `anon`/`authenticated`; limite de 5.000.000 bytes confirmado. |
