# Relatórios de Testes e Qualidade

Relatórios de validação, qualidade e conformidade do Habilita organizados por marco de entrega (Checkpoint).

| Gate / Marco | Tipo de Relatório | Documento | Status |
|---|---|---|:---:|
| **CP1** (18/10) | QA & Isolamento Multi-tenant | [Checklist de QA e Testes de Isolamento (H19)](cp1-checklist-qa.md) | ✅ Concluído |
| **CP2 / CP3** (01/11–17/11) | Suíte EP/BVA & Critérios J1–J3 | [Matriz de Rastreabilidade](../matriz.md) | ✅ Concluído |
| **CP4** (26/11) | 5 Categorias, Carga `k6`, E2E API & Triagem de Bugs | [Relatório v1 (CP4)](relatorio-v1.md) · [Laudo por SHA (`ef25c03`)](../relatorio-ef25c03.md) | ✅ Concluído |

---

## Relatórios Disponíveis

- **[Checkpoint 4 — Relatório Consolidado v1](relatorio-v1.md)** e **[Laudo de Execução (`ef25c03`)](../relatorio-ef25c03.md)**: evidência completa sobre `test/hab-cp4-suites-qa` (`ef25c03e3e5d192306cb0355ad92163afb5fd3ed`), com **345 passed / 85 xfailed (`strict=True`) / 0 failed / 0 skipped**, **98,1%** de cobertura no domínio, **98,0%** global, homologação E2E nos 4 perfis (**38/38**), carga `k6` (`20 VUs × 60 s`) e catálogo `BUG-001`..`BUG-014` (`[PO-15]`).
- **[Checkpoint 1 — Checklist de QA e Testes de Isolamento Multi-tenant](cp1-checklist-qa.md)**: evidência dos testes `T-ISO-01` a `T-ISO-10` da história `H19` e conformidade arquitetural.

## Artefatos Exportados (`[QA-10]`)

| Gate | JUnit | Cobertura | Carga (`k6`) | Homologação E2E (4 Perfis) | Lighthouse |
|---|---|---|---|---|---|
| **CP4 (`ef25c03`)** | [junit-cp4.xml](junit-cp4.xml) | [coverage-cp4.xml](coverage-cp4.xml) | [k6-resumo-cp4.json](k6-resumo-cp4.json) | [homologacao-e2e-cp4.json](homologacao-e2e-cp4.json) | *Aguardando UI (`PR #55`)* |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Página para os relatórios gerados pelo CI em cada gate. |
| 1.1 | 26/11/2026 | @CauaRVeloso | Inclusão dos relatórios do Checkpoint 4 (`relatorio-v1.md` e `relatorio-ef25c03.md`) e links para os artefatos XML/JSON exportados (`[QA-10]`). |
