# FinOps

| Campo | Valor |
|---|---|
| Responsável | DevOps (aprovação: Tech Lead) |
| Última atualização | 31/10/2026 |
| Relacionados | [Registro de gasto (CP2)](gasto.md), [Cotas dos planos gratuitos](cotas-gratuitas.md), [ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md) |

## Acompanhamento por gate

| Gate | Teto acumulado | Gasto real | Ação |
|---|---|---|---|
| CP2 | **US$ 0,00** | US$ 0,00 | Plano Custo Zero ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)); conferência em [Registro de gasto](gasto.md) |
| CP3 | **US$ 0,00** | [preencher] | Manter tier gratuito em Render, Supabase, Actions e Spark |
| CP4 | **US$ 0,00** (gatilho de cotas) | [preencher] | Auditoria de cotas no lugar do congelamento financeiro |

> **Nota histórica:** Tetos originais do Caderno: US$ 5 (CP2) / US$ 12 (CP3) / US$ 40 (CP4). Referência mantida apenas para rastreabilidade histórica.

A documentação é publicada no GitHub Pages e não consome orçamento de nuvem.

Estimativa da ficha do projeto original: **US$ 8 a 18** no ciclo inteiro (substituída pelo Plano Custo Zero com teto de US$ 0,00).

**Regra de ouro:** até a Semana 4, tudo roda localmente com Docker Compose e emuladores. Pergunta "precisamos subir para o Firebase já?" → resposta: não.

## Prova de custo zero (substitui os alertas 50/80/100%)

No **Plano Custo Zero** ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)), nenhum cartão de crédito é cadastrado e nenhuma conta de faturamento é vinculada aos serviços. Dessa forma, os alertas de orçamento em porcentagem (50%, 80% e 100% de US$ 50) no console de faturamento do GCP não se aplicam e não podem ser gerados.

A comprovação de custo zero é realizada mediante a captura de tela (print) do console de cada serviço evidenciando o plano gratuito (Spark / Free Tier) e a inexistência de formas de pagamento ativas.

As evidências fotográficas e o passo a passo de conferência independente estão registrados em detalhe na página [Registro de gasto — Checkpoint 2](gasto.md). Para o detalhamento de limites operacionais e impactos no produto, consulte [Cotas dos planos gratuitos](cotas-gratuitas.md).

## Controles de custo e cotas por serviço

| Serviço | Controle |
|---|---|
| **Firebase Spark** | Firestore com listas paginadas (máx. 50); sem *listeners* em tempo real sem justificativa; semáforo materializado para evitar leituras repetidas; testes automatizados pesados, k6 e seed volumoso rodam exclusivamente no emulador local. No plano Spark, quando a cota diária (50 mil leituras) é atingida, o serviço pausa até o dia seguinte, sem qualquer cobrança financeira. |
| **Render** | Web service gratuito com suspensão automática (*cold start*) após 15 min de inatividade; limitação a 1 único serviço por workspace para não exceder a franquia de 750 horas/mês gratuitas. |
| **Supabase Storage** | Limite máximo de 5 MB por arquivo configurado na aplicação; bucket privado com download por URL assinada; sem duplicação de arquivos no reuso; seed de testes controlado (máx 3 documentos por trabalhador); workflow semanal de manutenção para evitar pausa por inatividade após 7 dias. |
| **GitHub Actions** | Pipelines otimizados com cache de dependências; job de recálculo diário enxuto consumindo a cota de 2.000 min/mês (repositório privado) ou ilimitado (repositório público). |
| **UptimeRobot** | 50 monitores gratuitos checando `/readyz` a cada 5 min com alertas por e-mail, sem cartão cadastrado. |
| **Logging** | Logs estruturados e sem payloads grandes; nível `info` em produção; sem dados pessoais. |

**Nenhum recurso novo em nuvem sem ADR ou aprovação do Tech Lead** (Definition of Done).

## Plano de corte por esgotamento de cotas

Em substituição ao antigo gatilho financeiro de US$ 40, o plano de contenção atua preventivamente sobre o consumo das cotas gratuitas:

1. **Cota diária do Firestore próxima do teto ou estourada:** suspender consultas não essenciais; aguardar a renovação automática da cota diária (00:00 PST); auditar chamadas da API no ambiente local com emulador para identificar queries multiplicadoras de leitura.
2. **Consumo de horas do Render aproximando-se de 750 h/mês:** garantir que nenhum outro web service esteja ativo no workspace; pausar deploys automáticos em branches que não sejam a `main`.
3. **Projeto Supabase pausado por inatividade:** acionar manualmente o workflow de manutenção no GitHub Actions para restaurar a disponibilidade do bucket antes de testes ou demonstrações.
4. **Armazenamento do Supabase próximo de 1 GB:** executar script de expurgo de arquivos de teste e seeds antigos descartáveis não vinculados a trabalhadores ativos.
5. **Registrar a ação e o motivo:** documentar no relatório semanal de FinOps.

## Relatório semanal

Preencher a partir da Semana 3 (e consolidar na tabela por gate acima):

| Semana | Gasto na semana (US$) | Acumulado (US$) | Maior item | Ação |
|---|---|---|---|---|
| S1 | 0,00 | 0,00 | — | Tudo local |
| S2 | 0,00 | 0,00 | — | Tudo local (preparação CP2) |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Orçamento, limites por gate e alertas de 50/80/100%. |
| 1.1 | 31/10/2026 | @pedrolmelo | Atualização para o Plano Custo Zero: limites de US$ 0 por gate, substituição de alertas por prova de plano gratuito, controles de cotas para Firebase Spark, Render, Supabase e Actions, e plano de corte baseado em cotas. |
