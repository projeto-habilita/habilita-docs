# ADR 0012 — Teto de requisições do teste de carga em produção

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar em PR com o Tech Lead e mais um integrante; o valor do teto é proposta do Tech Lead e precisa da confirmação do tutor) |
| Data | 22/11/2026 |
| Decisores | Tech Lead, DevOps, QA |
| Consultados | Tutor e monitor |
| Relacionados | [ADR 0010](0010-plano-custo-zero.md) (regra 3 e riscos de cota), [Cotas gratuitas](../../finops/cotas-gratuitas.md), [Estratégia de testes](../../testes/estrategia.md) (carga local com k6), RNF04 |

## 1. Contexto

A ADR 0010 manda rodar testes pesados (k6, seed grande) **só no emulador local**. O feedback do CP3 pediu que fique registrado o **teto de requisições** para o caso de precisarmos medir algo contra o ambiente real. O Firestore Spark para tudo ao estourar 50.000 leituras/dia (20.000 escritas/dia), sem aviso e sem cobrança, e o Demo Day (05/12) não pode ser perdido por isso.

## 2. Opções consideradas

1. **Proibir qualquer carga em produção:** é o mais seguro, mas não deixa nenhuma evidência do p95 no ambiente real.
2. **Permitir um teste de fumaça com teto baixo e janela fechada:** mede o ambiente real sem consumir uma fração relevante da cota. *Escolhida.*
3. Liberar sem teto: pode esgotar a cota do dia. Descartada.

## 3. Decisão

Teste de carga contra o ambiente real só como **teste de fumaça**, com estes limites:

| Limite | Valor proposto | Como chegamos nele |
|---|---|---|
| Total de requisições por execução | **≤ 500** | Cada consulta lê poucos documentos; 500 requisições ficam bem abaixo de 1% das 50.000 leituras diárias, mesmo com 5 leituras por requisição (≈ 5% no pior caso) |
| Taxa | **≤ 5 requisições por segundo**, com no máximo 5 usuários virtuais | Evita pico e *cold start* em cascata |
| Execuções | **1 por dia**, no máximo 3 por gate | Preserva a cota do dia seguinte |
| Dados | Só o tenant de demonstração, sem criar documentos novos (somente leituras e a consulta da portaria) | Não consome escrita nem o 1 GB do Supabase |
| Janela proibida | **A partir de 28/11/2026** (uma semana antes do Demo Day) e nos dias de gate | A ADR 0010 já proíbe a véspera; aqui a janela é maior |
| Interrupção | Parar na primeira resposta 429/5xx ou se o painel do Firebase passar de **20.000 leituras no dia** | Margem de 60% da cota |
| Quem executa e registra | DevOps ou QA, com aviso ao Tech Lead; o resultado e o consumo de cota vão para o [Registro de gasto](../../finops/gasto.md) | Rastreabilidade |

Carga de verdade (1.000 trabalhadores, p95 ≤ 800 ms, RNF04) continua **só no Compose local**.

## 4. Consequências

- **Positivas:** há uma forma autorizada de medir o ambiente real e o risco para a cota fica limitado e registrado.
- **Negativas:** o teste de fumaça não prova o p95 com 1.000 trabalhadores; essa prova vem do teste local.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Alguém rodar o k6 apontando para produção por engano | O script exige a variável `ALVO=producao` e aborta acima do teto; revisão do Tech Lead no PR do script |
| Cota estourar mesmo assim | Parada em 20.000 leituras/dia; a cota volta no dia seguinte (00:00 PST), sem cobrança |

## 6. Validação da decisão

- [ ] Tutor confirmou o teto (500 requisições, 5 req/s).
- [ ] O script de carga implementa o teto e a variável `ALVO`.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 22/11/2026 | @osamuelleite | Versão inicial (débito do CP3: teto de requisições do teste de carga em produção). |
