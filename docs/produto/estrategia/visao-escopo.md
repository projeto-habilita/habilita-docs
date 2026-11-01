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

**Frase-resumo:** o Habilita guarda dados pessoais de trabalhadores terceirizados (nome, CPF em forma protegida e documentos de SST, incluindo o ASO, que contém informação de saúde) exclusivamente para verificar a aptidão para o trabalho exigida da contratante, pelo tempo de vigência do contrato acrescido de 90 dias, com expurgo automático (padrão até a orientação jurídica), e nunca os expõe a outro cliente.

| Ponto | Decisão no MVP |
|---|---|
| Papéis LGPD | A contratante (tenant) é a **controladora**; o Habilita atua como **operador**. |
| Dado pessoal sensível | O ASO registra aptidão (dado de saúde): acesso só por papéis autorizados, download por link temporário, nunca em logs. |
| Base legal provável | Cumprimento de obrigação legal ou regulatória pela controladora. **A confirmar** com orientação jurídica do lab ou do cliente. |
| Minimização | CPF guardado como hash com chave secreta (HMAC) mais versão mascarada; sem endereço, telefone, foto ou dados de saúde além do arquivo enviado. |
| Retenção | 90 dias após o fim do contrato, com expurgo automático. Padrão até a orientação jurídica (Q4, 15/11). |
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

## Questões em aberto

| # | Questão | Dono | Prazo |
|---|---|---|---|
| Q1 | Um documento recém-rejeitado deve deixar o trabalhador vermelho mesmo que exista outro aprovado e válido do mesmo tipo? (Proposta técnica: não; RN-06) | PO | 18/10 |
| Q2 | Na portaria, basta estar apto em **um** contrato vigente, ou a consulta deve indicar o contrato do serviço do dia? (Proposta técnica: basta um; RN-10) | PO | 18/10 |
| Q3 | A antecedência do alerta (30 dias) deve ser configurável por cliente? (Proposta técnica: fixa no MVP) | PO | 01/11 |
| Q4 | Prazo de retenção dos documentos após o fim do contrato | PO + orientação jurídica | 15/11 |
| Q5 | Alertas de vencimento (H05) são **Should** ou **Must**? A jornada J4 da ficha inclui alertas no MVP | PO | 18/10 |
| Q6 | Métrica principal da demo: tempo de liberação na portaria ou taxa de reuso? Ver [Defensibilidade](defensibilidade.md) | PO | 18/10 |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: página, frase de visão, público, problema e proposta. Do Tech Lead: frase e tabela de LGPD (C14), riscos e questões Q1 a Q6 para decisão da PO. |
| 1.1 | 01/11/2026 | @Lelafeta | Definição padrão de retenção de 90 dias com expurgo automático pós-contrato. |
