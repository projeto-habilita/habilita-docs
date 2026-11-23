# Controle semanal da equipe

| Campo | Valor |
|---|---|
| Status | Em uso |
| Responsável | Tech Lead (conduz e registra); PO (backlog) |
| Quando | Toda **terça-feira**, 20 a 30 minutos (Caderno B2B, "Controle semanal da equipe") |

Cada reunião vira uma entrada nesta página, a mais recente no topo, respondendo às 7 perguntas do Caderno **com a evidência** (link de PR, issue, CI ou relatório). Pergunta sem evidência fica marcada como "sem evidência". A revisão de prioridades do backlog é registrada pela PO em [/docs/backlog](../backlog/index.md); aqui fica só o link.

## Modelo (copie para cada semana)

```markdown
### Semana N · DD/MM/2026

Presentes: @… · Ausentes: @… · Duração: … min

| Pergunta | Resposta | Evidência |
|---|---|---|
| 1. O que foi concluído? | | PR #…, issue #… |
| 2. O que está bloqueado? | Bloqueio, impacto e responsável | issue #… |
| 3. Qual critério foi coberto? | HNN.CAn → teste | link do teste / CI |
| 4. Qual risco aumentou? | | ADR / backlog / mitigação |
| 5. Quanto já gastamos? | US$ … (meta do gate: ≤ US$ …) | [FinOps](../finops/index.md) |
| 6. O que será feito na próxima semana? | Histórias e responsáveis | board |
| 7. O que precisa de tutor? | Pergunta objetiva, contexto e prazo | [ADR 0011 §4](../arquitetura/adr/0011-divergencias-caderno.md) |

Decisões: …
Ações: [ ] … (@responsável, prazo)
```

## Registros

### Semana 4 · 27/10/2026

!!! todo "Preencher na reunião de 27/10"
    Pauta sugerida pelo Tech Lead: (1) pré-requisitos do CP2 (isolamento no CI, números alinhados ao Caderno); (2) aprovação em PR das ADRs 0001 a 0004 e revisão das 0006, 0010 e 0011; (3) proteção da `main` ativa nos dois repositórios; (4) jornadas 1 a 3 ponta a ponta até 31/10; (5) perguntas ao tutor (Q-T1 a Q-T4, prazo 29/10); (6) gasto atual × meta de US$ 5.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/10/2026 | @osamuelleite | Página criada com o modelo das 7 perguntas do Caderno (planilha TL-13). |
