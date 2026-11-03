# ADR 0006 — Arquivos no Cloud Storage privado, upload pela API e download por URL assinada

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend |
| Consultados | PO |
| Relacionados | H02, H12, AM-04, AM-05, [ADR 0010](0010-plano-custo-zero.md) (Plano Custo Zero), [Segurança](../../seguranca/index.md) |

> **Atenção:** os itens de hospedagem/armazenamento/agendamento/custo desta ADR estão sendo substituídos pela [ADR 0010](0010-plano-custo-zero.md) (Proposto).

## 1. Contexto

Trabalhadores enviam PDFs e imagens de documentos de SST, alguns com dado de saúde (ASO). O upload precisa ser seguro (tipo e tamanho validados, sem arquivo malicioso servido a outros usuários) e o download, restrito a quem pode ver. Firestore tem limite de tamanho por documento e cobra caro para guardar binários.

## 2. Opções consideradas

**Onde guardar**
1. Binário no Firestore (base64) — limite de tamanho por documento e custo alto. Descartado.
2. **Cloud Storage em bucket privado** — barato, feito para arquivos, regras próprias.

**Como enviar**
1. **Upload direto do navegador ao Storage com URL assinada** — não passa pela API, suporta arquivos grandes; mas a validação do conteúdo só acontece depois do arquivo gravado, exige fluxo em duas etapas e pode não se comportar igual no emulador.
2. **Upload multipart pela API** — a API valida tamanho e assinatura de bytes **antes** de gravar; um único fluxo; funciona igual no emulador. Limite: o Cloud Run aceita requisições HTTP/1 de até 32 MiB, folgado para o limite de 10 MiB do produto (valor original; o limite vigente é 5 MiB).

**Como baixar**
1. API repassa o arquivo (streaming) — controle total; consome CPU e banda da API.
2. **URL assinada de leitura com validade curta** — a API autoriza e gera a URL; o download vai direto ao Storage.

## 3. Decisão

- Arquivos em **bucket privado**, caminho `tenants/{tenantId}/trabalhadores/{trabalhadorId}/documentos/{documentoId}/{uuid}.{ext}`. O nome original do arquivo não é usado no caminho.
- **Upload multipart pela API**, limite de **5 MiB (5.242.880 bytes)**, tipos aceitos **somente PDF e JPEG**, validados pela **assinatura de bytes** (não pela extensão nem pelo `Content-Type` enviado). A API calcula **SHA-256** e grava junto do documento.
- **Download por URL assinada de leitura com validade de 5 minutos**, gerada apenas após autorização, com `Content-Disposition: attachment` para evitar execução no navegador.
- **Security Rules do Storage: deny-all.**

## 4. Consequências

- **Positivas:** validação antes de gravar; nenhum arquivo acessível sem autorização; hash permite detectar troca de arquivo e duplicidade; custo baixo.
- **Negativas:** o arquivo trafega pela API no upload (aceitável a 5 MiB); a geração de URL assinada exige que a conta de serviço tenha permissão de assinar (configurar no Checkpoint 3).
- **Risco aceito:** não há antivírus no MVP. Mitigação: tipos restritos, download como anexo, aprovação humana. Reavaliar se o produto sair do lab.
- **Risco a validar:** comportamento de URL assinada no emulador de Storage. Plano B em ambiente local: rota de download pela API habilitada só por configuração de desenvolvimento.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Arquivo malicioso sem antivírus no MVP | Tipos restritos por assinatura de bytes (somente PDF e JPEG), download como anexo, aprovação humana; reavaliar se sair do lab |
| URL assinada não funcionar no emulador de Storage | Spike na S3; plano B local: download pela API habilitado só por configuração de desenvolvimento |
| Conta de serviço sem permissão para assinar URL na nuvem | Configurar e testar no CP3 antes do primeiro deploy |

## 6. Validação da decisão

- [ ] Testes de valor-limite de tamanho (5.242.880 bytes é aceito; 5.242.881 bytes é recusado) e de arquivo com extensão falsa ([Estratégia de Testes](../../testes/estrategia.md)).
- [ ] Teste que confirma URL expirada após o prazo.
- [ ] Teste das Rules do Storage no emulador.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Arquivos no Cloud Storage privado, com download por URL assinada. Status: proposto. |
| 1.1 | 31/10/2026 | @pedrolmelo | Nota de substituição parcial pela ADR 0010 (Plano Custo Zero). |
| 1.2 | 02/11/2026 | @pedrolmelo | Ajuste do limite de upload para 5 MiB (5.242.880 bytes), restrição aos tipos PDF e JPEG e atualização dos valores de teste de borda (PO-06). |
