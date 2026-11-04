# Backlog

Backlog do Habilita com priorização MoSCoW. Os critérios de aceite detalhados de cada história estão em [Histórias de Usuário e Backlog](../produto/requisitos/historias-backlog.md).

As histórias abaixo cobrem as 5 jornadas ponta a ponta do MVP — **J1** (criar contratos e a matriz de exigências), **J2** (cadastrar trabalhadores e enviar documentos), **J3** (aprovar ou rejeitar documentos, com motivo), **J4** (atualizar vencimentos, status e alertas) e **J5** (consultar a liberação e registrar o acesso na portaria) — detalhadas em [Mapeamento de Jornadas](../produto/estrategia/jornadas.md).

| ID | História | MoSCoW | RF | Critério de aceite |
|---|---|---|---|---|
| H01 | Como gestor(a) de SST, quero definir os documentos exigidos por contrato, para padronizar a conferência | Must | RF01 | Contrato com `dataInicio ≤ dataFim`; matriz de 1 a 20 tipos; semáforo recalculado em até 60 s — ver [H01](../produto/requisitos/historias-backlog.md#h01) |
| H02 | Como terceirizada, quero enviar os documentos dos meus trabalhadores, para que sejam liberados | Must | RF02, RF03 | CPF válido e único por cliente; PDF/JPG até 5 MB; validade obrigatória; 413/415 fora disso — ver [H02](../produto/requisitos/historias-backlog.md#h02) |
| H03 | Como gestor(a), quero rejeitar um documento informando o motivo, para que a terceirizada corrija | Must | RF04 | Motivo de 10 a 500 caracteres; sem motivo → 400 e documento continua pendente — ver [H03](../produto/requisitos/historias-backlog.md#h03) |
| H04 | Como porteiro(a), quero consultar se o trabalhador está liberado, para autorizar a entrada | Must | RF09 | LIBERADO / BLOQUEADO / NAO_CADASTRADO; p95 ≤ 800 ms; CPF fora da URL; toda consulta auditada — ver [H04](../produto/requisitos/historias-backlog.md#h04) |
| H05 | Como gestor(a), quero ser avisado(a) de vencimentos próximos, para agir antes do bloqueio | Must | RF07 | 1 alerta por documento e faixa (30 dias, vencido); sem duplicar em reexecuções — ver [H05](../produto/requisitos/historias-backlog.md#h05) |
| H07 | Como usuário cadastrado (gestor, terceirizada ou porteiro), quero entrar com e-mail e senha e ver apenas as telas do meu papel, para trabalhar sem acessar o que não me cabe. | Must | — (RNF01) | ver [H07](../produto/requisitos/historias-backlog.md#h07) |
| H08 | Como gestor, quero cadastrar as empresas terceirizadas com quem tenho contrato, para vinculá-las aos contratos. | Must | RF01 | ver [H08](../produto/requisitos/historias-backlog.md#h08) |
| H09 | Como gestor, quero cadastrar os tipos de documento que exijo (ex.: ASO, NR-35) com o prazo de validade padrão, para que o sistema calcule os vencimentos. | Must | RF01 | ver [H09](../produto/requisitos/historias-backlog.md#h09) |
| H10 | Como responsável da terceirizada, quero vincular um trabalhador a um contrato da minha empresa, para que ele possa ser liberado nesse contrato. | Must | RF02 | ver [H10](../produto/requisitos/historias-backlog.md#h10) |
| H11 | Como responsável da terceirizada, quero que um documento já aprovado e válido conte automaticamente para qualquer contrato do mesmo cliente que o exija, para não reenviar o mesmo arquivo. | Must | RF08 | ver [H11](../produto/requisitos/historias-backlog.md#h11) |
| H12 | Como gestor, quero ver todos os documentos pendentes e abrir cada arquivo, para decidir sobre eles. | Must | RF04 | ver [H12](../produto/requisitos/historias-backlog.md#h12) |
| H13 | Como gestor, quero aprovar um documento confirmando sua data de validade, para que ele conte no semáforo. | Must | RF04 | ver [H13](../produto/requisitos/historias-backlog.md#h13) |
| H14 | Como responsável da terceirizada, quero ver por que um documento foi rejeitado e enviar um novo, para regularizar o trabalhador. | Must | RF03, RF04 | ver [H14](../produto/requisitos/historias-backlog.md#h14) |
| H15 | Como gestor, quero que o sistema recalcule sozinho a situação de todos os documentos todo dia, para não depender de ninguém lembrar. | Must | RF05 | ver [H15](../produto/requisitos/historias-backlog.md#h15) |
| H16 | Como gestor, quero ver o semáforo de cada trabalhador em cada contrato, para saber quem está apto. | Must | RF06 | ver [H16](../produto/requisitos/historias-backlog.md#h16) |
| H17 | Como gestor, quero receber um resumo diário dos alertas por e-mail, para não precisar abrir o sistema. | Could | RF07 | ver [H17](../produto/requisitos/historias-backlog.md#h17) |
| H18 | Como porteiro, quero confirmar se o trabalhador entrou ou foi barrado, para que fique registrado quem acessou o local. | Must | RF09 | ver [H18](../produto/requisitos/historias-backlog.md#h18) |
| H19 | Como gestor de um cliente, quero ter a garantia de que nenhum outro cliente vê meus dados, para confiar meus documentos ao Habilita. | Must | — (RNF01) | ver [H19](../produto/requisitos/historias-backlog.md#h19) |
| H20 | Como gestor, quero que toda decisão e todo acesso fiquem registrados de forma imutável, para responder a uma auditoria ou a um incidente. | Must | RF10 | ver [H20](../produto/requisitos/historias-backlog.md#h20) |
| H21 | Como gestor, quero filtrar a trilha por período, ação e recurso, para investigar uma decisão. | Should | RF10 | ver [H21](../produto/requisitos/historias-backlog.md#h21) |
| H22 | Como gestor, quero exportar o semáforo de um contrato em CSV, para anexar a relatórios. | Could | RF06 | ver [H22](../produto/requisitos/historias-backlog.md#h22) |
| H23 | Como gestor, quero ver, por contrato, quantos trabalhadores estão em cada cor e quantos documentos vencem nos próximos 30 dias, para priorizar ações. | Must | RF06 | ver [H23](../produto/requisitos/historias-backlog.md#h23) |

!!! note "Legenda MoSCoW"
    **Must** = sem isso não há MVP · **Should** = importante · **Could** = se der tempo · **Won't** = fora deste ciclo.

## Won't — fora deste ciclo

Itens fora do MVP, com o motivo. Detalhes em [Histórias de Usuário e Backlog](../produto/requisitos/historias-backlog.md#wont).

| Item | Motivo |
|---|---|
| Leitura automática de documentos (OCR) | Fora da ficha; custo e complexidade incompatíveis com 9 semanas |
| Integração com o eSocial | Fora da ficha; Habilita não é integrador do eSocial |
| Assinatura digital ICP-Brasil | Fora da ficha |
| App nativo iOS ou Android | Fora da ficha; a portaria usa navegador (web responsiva) |
| Pagamentos ou cobrança | Fora da ficha |
| Integração com catraca ou biometria | Fora da classificação "Não Faz" |
| Verificar autenticidade do documento junto à clínica ou ao órgão emissor | Fora da classificação "Não Faz"; a aprovação humana é o controle no MVP |
| Cadastro self-service de novos clientes e usuários | Clientes e usuários entram por seed; reduz superfície de ataque |
| Reuso de cadastro e documentos entre contratantes diferentes (efeito de rede) | LGPD e isolamento por tenant. Exigiria consentimento da prestadora e um cadastro fora do tenant. O MVP prova o reuso dentro de um mesmo cliente (H11) |
| Foto do trabalhador na consulta de portaria | Dado biométrico, aumenta o risco LGPD sem ser essencial ao MVP |

## Revisões do backlog

| Data | O que mudou | Por quê |
|---|---|---|
| 02/11/2026 | H05, H14 e H23 para Must (H23 implementada após o CP2); H07 a H23 validadas; H06 removida; critérios de upload (5 MB, PDF/JPG) e painel (500 trabalhadores em ≤ 2 s) | Feedback do Checkpoint 1, itens 3, 5 e 8 |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 18/10/2026 | @Lelafeta | Versão inicial: página de índice do backlog com links para Histórias de Usuário e Requisitos Funcionais. |
| 1.1 | 02/11/2026 | @Lelafeta | Inclusão da tabela do backlog MoSCoW e do registro de revisões semanais do backlog. |
| 1.2 | 02/11/2026 | @Lelafeta | Inclusão do parágrafo no topo listando as jornadas J1 a J5 com link para o Mapeamento de Jornadas. |
| 1.3 | 03/11/2026 | @Lelafeta | Inclusão da seção Won't com a tabela de itens fora deste ciclo. |
