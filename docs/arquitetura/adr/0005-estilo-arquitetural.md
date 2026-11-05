# ADR 0005 — Monólito modular em camadas

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend, Frontend |
| Consultados | PO |
| Relacionados | [Visão Arquitetural](../index.md), ADR 0001, ADR 0003 |

## 1. Contexto

O MVP tem 5 jornadas, 9 semanas de prazo, equipe pequena e teto de US$ 50. O lab exige arquitetura em camadas (Apresentação, Aplicação, Domínio, Dados, Infra) e que o domínio atinja ≥ 80% de cobertura. O sistema precisa rodar localmente com um único `docker compose up`.

## 2. Opções consideradas

1. **Microsserviços por jornada** — isolamento de deploy; mas multiplica pipelines, contêineres, custos de cold start e pontos de falha. Não se justifica para o volume e o prazo.
2. **Funções serverless por rota (Cloud Functions)** — barato e simples por rota; mas espalha regras de autorização por dezenas de funções, dificulta testar o domínio de forma isolada e diverge do ambiente local em contêiner.
3. **Monólito modular em camadas, uma API em contêiner** — um deploy, um ponto de autorização, domínio isolado e testável; escala a zero no Render gratuito (ADR 0010).

## 3. Decisão

Adotar **um monólito modular** com duas aplicações: `apps/web` (SPA) e `apps/api` (API HTTP em contêiner). Dentro da API, código organizado por **módulo de negócio** (contratos, trabalhadores, documentos, conformidade, portaria, auditoria) e, dentro de cada módulo, pelas camadas **aplicação → domínio ← infra**. O domínio não importa nada de infraestrutura.

## 4. Consequências

- **Positivas:** uma única fronteira de autorização; domínio testável sem emuladores; mesmo contêiner no local e na nuvem; base simples para evoluir.
- **Negativas:** um bug pode derrubar a API inteira; a rota do job diário divide recursos com as rotas de usuário (aceitável no volume do MVP).
- **Obrigatório:** regra de dependência verificada em revisão (domínio sem imports de SDK); módulos se comunicam por interfaces de aplicação, não por acesso direto às coleções de outro módulo.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Um bug derruba a API inteira | Testes no CI bloqueando merge; rollback pelo painel do Render (ADR 0010) |
| Camadas se misturam com a pressa do prazo | Regra de lint de dependências entre pastas + checklist de revisão no template de PR |

## 6. Validação da decisão

- [ ] Revisão de PR com checklist "domínio sem dependência de infra".
- [ ] Regra de dependências entre camadas verificada no CI com **import-linter** (o `dominio/` não importa `infra/` nem FastAPI).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Monólito modular em camadas. Status: proposto, para aprovação até o CP2. |
| 1.1 | 03/11/2026 | @osamuelleite | Referências de hospedagem atualizadas para o Plano Custo Zero (ADR 0010). |
