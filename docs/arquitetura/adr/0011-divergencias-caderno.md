# ADR 0011 — Registro de divergências e interpretações em relação ao Caderno B2B

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar em PR com o Tech Lead e mais um integrante) |
| Data | 26/10/2026 · revisado em 03/11/2026 |
| Decisores | Tech Lead, PO |
| Consultados | Tutor e monitor (perguntas abertas na seção 4) |
| Relacionados | ADR 0002, ADR 0003, ADR 0006, ADR 0007, ADR 0008, [ADR 0010](0010-plano-custo-zero.md) (Plano Custo Zero), [Modelo de dados](../dados.md), feedback do Checkpoint 1 (itens 3 e 7) |

## 1. Contexto

O feedback do Checkpoint 1 estabeleceu a regra: **onde o Caderno ou a ficha definem um valor, ele vale; para divergir, é preciso registrar em ADR**, como fizemos com o CPF na ADR 0008. Esta ADR é o **índice único** dessas divergências, inclusive das leituras que fizemos do **Plano Custo Zero** do monitor (ADR 0010). Quem revisa um PR, ou o tutor num gate, encontra aqui, em um lugar só, tudo o que o Habilita faz diferente do Caderno, além das leituras que fizemos quando o texto do Caderno não fixa o detalhe.

## 2. Opções consideradas

1. Registrar cada divergência só na ADR do tema: a informação fica espalhada e é fácil esquecer uma.
2. **Uma ADR índice que aponta para a ADR de cada tema**: uma tabela única para conferir, com o detalhe na ADR específica.

## 3. Decisão

Adotamos a opção 2. **Toda divergência ou interpretação nova entra nesta tabela no mesmo PR que a introduz** (regra da revisão de PR do Tech Lead).

### 3.1 Divergências mantidas (decisão consciente, diferente do Caderno ou da ficha)

| # | Tema | Caderno / ficha | O que o Habilita faz | Motivo | Onde |
|---|---|---|---|---|---|
| D1 | Identificação na portaria | Consulta de liberação (a ficha cita QR) | Consulta por **CPF no corpo de um POST**, guardando só HMAC e versão mascarada | CPF fora de URLs, logs e histórico (LGPD); busca em uma leitura | [ADR 0008](0008-consulta-liberacao-cpf.md) |
| D2 | Nome dos papéis | "prestadora" e "portaria" | Identificadores `terceirizada` e `porteiro` | Só o nome muda; permissões iguais às da ficha. Evita renomear o código e o contrato já existentes | [ADR 0002](0002-tenant-autorizacao.md) |
| D3 | Miniaturas | "miniaturas" | Miniatura **só para JPG**; PDF mostra ícone | Renderizar PDF exige biblioteca nativa na imagem da API (tamanho, cold start, superfície de ataque) | [ADR 0006](0006-armazenamento-de-documentos.md) §3.3 |
| D4 | Infraestrutura em nuvem | Cloud Run, Cloud Storage, Cloud Scheduler, Workload Identity, Secret Manager, Cloud Monitoring | Render, Supabase Storage, GitHub Actions, chave em variável protegida, UptimeRobot | Sem voucher do GCP e sem cartão; plano definido pelo monitor. As 6 exceções ficam listadas e justificadas na ADR 0010 | [ADR 0010](0010-plano-custo-zero.md) §6 |

### 3.2 Interpretações (o Caderno não fixa o detalhe; adotamos a leitura mais segura)

| # | Tema | Texto do Caderno / ficha | Nossa leitura | Por quê | Onde |
|---|---|---|---|---|---|
| I1 | Limite de upload | "até 5 MB" | **5.000.000 bytes** (5.000.000 aceito; 5.000.001 → 413) | Atende o Caderno tanto se "MB" for decimal quanto se for MiB | [ADR 0006](0006-armazenamento-de-documentos.md) §3.1 |
| I2 | Expurgo de 90 dias | Caderno: "lifecycle de 90 dias"; Plano Custo Zero: "o mesmo workflow apaga os arquivos com mais de 90 dias" | Expurgo **90 dias depois de o arquivo deixar de ser necessário** (rejeitado, substituído, vencido sem vínculo), feito pelo **job diário** a partir de `arquivo.expurgoPrevistoEm`, e não 90 dias após o envio | Contar do envio apagaria um ASO ainda válido (validade típica de 12 meses) e quebraria o semáforo e a cena de reuso da demo | [ADR 0006](0006-armazenamento-de-documentos.md) §3.4, [ADR 0007](0007-recalculo-idempotente.md) |
| I3 | Recurso de outro tenant | O rascunho de escopo citava 403 | **404** | Não revela que o recurso existe | [ADR 0002](0002-tenant-autorizacao.md) |
| I5 | Envio de arquivo | Plano Custo Zero: "a API gera a URL assinada curta para envio e para download" | **Upload multipart pela API**; URL assinada curta (300 s) só para **download** | A API valida tamanho e assinatura de bytes **antes** de gravar e a chave do Supabase fica só no servidor; atende o objetivo do plano (bucket privado, chave fora do navegador) | [ADR 0006](0006-armazenamento-de-documentos.md) §3.1 |
| I6 | "Rules deny-all" do Storage | Caderno: Security Rules deny-all (Firestore e Storage) | Firestore: Security Rules deny-all. Supabase: bucket privado **sem nenhuma política** para `anon`/`authenticated`; o teste do H19.CA4 passa a provar que a chave `anon` não lê nem grava | O Supabase não usa Security Rules do Firebase; ausência de política é o equivalente a deny-all | [ADR 0006](0006-armazenamento-de-documentos.md) §3.5 |
| I7 | Rota do job diário | Plano Custo Zero cita `POST /jobs/recalcular` | **`POST /internal/jobs/recalculo-vencimentos`**, com `Authorization: Bearer <JOB_TOKEN>` | A rota já estava no contrato; `/internal/` deixa claro que não é rota de usuário | [ADR 0007](0007-recalculo-idempotente.md) |
| I8 | Região de Render e Supabase | Caderno: região dos EUA (GCP) | Firestore em `us-central1`; Render e Supabase numa região dos EUA escolhida pelo DevOps ao criar os serviços: `[preencher região do Render]`, `[preencher região do Supabase]` | Mantém tudo nos EUA, como no Caderno, e perto do Firestore | [ADR 0003](0003-ci-cd.md) §3.2 |
| I4 | Caminho da matriz de rastreabilidade | `/docs/rastreabilidade.md` | A página existe nesse caminho e aponta para `docs/testes/matriz.md`, onde o QA mantém a matriz | Atende o caminho do Caderno sem duplicar conteúdo | [Matriz de rastreabilidade](../../testes/matriz.md) (a página `docs/rastreabilidade.md` entra pelo PR da PO) |

