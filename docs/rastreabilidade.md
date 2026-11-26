# Rastreabilidade

A matriz requisito → critério de aceite → teste → categoria → resultado → evidência fica em [Matriz de rastreabilidade](testes/matriz.md).

## Cobertura dos critérios Must (MVP)

A matriz rastreia **100% das 19 histórias Must** do [Backlog](produto/requisitos/historias-backlog.md), totalizando **78 critérios de aceite (CA)**:

| Escopo | Histórias Must | Quantidade de CAs | Observação |
|---|---|---:|---|
| **Jornada J1** (Contratos, matriz, terceirizadas e tipos) | `H01` (6), `H08` (3), `H09` (3) | 12 | `H09.CA3` depende da interface Web (`PR #55`) |
| **Jornada J2** (Trabalhadores, upload, alocação e reuso) | `H02` (11), `H10` (3), `H11` (3) | 17 | `H02.CA11` depende da interface Web (`PR #55`) |
| **Jornada J3** (Fila de pendentes, aprovação, rejeição e reenvio) | `H03` (4), `H12` (3), `H13` (3), `H14` (2) | 12 | `H12.CA2` depende da interface Web (`PR #55`) |
| **Jornada J4** (Alertas, job diário, semáforo e painel) | `H05` (4), `H15` (5), `H16` (4), `H23` (2) | 15 | Especificações e contrato cobertos; 4 rotas J4 pendentes na API |
| **Jornada J5** (Consulta de liberação e acesso na portaria) | `H04` (6), `H18` (4) | 10 | Especificações e contrato cobertos; 2 rotas J5 pendentes na API |
| **Transversais** (Autenticação, isolamento multi-tenant e auditoria) | `H07` (4), `H19` (5), `H20` (3) | 12 | Rota `/auditoria` e tela inicial `porteiro` (`PR #55`) pendentes |
| **Total Must** | **19 histórias** | **78** | **100% dos critérios Must mapeados** |

> As histórias `H17` (*Could*, 2 CAs), `H21` (*Should*, 2 CAs) e `H22` (*Could*, 1 CA) não integram o escopo obrigatório **Must** do MVP.

## Relatórios de execução

- [Visão geral dos relatórios](testes/relatorios/index.md)
- [Checkpoint 4 — Laudo de QA (`a012ee3`)](testes/relatorio-a012ee3.md) e [Relatório Consolidado v1](testes/relatorios/relatorio-v1.md)
- [Checklist do Checkpoint 1 — Isolamento Multi-tenant](testes/relatorios/cp1-checklist-qa.md)

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 01/11/2026 | @Lelafeta | Página criada no caminho pedido pelo Caderno, apontando para a matriz. |
| 1.1 | 26/11/2026 | @CauaRVeloso | Inclusão do resumo de cobertura 100% dos 78 critérios Must (19 histórias, J1–J5 e transversais) e links para os relatórios e laudo do CP4 (`a012ee3`). |
