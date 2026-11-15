# Demo Day

Roteiro: problema → hipótese de valor → jornada principal → critérios comprovados → segurança → qualidade → custo → defensibilidade.

Cena: trabalhador, documento, semáforo e portaria — incluindo reutilizar documento válido em outro contrato.

> **Demonstração no Checkpoint 3 ([ADR 0011, D5](../arquitetura/adr/0011-divergencias-caderno.md)):** conforme combinado com o tutor e registrado na ADR 0011, a demonstração ponta a ponta das jornadas ocorre localmente com `docker compose up`, sendo o deploy em nuvem entrega complementar. No CP3, o frontend está configurado para publicação no Firebase Hosting (`habilita-eb7ca`), enquanto a publicação da API em nuvem ficou registrada como débito técnico porque o Render e o Hugging Face Spaces exigiram cartão de crédito/plano pago (vedado pela Regra de Ouro do [Plano Custo Zero — ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)).

| Evidência | Link |
|---|---|
| Produção | Frontend no Firebase Hosting (`https://habilita-eb7ca.web.app` — publicado após merge do workflow `deploy-web.yml`); API pública pendente (débito técnico do CP3 — ver [Operação](../operacao/index.md#1-estado-operacional-no-checkpoint-3)) |
| CI | [GitHub Actions — Workflow `CI` (`projeto-habilita/Habilita`)](https://github.com/projeto-habilita/Habilita/actions/workflows/ci.yml) e [Workflow `docs` (`projeto-habilita/habilita-docs`)](https://github.com/projeto-habilita/habilita-docs/actions/workflows/docs.yml) |
| Testes e cobertura | [Estratégia de testes](../testes/estrategia.md) e [Matriz de rastreabilidade](../testes/matriz.md) |
| Lighthouse | *(pendente para o CP4 / Demo Day)* |
| Custos | [FinOps — Acompanhamento por gate](../finops/index.md) e [Registro de gasto — CP2 e CP3 (US$ 0,00)](../finops/gasto.md) |
| Rollback | [Runbook de Operação — Procedimento de Rollback](../operacao/index.md#4-procedimento-de-rollback) |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Roteiro e cena do Demo Day, com tabela de evidências a preencher. |
| 1.1 | 15/11/2026 | @pedrolmelo | Preenchimento dos links verificáveis no Checkpoint 3 (CI, testes, custos US$ 0,00 e runbook de rollback) e nota sobre a demonstração local (ADR 0011, D5) e o débito técnico da API pública. |