### 3.3 Alinhadas ao Caderno nesta revisão (deixaram de ser divergência em 26/10/2026)

| Tema | Antes | Agora | Onde |
|---|---|---|---|
| Papéis | 3 (`gestor`, `terceirizada`, `porteiro`) | **4**: `admin_contratante`, `analista_sst`, `terceirizada`, `porteiro`; o analista não edita a matriz | [ADR 0002](0002-tenant-autorizacao.md) |
| Região | `southamerica-east1` | **`us-central1`** no Firestore; Render e Supabase nos EUA (I8) | [ADR 0003](0003-ci-cd.md) |
| Upload | PDF/JPEG/PNG até 10 MiB | **PDF/JPG até 5 MB**, com compressão no cliente | [ADR 0006](0006-armazenamento-de-documentos.md) |
| Retenção | Sem lifecycle | **90 dias** com expurgo pelo job diário (I2) | [ADR 0006](0006-armazenamento-de-documentos.md) |
| Dashboard | 200 trabalhadores, sem coleção de contagens | **500 trabalhadores em ≤ 2 s**, lendo `contagens/{contratoId}` | [Modelo de dados](../dados.md) |
| Portaria | Devolvia pendências (tipo de documento e motivo) | **Só o resultado** e uma orientação genérica ("procure o SST") | [ADR 0008](0008-consulta-liberacao-cpf.md) |

### 3.4 Fora do escopo técnico desta ADR (responsabilidade da PO)

- **Defensibilidade:** a ficha fala em efeito de rede entre contratantes; o MVP prova o reuso dentro de um tenant. A PO registra isso no Won't, com o motivo (LGPD e isolamento) e o que seria preciso, como o consentimento da prestadora (feedback do CP1, item 4).
- Os critérios de aceite do backlog (H02, H04, H23) e os requisitos funcionais (RF03) precisam refletir as seções 3.1 a 3.3 (feedback do CP1, item 8).

## 4. Perguntas ao tutor e ao monitor

| # | Pergunta | Padrão adotado até a resposta |
|---|---|---|
| Q-T1 | "5 MB" é 5.000.000 bytes ou 5 MiB (5.242.880)? E um arquivo de exatamente 5 MB deve ser aceito? | 5.000.000 bytes, aceito no limite exato (I1). Decisão do Tech Lead confirmada em 03/11/2026 |
| Q-T2 | Os 90 dias podem contar a partir de quando o arquivo deixa de ser necessário (rejeitado, substituído, vencido ou sem vínculo), e não do envio? | Sim, pelo job diário (I2) |
| Q-T3 | Para a transferência internacional (região nos EUA), basta registrar as garantias contratuais do provedor, ou o lab espera outra base legal? | Garantias contratuais + minimização (ADR 0003 §3.1) |
| Q-T4 | Miniatura só para JPG, com ícone para PDF, atende a ficha? | Sim (D3) |
| Q-T5 | O professor aprova as 6 exceções do Plano Custo Zero (ADR 0010 §6)? Onde registramos a aprovação? | Plano adotado; aprovação registrada no cabeçalho da ADR 0010 quando vier |
| Q-T6 | Upload multipart pela API (I5), em vez de URL assinada de envio, atende o plano? | Sim (I5) |

Quando o tutor responder, a resposta é registrada aqui e a divergência é fechada ou mantida.

## 5. Consequências

- **Positivas:** uma tabela única para o tutor e para a revisão de PR; nenhuma divergência implícita.
- **Negativas:** exige disciplina para atualizar esta ADR no mesmo PR. Mitigação: item no checklist de revisão do Tech Lead e no template de PR.

## 6. Validação da decisão

- [ ] Tutor e monitor responderam Q-T2 a Q-T6 e as respostas foram registradas.
- [ ] Nenhum valor do Caderno diverge dos docs sem linha nesta ADR (revisão do Tech Lead antes de cada gate).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/10/2026 | @osamuelleite | Versão inicial (feedback do CP1, itens 3 e 7; planilha TL-06 e TL-11). |
| 1.1 | 03/11/2026 | @osamuelleite | Renumerada de 0010 para 0011 (a 0010 é o Plano Custo Zero). D4 (infraestrutura Custo Zero); I2 com expurgo pelo job; I5 a I8 (upload pela API, deny-all no Supabase, rota do job, região); Q-T1 decidida; Q-T5 e Q-T6. |
