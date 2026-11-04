# Visão e Escopo do Produto

!!! info "Sobre esta página"
    **Para que serve:** Uma página que qualquer pessoa lê em 2 minutos e entende o Habilita.

    **O que escrever:** Revisar e completar o rascunho abaixo, que veio do Guia de Projetos.

    **Prazo:** CP1 · 18/10

## Frase de visão

> **Documentação em dia. Acesso sob controle.**

## Para quem

Empresas que **contratam terceirizados**. Quem usa no dia a dia: **gerente de SST** (Saúde e Segurança do Trabalho) ou de **Facilities**.

Usuários do sistema: gestor(a) de SST/Facilities, responsável da empresa terceirizada e porteiro(a). Ver [Personas](personas.md).

## Problema

Os documentos dos trabalhadores terceirizados ficam espalhados em planilhas e e-mails. Fica difícil saber, na hora, **quem está apto a trabalhar**.

**Detalhamento (hipóteses a validar nas [entrevistas](entrevistas.md)):**

- **Vencimentos invisíveis.** Cada documento tem uma periodicidade diferente; o controle em planilha depende de alguém lembrar de olhar.
- **Portaria às cegas.** Quem libera a entrada não sabe, na hora, se o trabalhador está apto: libera sem verificar ou barra quem estava em dia.
- **Sem trilha.** Não há registro confiável de quem aprovou qual documento, nem de quem entrou e quando.

**Por que isso importa para a contratante:** a Lei nº 6.019/1974, art. 5º-A, § 3º, atribui à contratante a responsabilidade de garantir as condições de segurança, higiene e salubridade dos terceirizados quando o trabalho é feito em suas dependências. Ver [Evidências secundárias](evidencias-secundarias.md). As causas estão no [Ishikawa](ishikawa.md).

## Proposta

- **Semáforo de conformidade** por trabalhador e por contrato.
- **Alertas de vencimento** de documentos.
- **Consulta de liberação na portaria**, com registro de acesso.

**Declaração de posicionamento:** *para* empresas contratantes de terceirizados, *cujo problema é* ter documentos de SST dispersos e nenhuma forma rápida de saber quem está apto, *o Habilita é* uma plataforma B2B de conformidade documental *que* mostra o semáforo, alerta antes do vencimento e responde à portaria. *Diferente de* planilhas, e-mails e listas em papel, *o Habilita* calcula vencimentos sozinho, reaproveita documentos válidos entre contratos, separa os dados de cada cliente e registra quem aprovou e quem entrou.

## Escopo do MVP

Ver [É / Não É / Faz / Não Faz](e-nao-e.md) e [Jornadas](jornadas.md).

## Métricas de sucesso do MVP

| Métrica | Meta no Demo Day | Como medir |
|---|---|---|
| Tempo de resposta da consulta de portaria | p95 ≤ 800 ms | Logs de latência da rota (RNF04) |
| Documentos com vencimento em até 30 dias que geram alerta | 100% | Teste automatizado com massa de dados |
| Acesso indevido entre clientes | 0 ocorrências | Suíte de testes de isolamento (RNF01) |
| Métrica de defensibilidade | Ver [Defensibilidade](defensibilidade.md) | Cena da demo |
| Decisões de documento com autor e data registrados | 100% | Trilha de auditoria |

## Privacidade e LGPD

**Frase-resumo:** o Habilita guarda dados pessoais de trabalhadores terceirizados (nome, CPF em forma protegida e documentos de SST, incluindo o ASO, que contém informação de saúde) exclusivamente para verificar a aptidão para o trabalho exigida da contratante, enquanto forem necessários e por mais 90 dias, contados de quando o arquivo deixa de ser necessário, com expurgo automático pelo job diário (padrão até a orientação jurídica), e nunca os expõe a outro cliente.

| Ponto | Decisão no MVP |
|---|---|
| Papéis LGPD | A contratante (tenant) é a **controladora**; o Habilita atua como **operador**. |
| Dado pessoal sensível | O ASO registra aptidão (dado de saúde): acesso só por papéis autorizados, download por link temporário, nunca em logs. |
| Base legal provável | Cumprimento de obrigação legal ou regulatória pela controladora. **A confirmar** com orientação jurídica do lab ou do cliente. |
| Minimização | CPF guardado como hash com chave secreta (HMAC) mais versão mascarada; sem endereço, telefone, foto ou dados de saúde além do arquivo enviado. |
| Retenção | 90 dias contados de quando o arquivo deixa de ser necessário, com expurgo automático pelo job diário. Padrão até a orientação jurídica (Q4, 15/11). |
| Dados de demonstração | Somente dados fictícios (RNF09). |

