# Defensibilidade e Métrica da Demo

!!! info "Sobre esta página"
    **Para que serve:** Responder: por que alguém usaria o Habilita e não uma planilha? E como vamos provar isso no Demo Day?

    **O que escrever:** Uma hipótese de defensibilidade e **uma métrica mensurável** para a demo. Ambas obrigatórias no CP1.

    **Prazo:** CP1 · 18/10

| Item | Conteúdo |
|---|---|
| Hipótese de defensibilidade | Reaproveitar documentos válidos entre contratos reduz retrabalho. Cada documento aprovado passa a valer para todos os contratos do cliente que o exigem; quanto mais contratos, menos trabalho a cada novo contrato, e trocar de ferramenta significa recomeçar do zero. O MVP prova o reuso dentro de um mesmo cliente; o efeito de rede entre contratantes é a evolução (ver Won't no Backlog). |
| Métrica da demo | **Opção B, principal (Q6, decidida em 01/11/2026):** taxa de reuso na habilitação. **Opção A, complementar:** tempo para liberar um trabalhador na portaria. Ver abaixo. |
| Valor atual (sem o Habilita) | A medir nas entrevistas (pergunta P5 e P7 do roteiro) |
| Meta com o Habilita | Opção A: p95 ≤ 800 ms na consulta. Opção B: ≥ 70% das exigências atendidas por reuso e semáforo verde em até 2 minutos |

## Opção A detalhada — tempo para liberar um trabalhador na portaria

> Dado um trabalhador cadastrado, medir (1) o tempo de resposta da consulta por CPF e (2) se a decisão (`LIBERADO` ou `BLOQUEADO`) está correta.

**Por que a PO propôs a Opção A:** a portaria é a ponta mais visível do problema: hoje ela confere listas em papel e libera sem verificar ou barra quem estava em dia (HP3). A Opção A mostra o efeito prático do Habilita: o porteiro digita o CPF, recebe uma resposta direta e o acesso fica registrado.

**Metas:** p95 ≤ 800 ms com 1.000 trabalhadores (H04.CA5), comprovado por teste de carga; 100% dos trabalhadores com documento vencido ou reprovado retornam `BLOQUEADO`; 1 registro de auditoria por consulta (H20).

**Cena da demo:** consultar um CPF verde, um amarelo e um vermelho; os dois primeiros retornam `LIBERADO` e o terceiro `BLOQUEADO`, com o acesso registrado na trilha.

**Limite:** mede velocidade e acerto, que uma planilha bem organizada também poderia oferecer; por isso não prova sozinha a defensibilidade e funciona como métrica complementar à Opção B.

## Opção B detalhada — taxa de reuso na habilitação

> Dado um trabalhador já habilitado no Contrato A, ao vinculá-lo ao Contrato B, medir (1) a porcentagem das exigências de B atendidas sem novo upload e (2) o tempo até o semáforo ficar verde.

**Por que o Tech Lead sugere a Opção B como principal:** a métrica da demo deve provar a **hipótese de defensibilidade**. A Opção A mede velocidade da portaria, que uma planilha compartilhada bem organizada também poderia oferecer; a Opção B mede exatamente o reaproveitamento entre contratos, que é a hipótese desta página e a cena da demo da ficha do projeto. As duas podem ser mostradas; a decisão de qual é a principal é da PO.

## Outras fontes de defensibilidade

- **Matrizes de exigência configuradas por contrato**, que codificam o conhecimento de SST do cliente.
- **Trilha de auditoria contínua** (quem aprovou, quem entrou), que tem valor justamente por ser ininterrupta.

As funcionalidades isoladas (upload, semáforo, alerta) são copiáveis; a defesa está no que se acumula com o uso.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: página e métrica de exemplo (tempo de liberação na portaria, Opção A). Do Tech Lead: hipótese de reuso e métrica Opção B, para decisão da PO (C3). |
| 1.1 | 18/10/2026 | @Lelafeta | Detalhamento da Opção A (tempo de liberação na portaria) com justificativa da PO, metas mensuráveis, cena da demo e limites. |
| 1.2 | 01/11/2026 | @Lelafeta | Registro da Opção B como métrica principal da demo e complementação da hipótese de defensibilidade. |
