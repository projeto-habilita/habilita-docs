# ADR 0001 — Stack principal

| Campo | Valor |
|---|---|
| Status | **Proposto** — linguagem da API decidida pela equipe em 14/10/2026 (Python + FastAPI); aprovar a ADR em PR até o Checkpoint 1 (18/10/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead + equipe de desenvolvimento |
| Consultados | PO, Tutores AI LAB / MAKERS |
| Relacionados | ADR 0002 (tenant e autorização), ADR 0003 (CI/CD), ADR 0004 (documentação), ADR 0005 (estilo arquitetural), [ADR 0010](0010-plano-custo-zero.md) (Plano Custo Zero), [Qualidade e ferramentas](../../engenharia/qualidade.md), [Segurança](../../seguranca/index.md) |

> **Atenção:** os itens de hospedagem/armazenamento/agendamento/custo desta ADR estão sendo substituídos pela [ADR 0010](0010-plano-custo-zero.md) (Proposto).

## 1. Contexto

Cada escolha de tecnologia precisa caber em três limites do ciclo: **teto de US$ 50**, **prazo de 9 semanas** e **o que a equipe já sabe**. A base comum do ciclo é **GCP + Firebase**. O frontend é React. Até a Semana 4 tudo roda localmente (Docker Compose + emuladores); o Caderno exige stack justificada em ADR.

A API roda em contêiner no Cloud Run, usa o Firebase Admin SDK (verificar tokens, Firestore, Storage) e precisa de cold start curto (`min-instances=0` e meta de p95 de leitura ≤ 800 ms, RNF04), boa ferramenta de testes, validação forte de entrada e contrato OpenAPI.

## 2. Opções consideradas

### 2.1 Linguagem e framework da API

| Critério | A · Node.js + TypeScript (Fastify ou NestJS) | **B · Python + FastAPI** | C · Java + Spring Boot |
|---|---|---|---|
| Escolha da equipe (14/10/2026) | — | **Escolhida** | — |
| Firebase Admin SDK | SDK oficial | SDK oficial | SDK oficial |
| Cold start no Cloud Run | Curto | Curto (com importações enxutas) | Mais longo por padrão (JVM) |
| Validação e tipagem de entrada | Bibliotecas de esquema (ex.: Zod) | **Pydantic v2 integrado ao framework**, com validação declarativa e tipos | Bean Validation |
| OpenAPI | Biblioteca adicional | **Gerado pelo próprio FastAPI (OpenAPI 3.1)**, permitindo comparar com o contrato | springdoc |
| Ferramentas de teste do Caderno | Disponíveis | **pytest; Schemathesis e Playwright têm versão Python** | Disponíveis |
| Tipos compartilhados com o frontend | Sim, mesma linguagem | Não; tipos do frontend gerados a partir do `openapi.yaml` | Não; gerados do OpenAPI |

### 2.2 Demais camadas

As outras camadas seguem a base obrigatória do ciclo (GCP + Firebase) e não tiveram alternativa realista dentro do orçamento; as descartadas estão listadas na seção 5.

## 3. Decisão

**API em Python 3.12 + FastAPI**, decidida pela equipe em 14/10/2026. A favor, além da escolha da equipe: validação de dados e OpenAPI nativos do framework e ferramentas de teste do Caderno disponíveis em Python.

!!! todo "Registrar no PR de aprovação"
    O motivo principal da equipe para a escolha (por exemplo, familiaridade com Python), em uma frase.

| Camada | Tecnologia | Justificativa | Detalhe |
|---|---|---|---|
| Frontend | **React + TypeScript (modo `strict`)**, empacotado com **Vite** (SPA) | React previsto no escopo; tipagem estrita no cliente | [Qualidade](../../engenharia/qualidade.md) |
| Hospedagem do frontend | **Firebase Hosting** | CDN, HTTPS, preview por PR, rewrite `/api` para o Cloud Run (mesma origem, sem CORS) | ADR 0003 |
| API | **Python 3.12 + FastAPI**, servidor ASGI **Uvicorn** | Decisão da equipe (seção 2.1) | ADR 0005 |
| Validação e tipos da API | **Pydantic v2** (modelos de entrada e saída) + **mypy** em modo estrito | Validação declarativa; bloqueio de campos extras; respostas enxutas | [Qualidade](../../engenharia/qualidade.md) |
| Configuração | **pydantic-settings** (variáveis de ambiente tipadas) | Falha na subida se faltar configuração | — |
| Acesso ao Firebase | **firebase-admin** (verificação de token, Storage) e cliente Firestore do Google Cloud | SDKs oficiais | ADR 0002 |
| Execução da API | **Cloud Run**, `min-instances=0` | Mesmo contêiner do Compose; escala a zero | ADR 0003 |
| Autenticação | **Firebase Authentication** + custom claims | Claims levam tenant e papel sem leitura extra | ADR 0002 |
| Banco | **Cloud Firestore**, tenant no caminho, Rules deny-all | Base do ciclo; isolamento estrutural | ADR 0002 |
| Arquivos | **Cloud Storage** privado + URL assinada de 5 min | Barato para binários; acesso só via API | ADR 0006 |
| Segredos | **Secret Manager** (nuvem) e `.env` local fora do Git | Chave HMAC do CPF e credenciais fora do código | ADR 0008 |
| Segurança da aplicação | Rate limit, cabeçalhos de segurança, limite de tamanho, logs sem dados pessoais | Controles por camada | [Segurança](../../seguranca/index.md) |
| Agendamento | **Cloud Scheduler** → rota interna com OIDC | Recálculo diário idempotente | ADR 0007 |
| Ambiente local | **Docker Compose** + **Firebase Emulator Suite** (Auth, Firestore, Storage) | Exigência do CP1; gasto zero até a S4 | — |
| CI/CD | **GitHub Actions** + **Workload Identity Federation** | Sem chave JSON permanente | ADR 0003 |
| Imagens | **Artifact Registry**, tag = SHA, retenção das últimas 5 | Exigência do CP3 | ADR 0003 |
| Contrato da API | **OpenAPI 3.1** em `openapi.yaml` na raiz (contrato primeiro) | Fonte única; o FastAPI precisa estar conforme | [Qualidade](../../engenharia/qualidade.md) |
| Testes e qualidade | pytest, Schemathesis, Playwright, Ruff, mypy, Vitest e outros | Ferramentas e metas por nível | [Qualidade](../../engenharia/qualidade.md) |
| Documentação | **MkDocs + Material** | — | ADR 0004 |
| Observabilidade | **Cloud Logging** (logs JSON) + alertas do Cloud Monitoring | Nativo do Cloud Run | — |

## 4. Consequências

**Positivas**

- A equipe trabalha na linguagem que escolheu.
- Validação de entrada declarativa (Pydantic) atende diretamente o checklist de segurança (allowlist de campos, tamanhos, formatos).
- Testes de API, contrato e E2E na mesma linguagem (pytest, Schemathesis, Playwright para Python).

**Negativas / custos**

- **Duas linguagens no repositório** (Python na API, TypeScript no frontend): tipos não são compartilhados. Mitigação: tipos do frontend gerados a partir do `openapi.yaml`.
- **Dois contratos possíveis** (o `openapi.yaml` escrito e o gerado pelo FastAPI). Mitigação: o `openapi.yaml` é a fonte (contrato primeiro) e um teste no CI falha se a API divergir dele.
- O SDK `firebase-admin` é síncrono. Rotas que o usam são declaradas com `def` (não `async def`), para o FastAPI executá-las em *thread pool* sem bloquear o servidor.
- Firestore exige modelagem orientada a consultas (sem joins); compensado pelo semáforo materializado (ADR 0007).

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Cold start com `min-instances=0` estourar p95 de 800 ms | Imagem *slim*; importações enxutas na subida; medir cedo; se necessário, `min-instances=1` só no Demo Day, registrado em ADR e no relatório de custo |
| API divergir do `openapi.yaml` | Teste de conformidade e Schemathesis no CI ([Qualidade](../../engenharia/qualidade.md)) |
| Rota síncrona declarada como `async def` travar o servidor | Regra no checklist de revisão de PR |
| Emulador de Storage não reproduzir URL assinada | Spike na S3; plano B na ADR 0006 |
| Dependência pesada piorar o Lighthouse ou a imagem | Toda dependência nova justificada no PR |

**Alternativas descartadas:** Node.js e Java (não escolhidas pela equipe); Kubernetes e Cloud SQL (custo fixo mensal); microsserviços (ADR 0005); acesso direto do navegador ao Firestore (ADR 0002).

## 6. Validação da decisão

- [x] Linguagem da API decidida pela equipe (14/10/2026).
- [ ] Esqueleto sobe com `docker compose up` (API FastAPI, web, emuladores Auth/Firestore/Storage) até 18/10.
- [ ] CI com Ruff, mypy e build verdes até 18/10.
- [ ] Teste de conformidade da API com o `openapi.yaml` ativo no CI até o CP2.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: tema e formato do ADR. Do Tech Lead: stack justificada, com a decisão de linguagem incorporada (C8). |
| 1.1 | 31/10/2026 | @pedrolmelo | Nota de substituição parcial pela ADR 0010 (Plano Custo Zero). |
