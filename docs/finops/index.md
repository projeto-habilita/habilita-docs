# FinOps

| Campo | Valor |
|---|---|
| Responsável | DevOps (aprovação: Tech Lead) |
| Última atualização | 15/11/2026 |
| Relacionados | [Registro de gasto (CP2 e CP3)](gasto.md), [Cotas dos planos gratuitos](cotas-gratuitas.md), [ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md) |

## Acompanhamento por gate

| Gate | Teto acumulado | Gasto real | Ação |
|---|---|---|---|
| CP2 | **US$ 0,00** | **US$ 0,00** | Plano Custo Zero ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)); conferência em [Registro de gasto](gasto.md) |
| CP3 | **US$ 0,00** | **US$ 0,00** | Planos gratuitos comprovados em Firebase Spark (`habilita-eb7ca`), Supabase Free, GitHub Actions e UptimeRobot ([Registro de gasto — CP3](gasto.md#checkpoint-3-15112026)). API pública não publicada pois Render e Hugging Face Spaces exigiram cartão/plano pago (débito técnico; demo local via Compose conforme [ADR 0011, D5](../arquitetura/adr/0011-divergencias-caderno.md)) |
| CP4 | **US$ 0,00** (gatilho de cotas) | [preencher] | Auditoria de cotas no lugar do congelamento financeiro |

> **Nota histórica:** Tetos originais do Caderno: US$ 5 (CP2) / US$ 12 (CP3) / US$ 40 (CP4). Referência mantida apenas para rastreabilidade histórica.

A documentação é publicada no GitHub Pages e não consome orçamento de nuvem.

Estimativa da ficha do projeto original: **US$ 8 a 18** no ciclo inteiro (substituída pelo Plano Custo Zero com teto de US$ 0,00).

**Regra de ouro:** nenhum integrante cadastra cartão de crédito ou ativa conta de faturamento em qualquer serviço. Quando um provedor passa a exigir cartão para criar recurso gratuito (como ocorreu com o Render e o Hugging Face Spaces no CP3), o serviço não é ativado e o item é registrado como débito técnico.

## Prova de custo zero (substitui os alertas 50/80/100%)

No **Plano Custo Zero** ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)), nenhum cartão de crédito é cadastrado e nenhuma conta de faturamento é vinculada aos serviços. Dessa forma, os alertas de orçamento em porcentagem (50%, 80% e 100% de US$ 50) no console de faturamento do GCP não se aplicam e não podem ser gerados.

A comprovação de custo zero é realizada mediante a captura de tela (print) do console de cada serviço evidenciando o plano gratuito (Spark / Free Tier) e a inexistência de formas de pagamento ativas.

As evidências fotográficas e o passo a passo de conferência independente estão registrados em detalhe na página [Registro de gasto — Checkpoints 2 e 3](gasto.md). Para o detalhamento de limites operacionais e impactos no produto, consulte [Cotas dos planos gratuitos](cotas-gratuitas.md).

## Controles de custo e cotas por serviço

| Serviço | Controle |
|---|---|
| **Firebase Spark** | Firestore com listas paginadas (máx. 50); sem *listeners* em tempo real sem justificativa; semáforo materializado para evitar leituras repetidas; testes automatizados pesados, k6 e seed volumoso rodam exclusivamente no emulador local. No plano Spark, quando a cota diária (50 mil leituras) é atingida, o serviço pausa até o dia seguinte, sem qualquer cobrança financeira. |
| **Render (API — pendente no CP3)** | Previsto na [ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md) com franquia de 750 horas/mês por workspace e suspensão após 15 min de inatividade. No CP3, não foi provisionado porque o cadastro passou a exigir cartão de crédito; registrado como débito técnico. |
| **Supabase Storage** | Limite máximo de 5 MB (**5.000.000 bytes**, [ADR 0006](../arquitetura/adr/0006-armazenamento-de-documentos.md)) por arquivo configurado na aplicação; bucket privado com download por URL assinada de 300 s; expurgo de 90 dias feito pelo job diário; seed de testes controlado (máx. 3 documentos por trabalhador). |
| **GitHub Actions** | Pipelines otimizados com cache de dependências (`uv` e `npm`); deploy do Firebase Hosting acionado apenas em PRs do próprio repositório e após `CI` verde na `main`; job diário enxuto. |
| **UptimeRobot** | 50 monitores gratuitos checando a disponibilidade a cada 5 min com alertas por e-mail, sem cartão cadastrado. |
| **Logging** | Logs estruturados e sem payloads grandes; nível `info` em produção; sem dados pessoais. |

**Nenhum recurso novo em nuvem sem ADR ou aprovação do Tech Lead** (Definition of Done).

## Plano de corte por esgotamento de cotas

Em substituição ao antigo gatilho financeiro de US$ 40, o plano de contenção atua preventivamente sobre o consumo das cotas gratuitas:

1. **Cota diária do Firestore próxima do teto ou estourada:** suspender consultas não essenciais; aguardar a renovação automática da cota diária (00:00 PST); auditar chamadas da API no ambiente local com emulador para identificar queries multiplicadoras de leitura.
2. **Consumo de horas de hospedagem da API (750 h/mês quando aplicável):** garantir que no máximo um único web service esteja ativo no workspace; pausar deploys automáticos em branches que não sejam a `main`.
3. **Projeto Supabase pausado por inatividade:** restaurar o projeto no painel ou acionar rotina de manutenção antes de testes ou demonstrações.
4. **Armazenamento do Supabase próximo de 1 GB:** executar expurgo de arquivos vencidos/descartáveis não vinculados a trabalhadores ativos.
5. **Registrar a ação e o motivo:** documentar no relatório semanal de FinOps.

## Relatório semanal

Preencher a partir da Semana 3 (e consolidar na tabela por gate acima):

| Semana | Gasto na semana (US$) | Acumulado (US$) | Maior item | Ação |
|---|---|---|---|---|
| S1 | 0,00 | 0,00 | — | Tudo local |
| S2 | 0,00 | 0,00 | — | Tudo local (preparação CP2) |
| S3 | 0,00 | 0,00 | — | Emuladores locais e ajustes pós-CP2 |
| S4 | 0,00 | 0,00 | — | Integração de Supabase Storage local no Compose |
| S5 | 0,00 | 0,00 | — | CP3: Firebase Hosting (`habilita-eb7ca`), Supabase Free e UptimeRobot Free ativos sem cartão; API pública registrada como débito técnico |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Orçamento, limites por gate e alertas de 50/80/100%. |
| 1.1 | 31/10/2026 | @pedrolmelo | Atualização para o Plano Custo Zero: limites de US$ 0 por gate, substituição de alertas por prova de plano gratuito, controles de cotas para Firebase Spark, Render, Supabase e Actions, e plano de corte baseado em cotas. |
| 1.2 | 02/11/2026 | @pedrolmelo | Ajuste na menção ao limite de upload de documentos para 5 MiB (5.242.880 bytes). |
| 1.3 | 15/11/2026 | @pedrolmelo | Registro do Checkpoint 3 (US$ 0,00), atualização do relatório semanal (S3–S5), alinhamento do limite de upload a 5 MB (5.000.000 bytes, ADR 0006 v3) e registro do débito técnico da API pública. |
