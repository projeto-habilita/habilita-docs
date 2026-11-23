# Checklist do Checkpoint 1 — Fundação (18/10/2026)

| Campo | Valor |
|---|---|
| Responsável pela conferência | PO (cobra) e Tech Lead (itens técnicos) |
| Última atualização | 02/11/2026 |

## Produto e negócio (PO)

| Item | Status | Evidência |
|---|---|---|
| Visão do produto com frase de LGPD | em revisão (PRs das rodadas 1 e 2) | [Visão e Escopo](../produto/estrategia/visao-escopo.md) |
| Ishikawa (6M, um grupo por causa no mínimo) | feito (imagem neste PR) | [Ishikawa](../produto/estrategia/ishikawa.md) |
| Backlog MoSCoW com pelo menos um Won't | em revisão (PR da rodada 3) | [Backlog](../produto/requisitos/historias-backlog.md) |
| Critérios de aceite mensuráveis | em revisão (PRs das rodadas 1 e 3) | [Requisitos Funcionais](../produto/requisitos/funcionais.md) e [Backlog](../produto/requisitos/historias-backlog.md) |
| 2 entrevistas ou evidências rotuladas | pendente | [Entrevistas e Evidências](../produto/estrategia/entrevistas.md) |
| Hipótese de defensibilidade + métrica da demo | em revisão (PR da rodada 2) | [Defensibilidade](../produto/estrategia/defensibilidade.md) |

## Arquitetura (Tech Lead)

| Item | Status | Evidência |
|---|---|---|
| 4 ADRs (0001 a 0004), incluindo multi-tenancy e RBAC | propostas · aprovação em PR | [ADRs](../arquitetura/adr/index.md) |
| Linguagem da API (ADR 0001) | Python + FastAPI (decidido em 14/10) | [ADR 0001](../arquitetura/adr/0001-stack.md) |
| OpenAPI v0.1 | 25 rotas de usuário + job + health | [API](../arquitetura/api.md) |
| Repositório organizado | documentado · 🔧 criar `apps/`, `seed/` | `README.md` na raiz |
| Definition of Done | [DoD](definition-of-done.md) |
| Estratégia de testes | [Estratégia de testes](../testes/estrategia.md) |
| Stack justificada | [ADR 0001](../arquitetura/adr/0001-stack.md) |
| Ferramentas de qualidade, tipagem e validação | [Qualidade](qualidade.md) |

## Ambiente (DevOps)

| Item | Status |
|---|---|
| Proteção da `main`, CODEOWNERS e template de PR | template · proteção e CODEOWNERS ([Como contribuir](contribuir.md)) |
| `.env.example` sem segredos |
| `docker compose up` com API, web e emuladores Auth/Firestore/Storage |
| Teste em máquina limpa por outro integrante |
| CI com Ruff, mypy, ESLint, `tsc` e build verdes |
| `mkdocs build --strict` verde e site no GitHub Pages | ([ADR 0004](../arquitetura/adr/0004-documentacao-mkdocs.md)) |
| Seed reproduzível |

## Segurança e custos

| Item | Status |
|---|---|
| Alertas de orçamento 50/80/100% (print) | 🔧 ([FinOps](../finops/index.md)) |
| Teste de isolamento de tenant **escrito** (pode falhar) | ✅ implementado (T-ISO-01 a 07) · [Relatório](../testes/relatorios/cp1-checklist-qa.md) |

## Ensaio da apresentação do gate

1. **Quem tem o problema** → Visão e Escopo, Personas.
2. **Qual é o MVP** → É / Não É, Jornadas, Backlog (Won't).
3. **Como será construído** → Arquitetura e ADRs 0001 a 0004.
4. **Como será validado** → Entrevistas e Estratégia de testes (EP/BVA, isolamento).
5. **Quanto pretende gastar** → FinOps: US$ 0 até a S4, alertas configurados, teto US$ 50.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: página criada pelo Tech Lead na conciliação. Checklist do Checkpoint 1 com status e evidência de cada item. |
| 1.1 | 18/10/2026 | @Lelafeta | Remoção da legenda e de emojis dos itens do checklist, com simplificação dos textos de status. |
| 1.2 | 02/11/2026 | @Lelafeta | Atualização da data e da coluna Status na tabela de Produto e negócio (PO). |
