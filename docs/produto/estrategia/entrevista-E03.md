# Entrevista E03 — Advogada trabalhista (escritório e jurídico de empresa)

| Campo | Valor |
|---|---|
| Código | E03 |
| Data | 10/11/2026 |
| Perfil | Advogada trabalhista, com experiência em escritório e como advogada de empresa; relata casos de empresas de grande porte (mais de 200 funcionários), construção civil e limpeza |
| Canal | Vídeo |
| Entrevistador / anotador | PO |
| Consentimento verbal | Sim |
| Gravação | Autorizada; apagar após a conferência da transcrição |

Entrevista de 36 minutos, conduzida com o roteiro voltado ao corpo jurídico. As respostas abaixo foram distribuídas pelas perguntas do [Roteiro de entrevista](roteiro-entrevista.md); onde a entrevistada não respondeu, o campo traz "não informado".

## Respostas

| # | Resumo da resposta (fatos e números) | Citação curta relevante (opcional, anonimizada) |
|---|---|---|
| C1 | Advogada trabalhista, em escritório e no jurídico de empresas. Tempo na função: não informado | |
| C2 | Não deu número de terceirizados. Terceiros fazem logística, limpeza e serviços gerais em grandes empresas. Dois modelos: contratos fixos renovados mês a mês (ex.: limpeza de órgãos públicos) e contratação por demanda (ex.: construção civil, a cada obra ou licitação) | "é complicado fazer um número redondo" |
| P1 | Não informado | |
| P2 | Parcial: na construção civil, cada obra ou licitação gera nova contratação de empresa terceira. Frequência e etapas: não informado | |
| P3 | O departamento pessoal de cada empresa usa sistema próprio: planilha, pastas locais ou sistema de RH. Recomenda sistema online em nuvem, para não perder documentos com troca de máquina, de funcionário ou de senha. Citou guarda de 15 anos (prazo não conferido; ver "Surpresas") | "É muito comum as empresas fazerem esse controle por meio de planilhamento" |
| P4 | Vencimento sem ninguém perceber "acontece sempre", sobretudo em empresas com mais de 200 funcionários e sem lembretes. A renovação do ASO periódico depende de o time médico acionar o empregado. A falha aparece em auditoria interna, no canal de denúncias e, com mais frequência, no processo trabalhista. Caso relatado: empregado demitido para quem não se encontrou ficha de EPI, treinamento nem assinatura de entrega | "esse tipo de questão acontece sempre" |
| P5 | Quando o trabalhador volta a prestar serviço depois de um tempo, entrega tudo de novo e refaz o ASO. Considera o retrabalho necessário, porque a empresa não sabe o que aconteceu no intervalo. Não tratou de contratos simultâneos. Tempo gasto: não informado | "gera um retrabalho, mas é algo necessário" |
| P6 | Rejeição por: documento fora do prazo, sem a formalidade legal, ausência de documento ou dados inconsistentes com o cadastro anterior. Prazo usual de correção: 15 dias, prorrogáveis por mais 15. A aprovação cabe a engenheiro ou técnico de segurança com registro ativo no conselho. Como avisam a recusa e tempo de conferência: não informado | |
| P7 | Hoje praticamente não há aviso na entrada. O ideal seria a segurança do trabalho avisar a portaria e bloquear o acesso depois de uma abordagem com o trabalhador. Em empresas grandes, leva de 3 a 4 dias até o supervisor saber de um problema. Casos concretos de entrada ou bloqueio indevido: não informado | "hoje isso praticamente não é visto" |
| P8 | Tempo para montar a situação de todos os terceirizados: não informado. A prova da fiscalização é a troca de e-mails, as pastas compartilhadas por contratada, telegramas e avisos de recebimento. Sem resposta após três cobranças, a contratante pode rescindir o contrato | |
| P9 | Pergunta não feita nesse formato. Indicou o que espera de um sistema: nuvem confiável, lembretes de vencimento, dupla conferência, acesso por perfil (documento médico só para o médico; documentos de segurança só para o engenheiro) e senha pessoal e intransferível. Preocupação: vazamento de dados | |
| P10 | A compra fica com o financeiro, alinhado com o jurídico e com os técnicos de segurança do trabalho. Valor para software: não informado (os valores citados eram de fornecimento de EPI) | |

## Hipóteses

| Hipótese | Resultado | Justificativa |
|---|---|---|
| HP1 · Controle em planilha ou e-mail | Confirmada | Planilha e pastas locais são comuns (P3, P4); relato indireto, de quem atua no jurídico |
| HP2 · Vencimentos despercebidos | Confirmada | "Acontece sempre"; a falha é descoberta tarde, por auditoria, denúncia ou processo (P4) |
| HP3 · Portaria sem verificação na hora | Confirmada | Hoje praticamente não há aviso na entrada; de 3 a 4 dias até o supervisor saber (P7) |
| HP4 · Reenvio de documentos entre contratos | Inconclusiva | Confirma o reenvio quando o trabalhador volta, mas o considera necessário; não falou de contratos simultâneos (P5) |
| HP5 · Decisor é SST ou Facilities | Refutada | Quem decide é o financeiro; jurídico e segurança do trabalho influenciam (P10) |

## Surpresas e novas hipóteses

- **Retenção:** a entrevistada citou guarda de 15 anos para prova judicial. O prazo não foi conferido em fonte oficial e conflita com o expurgo de 90 dias do MVP. Verificar a base legal antes de mudar a regra.
- **Acesso por perfil:** o ASO é documento médico e, segundo ela, só deveria ser acessado pelo médico da empresa.
- **Aprovador habilitado:** a aprovação deveria registrar o profissional responsável e o seu registro no conselho, não só o usuário.
- **Prazo de correção:** 15 dias, mais 15, é o prazo que ela vê no mercado para reenvio após rejeição.
- **Prova da cobrança:** a defesa jurídica depende do histórico de cobranças à prestadora, não só do documento.
- **EPI:** ficha de entrega e vida útil de EPI aparecem em quase todas as respostas e estão fora do escopo do MVP.
- **Limite desta entrevista:** P1, P8 e P9 não foram respondidas, e o baseline da métrica (reenvios e tempo para habilitar um trabalhador) não foi medido. A próxima entrevista deve ser com gestor(a) de SST ou Facilities.

## Impacto no backlog

| Mudança sugerida | História afetada | Prioridade sugerida |
|---|---|---|
| Rever a regra de retenção de 90 dias diante da guarda para prova judicial | Q4 da Visão; [ADR 0006](../../arquitetura/adr/0006-armazenamento-de-documentos.md) | Should (decisão antes do CP4) |
| Restringir o acesso ao ASO por papel | H19 | Won't no MVP |
| Prazo de correção de 15 dias, prorrogável, após a rejeição | H14 | Won't no MVP |
| Registrar o responsável técnico na aprovação | H13 | Won't no MVP |
| Histórico de cobranças à prestadora exportável como prova | Nova | Won't no MVP |
| Ficha de entrega de EPI | Nova | Won't no MVP |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 16/11/2026 | @Lelafeta | Registro da entrevista E03 a partir da transcrição da gravação, sem dados pessoais. |