Detalhes e controles em [Segurança](../../seguranca/index.md).

## Restrições do ciclo

| Item | Valor |
|---|---|
| Período | 04/10 a 05/12/2026 (9 semanas) |
| Base técnica | GCP + Firebase |
| Orçamento | Teto de US$ 50 (estimativa do projeto: US$ 8–18) |
| Complexidade | Média |

## Riscos de produto

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Não conseguir entrevistar gestores de SST a tempo | Média | Alto | Evidências públicas rotuladas como hipótese; contatos via rede do lab |
| Regras de validade variam muito entre clientes | Alta | Médio | Validade informada por documento e prazo padrão configurável por tipo |
| Porteiro sem tempo ou treino para usar o sistema | Média | Alto | Tela de um campo com resposta em cor; p95 ≤ 800 ms |
| Documento falso aprovado por engano | Média | Alto | Verificar autenticidade está fora do MVP; aprovação humana obrigatória e trilha de quem aprovou |
| Prazo apertado: jornadas J1 a J3 começaram tarde | Alta | Alto | Mesclar a branch do backend e adiar painel e compressão de imagem |
| Entrevistas reais ainda não realizadas | Média | Médio | Convidar 3 ou 4 pessoas; manter as personas rotuladas como hipótese |
| API no Render dorme após 15 min sem uso (cerca de 1 min para acordar) | Alta | Médio | Acordar a API antes da demo (chamada a /healthz ou monitor gratuito) |
| Supabase pausa após 1 semana sem uso | Média | Médio | Uso semanal; o job diário de expurgo mantém o projeto ativo |
| Cota diária do Firestore (plano Spark) estourar | Média | Alto | Testes e seed só no emulador; painel lê só a coleção de contagens |

## Questões em aberto

| # | Questão | Dono | Prazo | Decisão (01/11/2026) |
|---|---|---|---|---|
| Q1 | Um documento recém-rejeitado deve deixar o trabalhador vermelho mesmo que exista outro aprovado e válido do mesmo tipo? (Proposta técnica: não; RN-06) | PO | 18/10 | Não. Um documento rejeitado não derruba outro aprovado e válido do mesmo tipo (RN-06). |
| Q2 | Na portaria, basta estar apto em **um** contrato vigente, ou a consulta deve indicar o contrato do serviço do dia? (Proposta técnica: basta um; RN-10) | PO | 18/10 | Basta estar apto em um contrato vigente (RN-10). |
| Q3 | A antecedência do alerta (30 dias) deve ser configurável por cliente? (Proposta técnica: fixa no MVP) | PO | 01/11 | Em aberto. |
| Q4 | Prazo de retenção dos documentos após o fim do contrato | PO + orientação jurídica | 15/11 | Padrão de 90 dias contados de quando o arquivo deixa de ser necessário, com expurgo pelo job diário, até a orientação jurídica. |
| Q5 | Alertas de vencimento (H05) são **Should** ou **Must**? A jornada J4 da ficha inclui alertas no MVP | PO | 18/10 | H05 é Must: a jornada J4 da ficha inclui alertas no MVP. |
| Q6 | Métrica principal da demo: tempo de liberação na portaria ou taxa de reuso? Ver [Defensibilidade](defensibilidade.md) | PO | 18/10 | Opção B (taxa de reuso) como principal; Opção A (tempo na portaria) como complementar. |
| Q7 | Limite de upload: 5 MB (Caderno) ou 10 MB (carta da cliente)? | PO | — | 5 MB (5.000.000 bytes) no MVP, só PDF/JPG, configurável; 10 MB fica para depois do MVP, a confirmar com a cliente. |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: página, frase de visão, público, problema e proposta. Do Tech Lead: frase e tabela de LGPD (C14), riscos e questões Q1 a Q6 para decisão da PO. |
| 1.1 | 01/11/2026 | @Lelafeta | Definição padrão de retenção de 90 dias com expurgo automático pós-contrato. |
| 1.2 | 01/11/2026 | @Lelafeta | Registro das decisões da PO para as questões Q1, Q2, Q4, Q5 e Q6. |
| 1.3 | 02/11/2026 | @Lelafeta | Inclusão dos riscos de prazo das jornadas J1 a J3 e de entrevistas reais pendentes na tabela de riscos de produto. |
| 1.4 | 03/11/2026 | @Lelafeta | Ajuste da regra de retenção de 90 dias contados de quando o arquivo deixa de ser necessário, com expurgo pelo job diário. |
| 1.5 | 04/11/2026 | @Lelafeta | Inclusão dos riscos de infraestrutura gratuita e da questão Q7 sobre o limite de upload. |
