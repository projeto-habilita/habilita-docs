# ADR 0011 — Registro de divergências e interpretações em relação ao Caderno B2B

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar em PR até o Checkpoint 2 — 01/11/2026) |
| Data | 26/10/2026 |
| Decisores | Tech Lead, PO |
| Consultados | Tutor (perguntas abertas na seção 4) |
| Relacionados | ADR 0002, ADR 0003, ADR 0006, ADR 0008, [Modelo de dados](../dados.md), feedback do Checkpoint 1 (itens 3 e 7) |

## 1. Contexto

O feedback do Checkpoint 1 estabeleceu a regra: **onde o Caderno ou a ficha definem um valor, ele vale; para divergir, é preciso registrar em ADR**, como fizemos com o CPF na ADR 0008. Esta ADR é o **índice único** dessas divergências. Quem revisa um PR, ou o tutor num gate, encontra aqui, em um lugar só, tudo o que o Habilita faz diferente do Caderno, além das leituras que fizemos quando o texto do Caderno não fixa o detalhe.

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

### 3.2 Interpretações (o Caderno não fixa o detalhe; adotamos a leitura mais segura)

| # | Tema | Texto do Caderno / ficha | Nossa leitura | Por quê | Onde |
|---|---|---|---|---|---|
| I1 | Limite de upload | "até 5 MB" | **5.000.000 bytes** (5.000.000 aceito; 5.000.001 → 413) | Atende o Caderno tanto se "MB" for decimal quanto se for MiB | [ADR 0006](0006-armazenamento-de-documentos.md) §3.1 |
| I2 | Lifecycle de 90 dias | "lifecycle de 90 dias no bucket, com expurgo" | Expurgo **90 dias depois de o arquivo deixar de ser necessário** (`daysSinceCustomTime`), e não 90 dias após o envio | Uma regra por idade apagaria um ASO ainda válido (validade típica de 12 meses) | [ADR 0006](0006-armazenamento-de-documentos.md) §3.4 |
| I3 | Recurso de outro tenant | O rascunho de escopo citava 403 | **404** | Não revela que o recurso existe | [ADR 0002](0002-tenant-autorizacao.md) |
| I4 | Caminho da matriz de rastreabilidade | `/docs/rastreabilidade.md` | A página existe nesse caminho e aponta para `docs/testes/matriz.md`, onde o QA mantém a matriz | Atende o caminho do Caderno sem duplicar conteúdo | [Rastreabilidade](../../rastreabilidade.md) |

### 3.3 Alinhadas ao Caderno nesta revisão (deixaram de ser divergência em 26/10/2026)

| Tema | Antes | Agora | Onde |
|---|---|---|---|
| Papéis | 3 (`gestor`, `terceirizada`, `porteiro`) | **4**: `admin_contratante`, `analista_sst`, `terceirizada`, `porteiro`; o analista não edita a matriz | [ADR 0002](0002-tenant-autorizacao.md) |
| Região | `southamerica-east1` | **`us-central1`** | [ADR 0003](0003-ci-cd.md) |
| Upload | PDF/JPEG/PNG até 10 MiB | **PDF/JPG até 5 MB**, com compressão no cliente | [ADR 0006](0006-armazenamento-de-documentos.md) |
| Retenção | Sem lifecycle | **90 dias** com expurgo | [ADR 0006](0006-armazenamento-de-documentos.md) |
| Dashboard | 200 trabalhadores, sem coleção de contagens | **500 trabalhadores em ≤ 2 s**, lendo `contagens/{contratoId}` | [Modelo de dados](../dados.md) |
| Portaria | Devolvia pendências (tipo de documento e motivo) | **Só o resultado** e uma orientação genérica ("procure o SST") | [ADR 0008](0008-consulta-liberacao-cpf.md) |

### 3.4 Fora do escopo técnico desta ADR (responsabilidade da PO)

- **Defensibilidade:** a ficha fala em efeito de rede entre contratantes; o MVP prova o reuso dentro de um tenant. A PO registra isso no Won't, com o motivo (LGPD e isolamento) e o que seria preciso, como o consentimento da prestadora (feedback do CP1, item 4).
- Os critérios de aceite do backlog (H02, H04, H23) e os requisitos funcionais (RF03) precisam refletir as seções 3.1 a 3.3 (feedback do CP1, item 8).

## 4. Perguntas ao tutor (prazo para decisão: 29/10/2026)

| # | Pergunta | Padrão adotado até a resposta |
|---|---|---|
| Q-T1 | "5 MB" é 5.000.000 bytes ou 5 MiB (5.242.880)? E um arquivo de exatamente 5 MB deve ser aceito? | 5.000.000 bytes, aceito no limite exato (I1) |
| Q-T2 | O lifecycle de 90 dias pode contar a partir de quando o arquivo deixa de ser necessário (rejeitado, substituído, vencido ou sem vínculo), e não do envio? | Sim, por `customTime` (I2) |
| Q-T3 | Para a transferência internacional (região nos EUA), basta registrar as garantias contratuais do provedor, ou o lab espera outra base legal? | Garantias contratuais + minimização (ADR 0003 §3.1) |
| Q-T4 | Miniatura só para JPG, com ícone para PDF, atende a ficha? | Sim (D3) |

Quando o tutor responder, a resposta é registrada aqui e a divergência é fechada ou mantida.

## 5. Consequências

- **Positivas:** uma tabela única para o tutor e para a revisão de PR; nenhuma divergência implícita.
- **Negativas:** exige disciplina para atualizar esta ADR no mesmo PR. Mitigação: item no checklist de revisão do Tech Lead e no template de PR.

## 6. Validação da decisão

- [ ] Tutor respondeu Q-T1 a Q-T4 e as respostas foram registradas.
- [ ] Nenhum valor do Caderno diverge dos docs sem linha nesta ADR (revisão do Tech Lead antes de cada gate).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/10/2026 | @osamuelleite | Versão inicial (feedback do CP1, itens 3 e 7; planilha TL-06 e TL-11). |
