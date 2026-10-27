# ADR 0007 — Recálculo diário idempotente de vencimentos

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend |
| Consultados | PO |
| Relacionados | H15, H05, RN-04, RN-12, RN-13, [ADR 0010](0010-plano-custo-zero.md) (Plano Custo Zero) |

> **Atenção:** os itens de hospedagem/armazenamento/agendamento/custo desta ADR estão sendo substituídos pela [ADR 0010](0010-plano-custo-zero.md) (Proposto).

## 1. Contexto

Documentos mudam de situação (válido → a vencer → vencido) apenas com a passagem do tempo, sem nenhuma ação de usuário. Alguém precisa recalcular todo dia. Jobs agendados podem rodar duas vezes (reentrega do agendador, nova tentativa após falha, execução manual). Rodar duas vezes **não pode** duplicar alertas nem deixar dados inconsistentes. A ficha do projeto destaca o "recálculo idempotente" como desafio de engenharia.

## 2. Opções consideradas

1. **Calcular a situação só na leitura** — sem job; mas alertas precisam ser gerados em algum momento, e a consulta de portaria ficaria mais lenta.
2. **Gatilhos do Firestore** — reagem a mudanças de dados, não à passagem do tempo. Não resolvem o problema.
3. **Cloud Scheduler chamando uma rota interna da API, com cálculo como função pura e escritas determinísticas.**

## 3. Decisão

1. **Agendamento:** Cloud Scheduler chama `POST /internal/jobs/recalculo-vencimentos` diariamente às 03:00 (America/Sao_Paulo), com token **OIDC** da conta de serviço do agendador. A rota rejeita qualquer outro chamador.
2. **Entrada explícita:** a requisição traz `dataReferencia` (padrão: hoje no fuso de São Paulo). O cálculo nunca lê o relógio diretamente; o relógio é injetado. Isso torna o job reproduzível e testável para qualquer data.
3. **Função pura:** `situacao = f(validoAte, dataReferencia)` (RN-04) e `semaforo = g(documentos, matriz, dataReferencia)` (RN-06). Mesmas entradas, mesmas saídas.
4. **Escritas idempotentes:**
   - situação e semáforo gravados com `set` (sobrescrever com o valor calculado, nunca incrementar);
   - alertas com **ID determinístico** `{documentoId}_{tipo}` criados com "criar se não existir" (RN-12);
   - nenhuma escrita usa contador ou acumulação.
5. **Controle de execução:** documento `execucoesJob/{dataReferencia}` com status (`EM_ANDAMENTO`, `CONCLUIDO`, `FALHOU`) e contagens. Uma nova execução para a mesma data **reprocessa tudo** (é seguro, por ser idempotente) e atualiza o registro.
6. **Recálculo por evento:** aprovação, rejeição, vínculo e alteração de matriz recalculam imediatamente as alocações afetadas usando as mesmas funções (RN-13).
7. **Contagens do dashboard (`contagens/{contratoId}`):** o job diário **recalcula do zero** as contagens de cada contrato a partir das alocações e grava com `set`, o que também corrige qualquer desvio. No recálculo por evento, a mesma transação que grava a alocação lê a cor antiga e o documento de contagens e grava o valor novo já calculado (também com `set`, sem `increment`). Se a cor não mudou, nada é escrito. A regra "nenhuma escrita usa contador" continua valendo.
8. **Marcação de expurgo:** o job grava o `customTime` dos arquivos que deixaram de ser necessários (vencidos e sem vínculo ativo), conforme a ADR 0006 §3.4. A marcação é idempotente: mesma data de referência, mesmo `customTime`.
9. **Proteção de custo:** processamento em lotes, apenas documentos `APROVADO`, com limite máximo de documentos por execução configurável (alerta se atingido).

## 4. Consequências

- **Positivas:** reexecutar é sempre seguro, o que simplifica a recuperação de falhas; o cálculo é 100% testável em unidade; a demo pode simular "daqui a 30 dias" passando outra `dataReferencia` em ambiente de teste.
- **Negativas:** reprocessar tudo a cada execução custa leituras proporcionais ao número de documentos aprovados (pequeno no MVP; monitorado em FinOps).
- **Obrigatório:** proibido usar `Date.now()` ou equivalente dentro do domínio; o relógio vem por injeção.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Reprocessar tudo a cada execução aumentar leituras | Ler só documentos aprovados; lotes; limite configurável com alerta; acompanhar em FinOps |
| Relógio do servidor usado dentro do domínio | Relógio injetado; revisão de PR reprova uso direto de data atual no domínio |
| Falha no meio do lote | Execução reexecutável com segurança; registro em `execucoesJob/{data}` |

## 6. Validação da decisão

- [ ] H15.CA1: executar duas vezes para a mesma data e comparar o estado final (teste de integração no emulador).
- [ ] H15.CA4: simular falha no meio do lote e reexecutar.
- [ ] Testes de valor-limite D−1, D, D+29, D+30, D+31 no domínio.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Recálculo diário idempotente de vencimentos. Status: proposto. |
| 1.1 | 31/10/2026 | @pedrolmelo | Nota de substituição parcial pela ADR 0010 (Plano Custo Zero). |
| 1.2 | 03/11/2026 | @osamuelleite | Contagens do dashboard (`contagens/{contratoId}`) recalculadas sem incremento; marcação de expurgo da ADR 0006 (feedback do CP1, item 3). |
