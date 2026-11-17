# Histórias de Usuário e Backlog (MoSCoW)

!!! info "Sobre esta página"
    **Para que serve:** Transformar os requisitos em histórias priorizadas. **Backlog MoSCoW é obrigatório no CP1.**

    **O que escrever:** Formato: *Como [persona], quero [ação], para [benefício]*. Classificar em Must / Should / Could / Won't.

    **Prazo:** CP1 · 18/10

!!! note "Origem das histórias"
    **H01 a H06** são da PO (texto e prioridade mantidos). **H07 a H23** foram propostas pelo Tech Lead para cobrir as 5 jornadas e os requisitos funcionais por completo; a PO valida a prioridade. Critérios de aceite detalhados abaixo da tabela, no formato Dado / Quando / Então.

| ID | História | MoSCoW | RF | Critério de aceite |
|---|---|---|---|---|
| H01 | Como gestor(a) de SST, quero definir os documentos exigidos por contrato, para padronizar a conferência | Must | RF01 | Contrato com `dataInicio ≤ dataFim`; matriz de 1 a 20 tipos; semáforo recalculado em até 60 s — ver [H01](#h01) |
| H02 | Como terceirizada, quero enviar os documentos dos meus trabalhadores, para que sejam liberados | Must | RF02, RF03 | CPF válido e único por cliente; PDF/JPG até 5 MB; validade obrigatória; 413/415 fora disso — ver [H02](#h02) |
| H03 | Como gestor(a), quero rejeitar um documento informando o motivo, para que a terceirizada corrija | Must | RF04 | Motivo de 10 a 500 caracteres; sem motivo → 400 e documento continua pendente — ver [H03](#h03) |
| H04 | Como porteiro(a), quero consultar se o trabalhador está liberado, para autorizar a entrada | Must | RF09 | LIBERADO / BLOQUEADO / NAO_CADASTRADO; p95 ≤ 800 ms; CPF fora da URL; toda consulta auditada — ver [H04](#h04) |
| H05 | Como gestor(a), quero ser avisado(a) de vencimentos próximos, para agir antes do bloqueio | Should | RF07 | 1 alerta por documento e faixa (30 dias, vencido); sem duplicar em reexecuções — ver [H05](#h05) |
| H06 | | Won't | — | OCR, eSocial, ICP-Brasil, app nativo, pagamentos (lista completa em [Won't](#wont)) |
| H07 | Como usuário cadastrado (gestor, terceirizada ou porteiro), quero entrar com e-mail e senha e ver apenas as telas do meu papel, para trabalhar sem acessar o que não me cabe. | Must | — (RNF01) | ver [H07](#h07) |
| H08 | Como gestor, quero cadastrar as empresas terceirizadas com quem tenho contrato, para vinculá-las aos contratos. | Must | RF01 | ver [H08](#h08) |
| H09 | Como gestor, quero cadastrar os tipos de documento que exijo (ex.: ASO, NR-35) com o prazo de validade padrão, para que o sistema calcule os vencimentos. | Must | RF01 | ver [H09](#h09) |
| H10 | Como responsável da terceirizada, quero vincular um trabalhador a um contrato da minha empresa, para que ele possa ser liberado nesse contrato. | Must | RF02 | ver [H10](#h10) |
| H11 | Como responsável da terceirizada, quero que um documento já aprovado e válido conte automaticamente para qualquer contrato do mesmo cliente que o exija, para não reenviar o mesmo arquivo. | Must | RF08 | ver [H11](#h11) |
| H12 | Como gestor, quero ver todos os documentos pendentes e abrir cada arquivo, para decidir sobre eles. | Must | RF04 | ver [H12](#h12) |
| H13 | Como gestor, quero aprovar um documento confirmando sua data de validade, para que ele conte no semáforo. | Must | RF04 | ver [H13](#h13) |
| H14 | Como responsável da terceirizada, quero ver por que um documento foi rejeitado e enviar um novo, para regularizar o trabalhador. | Should | RF03, RF04 | ver [H14](#h14) |
| H15 | Como gestor, quero que o sistema recalcule sozinho a situação de todos os documentos todo dia, para não depender de ninguém lembrar. | Must | RF05 | ver [H15](#h15) |
| H16 | Como gestor, quero ver o semáforo de cada trabalhador em cada contrato, para saber quem está apto. | Must | RF06 | ver [H16](#h16) |
| H17 | Como gestor, quero receber um resumo diário dos alertas por e-mail, para não precisar abrir o sistema. | Could | RF07 | ver [H17](#h17) |
| H18 | Como porteiro, quero confirmar se o trabalhador entrou ou foi barrado, para que fique registrado quem acessou o local. | Must | RF09 | ver [H18](#h18) |
| H19 | Como gestor de um cliente, quero ter a garantia de que nenhum outro cliente vê meus dados, para confiar meus documentos ao Habilita. | Must | — (RNF01) | ver [H19](#h19) |
| H20 | Como gestor, quero que toda decisão e todo acesso fiquem registrados de forma imutável, para responder a uma auditoria ou a um incidente. | Must | RF10 | ver [H20](#h20) |
| H21 | Como gestor, quero filtrar a trilha por período, ação e recurso, para investigar uma decisão. | Should | RF10 | ver [H21](#h21) |
| H22 | Como gestor, quero exportar o semáforo de um contrato em CSV, para anexar a relatórios. | Could | RF06 | ver [H22](#h22) |
| H23 | Como gestor, quero ver, por contrato, quantos trabalhadores estão em cada cor e quantos documentos vencem nos próximos 30 dias, para priorizar ações. | Should | RF06 | ver [H23](#h23) |

!!! note "Legenda MoSCoW"
    **Must** = sem isso não há MVP · **Should** = importante · **Could** = se der tempo · **Won't** = fora deste ciclo.

!!! warning "Questão Q5 para a PO"
    H05 (alertas) está como **Should**, mas a jornada J4 da ficha do projeto ("Atualizar vencimentos, status e alertas") faz parte do MVP. O Tech Lead sugere **Must**. Decisão da PO.

## Regras de escrita dos critérios

1. Todo critério de aceite (CA) é testável: tem número, limite ou resultado verificável.
2. O ID do critério é `HNN.CAn` e aparece no nome do teste que o cobre (DoD).
3. Regras de negócio (`RN-NN`) estão no [Modelo de dados](../../arquitetura/dados.md); códigos HTTP seguem a [API](../../arquitetura/api.md).

## Histórias da PO

### H01 · Definir documentos exigidos por contrato — **Must** { #h01 }

*Como gestor(a) de SST, quero definir os documentos exigidos por contrato, para padronizar a conferência.* (RF01)

Inclui criar o contrato e definir sua matriz de exigências.

- **CA1** Dado uma terceirizada existente, um identificador de 1 a 40 caracteres e `dataInicio` ≤ `dataFim`, quando salvo, então o contrato é criado (201) com status `VIGENTE` se hoje estiver no intervalo.
- **CA2** Dado `dataFim` anterior a `dataInicio`, quando salvo, então a API responde 400.
- **CA3** Dado um contrato com `dataFim` no passado, quando o semáforo é calculado, então seus trabalhadores não contam como liberados por ele (RN-10).
- **CA4** Dado um contrato e uma lista de 1 a 20 tipos de documento existentes, quando salvo a matriz, então ela substitui a anterior e o semáforo dos trabalhadores do contrato é recalculado em até 60 segundos.
- **CA5** Dado um tipo de documento de outro tenant ou inexistente, quando salvo, então a API responde 400.
- **CA6** Dado uma alteração de matriz, quando concluída, então a versão anterior e a nova ficam registradas na trilha de auditoria.

### H02 · Enviar documentos dos trabalhadores — **Must** { #h02 }

*Como terceirizada, quero enviar os documentos dos meus trabalhadores, para que sejam liberados.* (RF02, RF03)

Inclui cadastrar o trabalhador e enviar cada documento com a data de validade (RF03).

- **CA1** Dado nome com 3 a 120 caracteres e CPF com dígitos verificadores válidos, quando salvo, então o trabalhador é criado (201) e a resposta mostra apenas o CPF mascarado (`***.456.789-**`).
- **CA2** Dado um CPF inválido (incluindo sequências repetidas como `111.111.111-11`), quando salvo, então a API responde 400.
- **CA3** Dado um CPF já cadastrado no mesmo tenant, quando salvo, então a API responde 409.
- **CA4** O CPF completo não é persistido; é guardado apenas o hash HMAC e a versão mascarada (ver [ADR 0008](../../arquitetura/adr/0008-consulta-liberacao-cpf.md)).
- **CA5** Dado um arquivo PDF ou JPG com até 5 MB (5.000.000 bytes), quando envio, então o documento é criado com status `PENDENTE` (201).
- **CA6** Dado um arquivo com 5.000.001 bytes ou mais, quando envio, então a API responde 413.
- **CA7** Dado um arquivo cujo conteúdo real (assinatura de bytes) não é PDF ou JPG, mesmo com extensão `.pdf`, quando envio, então a API responde 415.
- **CA8** Dado uma `dataEmissao` no futuro, quando envio, então a API responde 400.
- **CA9** O arquivo fica em bucket privado; nenhum usuário consegue baixá-lo sem passar pela API (ver H12.CA3).
- **CA10** Dado um envio sem `validoAte`, ou com `validoAte` anterior à `dataEmissao`, quando envio, então a API responde 400 (RN-09).
- **CA11** Dado um arquivo recusado por tamanho (413) ou por tipo (415), quando envio, então a tela mostra uma mensagem que diz o limite: "Envie um PDF ou JPG de até 5 MB".

### H03 · Rejeitar documento com motivo obrigatório — **Must** { #h03 }

*Como gestor(a), quero rejeitar um documento informando o motivo, para que a terceirizada corrija.* (RF04)

- **CA1** Dado um documento `PENDENTE` e um motivo com 10 a 500 caracteres, quando rejeito, então o status muda para `REJEITADO` e o motivo é gravado.
- **CA2** Dado motivo ausente, vazio, só com espaços ou com menos de 10 caracteres, quando rejeito, então a API responde 400 e o documento continua `PENDENTE`.
- **CA3** Dado motivo com mais de 500 caracteres, quando rejeito, então a API responde 400.
- **CA4** Dado um documento que não está `PENDENTE`, quando tento rejeitar, então a API responde 409.

### H04 · Consultar liberação por CPF na portaria — **Must** { #h04 }

*Como porteiro(a), quero consultar se o trabalhador está liberado, para autorizar a entrada.* (RF09)

- **CA1** Dado um trabalhador verde ou amarelo em pelo menos um contrato vigente do meu tenant, quando consulto o CPF, então a resposta é `LIBERADO`, com o nome do trabalhador e os contratos em que está apto (sem foto; ver *Won't*).
- **CA2** Dado um trabalhador vermelho em todos os contratos vigentes, então a resposta é `BLOQUEADO` com a orientação "Procure o SST", sem tipo de documento, motivo ou contrato. O detalhe das pendências aparece só para o gestor.
- **CA3** Dado um CPF não cadastrado no meu tenant (inclusive se existir em outro tenant), então a resposta é `NAO_CADASTRADO`, sem nenhum dado do outro tenant.
- **CA4** O CPF é enviado no corpo da requisição, nunca na URL (ver [ADR 0008](../../arquitetura/adr/0008-consulta-liberacao-cpf.md)).
- **CA5** A consulta responde em p95 ≤ 800 ms com 1.000 trabalhadores no tenant (RNF04).
- **CA6** Toda consulta gera 1 registro de auditoria com o resultado.

### H05 · Ver alertas de vencimento — **Should** { #h05 }

*Como gestor(a), quero ser avisado(a) de vencimentos próximos, para agir antes do bloqueio.* (RF07)

- **CA1** Dado um documento que entra na faixa `A_VENCER`, quando o job roda, então é criado 1 alerta do tipo `VENCE_EM_30_DIAS`.
- **CA2** Dado um documento que entra na faixa `VENCIDO`, quando o job roda, então é criado 1 alerta do tipo `VENCIDO`.
- **CA3** Dado o mesmo documento na mesma faixa, quando o job roda em dias seguintes, então nenhum alerta novo é criado.
- **CA4** Dado um alerta, quando o marco como lido, então ele sai da lista de não lidos e o registro permanece.

### H06 · Won't { #h06 }

OCR, eSocial, ICP-Brasil, app nativo, pagamentos. Lista completa e motivos em [Won't](#wont).

## Histórias propostas pelo Tech Lead

### H07 · Entrar no sistema com o meu papel — **Must** { #h07 }

*Como* usuário cadastrado (gestor, terceirizada ou porteiro), *quero* entrar com e-mail e senha e ver apenas as telas do meu papel, *para* trabalhar sem acessar o que não me cabe.

- **CA1** Dado um usuário com papel `porteiro`, quando ele entra, então a tela inicial é a consulta de liberação e as rotas de gestão respondem 403.
- **CA2** Dado um token ausente, quando qualquer rota protegida é chamada, então a API responde 401 com corpo `application/problem+json`.
- **CA3** Dado um token expirado ou com assinatura inválida, quando qualquer rota protegida é chamada, então a API responde 401.
- **CA4** Dado um usuário sem as claims `tenantId` e `papel`, quando ele chama a API, então a resposta é 403 e o evento é registrado em log sem dados pessoais.

> Cadastro de usuários e de clientes no MVP é feito por script de seed (ver *Won't*: self-service).

### H08 · Cadastrar empresa terceirizada — **Must** { #h08 }

*Como* gestor, *quero* cadastrar as empresas terceirizadas com quem tenho contrato, *para* vinculá-las aos contratos.

- **CA1** Dado um CNPJ válido (dígitos verificadores corretos) e razão social com 3 a 150 caracteres, quando salvo, então a terceirizada é criada e a API responde 201.
- **CA2** Dado um CNPJ com dígito verificador inválido, quando salvo, então a API responde 400 indicando o campo `cnpj`.
- **CA3** Dado um CNPJ já cadastrado no mesmo tenant, quando salvo, então a API responde 409.

### H09 · Cadastrar tipos de documento com validade — **Must** { #h09 }

*Como* gestor, *quero* cadastrar os tipos de documento que exijo (ex.: ASO, NR-35) com o prazo de validade padrão, *para* que o sistema calcule os vencimentos.

- **CA1** Dado um nome de 2 a 80 caracteres e validade entre 1 e 120 meses, quando salvo, então o tipo é criado (201).
- **CA2** Dado validade 0 ou 121 meses, quando salvo, então a API responde 400.
- **CA3** Dado um tipo com prazo padrão de N meses, quando a terceirizada informa `dataEmissao` no envio, então a tela sugere `validoAte = dataEmissao + N meses`; se a validade informada ultrapassar esse padrão, o gestor vê um aviso ao aprovar (RN-09).

### H10 · Vincular trabalhador a contrato — **Must** { #h10 }

*Como* responsável da terceirizada, *quero* vincular um trabalhador a um contrato da minha empresa, *para* que ele possa ser liberado nesse contrato.

- **CA1** Dado um trabalhador e um contrato da mesma terceirizada, quando vinculo, então o vínculo é criado (201) e o semáforo do trabalhador no contrato é calculado imediatamente.
- **CA2** Dado um contrato de outra terceirizada, quando tento vincular, então a API responde 404.
- **CA3** Dado um vínculo já existente, quando tento vincular de novo, então a API responde 409.

### H11 · Reaproveitar documento válido em outro contrato — **Must** { #h11 }

*Como* responsável da terceirizada, *quero* que um documento já aprovado e válido conte automaticamente para qualquer contrato do mesmo cliente que o exija, *para* não reenviar o mesmo arquivo.

- **CA1** Dado um trabalhador com ASO aprovado e válido, quando ele é vinculado a um segundo contrato do mesmo tenant que exige ASO, então a exigência aparece como atendida sem novo upload.
- **CA2** Dado um trabalhador habilitado (verde) no Contrato A e um Contrato B que exige apenas tipos já aprovados e válidos, quando ele é vinculado a B, então o semáforo em B é verde em até 60 segundos.
- **CA3** O reuso nunca atravessa tenants: um documento do tenant A não atende exigência do tenant B (RN-08).

---

### H12 · Ver a fila de documentos pendentes — **Must** { #h12 }

*Como* gestor, *quero* ver todos os documentos pendentes e abrir cada arquivo, *para* decidir sobre eles.

- **CA1** Dado documentos pendentes, quando abro a fila, então vejo no máximo 50 por página, do mais antigo para o mais novo.
- **CA2** Dado uma fila vazia, quando abro, então vejo o estado vazio com a mensagem "Nenhum documento aguardando análise".
- **CA3** Dado um documento, quando peço para visualizar, então recebo um link temporário válido por no máximo 5 minutos; após esse prazo o link responde erro.

### H13 · Aprovar documento — **Must** { #h13 }

*Como* gestor, *quero* aprovar um documento confirmando sua data de validade, *para* que ele conte no semáforo.

- **CA1** Dado um documento `PENDENTE`, quando aprovo, então o status muda para `APROVADO`, o `validoAte` informado no envio (ou corrigido pelo gestor) é confirmado (RN-09) e o semáforo do trabalhador é recalculado em até 60 segundos.
- **CA2** Dado um documento que não está `PENDENTE`, quando tento aprovar, então a API responde 409.
- **CA3** Dado um `validoAte` anterior à `dataEmissao`, quando aprovo, então a API responde 400.

### H14 · Ver motivo da rejeição e reenviar — **Should** { #h14 }

*Como* responsável da terceirizada, *quero* ver por que um documento foi rejeitado e enviar um novo, *para* regularizar o trabalhador.

- **CA1** Dado um documento rejeitado, quando abro o trabalhador, então vejo o motivo exato registrado pelo gestor.
- **CA2** Dado um novo envio do mesmo tipo, quando concluído, então um novo documento `PENDENTE` é criado e o rejeitado permanece no histórico, inalterado.

---

### H15 · Recalcular vencimentos diariamente — **Must** { #h15 }

*Como* gestor, *quero* que o sistema recalcule sozinho a situação de todos os documentos todo dia, *para* não depender de ninguém lembrar.

- **CA1** Dado o job executado para a data de referência D, quando ele é executado 2 vezes seguidas para a mesma D, então o estado final (situações, semáforos e alertas) é idêntico e nenhum alerta é duplicado.
- **CA2** Dado um documento com `validoAte` = D + 31, D + 30 e D + 29, quando o job roda em D, então as situações são `VALIDO`, `A_VENCER` e `A_VENCER`, respectivamente (RN-04).
- **CA3** Dado `validoAte` = D e D − 1, quando o job roda em D, então as situações são `A_VENCER` e `VENCIDO`.
- **CA4** Dado uma falha no meio da execução, quando o job é executado de novo, então ele conclui sem efeitos duplicados.
- **CA5** O endpoint do job só aceita chamadas autenticadas da conta de serviço do agendador; qualquer outra chamada recebe 401 ou 403.

### H16 · Ver semáforo por trabalhador e contrato — **Must** { #h16 }

*Como* gestor, *quero* ver o semáforo de cada trabalhador em cada contrato, *para* saber quem está apto.

- **CA1** Dado todas as exigências obrigatórias atendidas por documentos com situação `VALIDO`, então o semáforo é **verde**.
- **CA2** Dado todas atendidas e pelo menos uma `A_VENCER`, então é **amarelo**.
- **CA3** Dado pelo menos uma exigência sem documento aprovado, ou atendida só por documento `VENCIDO`, então é **vermelho** (RN-06).
- **CA4** O semáforo do contrato mostra a contagem de trabalhadores por cor e assume a pior cor entre eles (RN-07).

### H17 · Receber alerta por e-mail — **Could** { #h17 }

*Como* gestor, *quero* receber um resumo diário dos alertas por e-mail, *para* não precisar abrir o sistema.

- **CA1** Dado ao menos 1 alerta novo no dia, quando o job termina, então 1 e-mail de resumo é enviado ao gestor do tenant; com 0 alertas, nenhum e-mail é enviado.
- **CA2** O e-mail não contém CPF nem anexo, apenas contagens e link para o sistema.

### H18 · Registrar entrada ou barramento — **Must** { #h18 }

*Como* porteiro, *quero* confirmar se o trabalhador entrou ou foi barrado, *para* que fique registrado quem acessou o local.

- **CA1** Dado uma consulta com resultado `LIBERADO` feita há no máximo 10 minutos, quando registro `ENTRADA`, então o acesso é gravado (201).
- **CA2** Dado uma consulta com resultado `BLOQUEADO` ou `NAO_CADASTRADO`, quando tento registrar `ENTRADA`, então a API responde 422; só `BARRADO` é aceito.
- **CA3** Dado uma consulta com mais de 10 minutos, quando registro, então a API responde 409 e exige nova consulta.
- **CA4** Dado uma consulta já usada em um registro, quando registro de novo, então a API responde 409.

---

### H19 · Isolar dados por cliente e papel — **Must** { #h19 }

*Como* gestor de um cliente, *quero* ter a garantia de que nenhum outro cliente vê meus dados, *para* confiar meus documentos ao Habilita.

- **CA1** Dado um usuário do tenant A, quando ele pede por ID qualquer recurso do tenant B (contrato, trabalhador, documento, alerta), então a API responde 404, sem revelar que o recurso existe.
- **CA2** Dado um usuário com papel sem permissão para a rota (matriz em [Estratégia de Testes](../../testes/estrategia.md)), quando ele a chama, então a API responde 403.
- **CA3** Dado um usuário `terceirizada` da empresa X, quando ele lista trabalhadores ou documentos, então só recebe itens da empresa X.
- **CA4** Dado qualquer cliente Firebase (navegador) acessando o Firestore ou o Storage diretamente, quando lê ou escreve qualquer caminho, então a operação é negada (Security Rules deny-all).
- **CA5** O `tenantId` usado em toda consulta vem da claim do token; um `tenantId` enviado no corpo, na query ou no cabeçalho é ignorado.

> **Checkpoint 1:** o teste de isolamento (CA1) precisa estar **escrito**, mesmo que falhe. Especificação em [Estratégia de Testes](../../testes/estrategia.md).

### H20 · Registrar trilha de auditoria — **Must** { #h20 }

*Como* gestor, *quero* que toda decisão e todo acesso fiquem registrados de forma imutável, *para* responder a uma auditoria ou a um incidente.

- **CA1** Dado qualquer operação de escrita (criar, aprovar, rejeitar, alterar matriz, registrar acesso) ou consulta de liberação, quando ela termina com sucesso, então existe exatamente 1 registro de auditoria com `ator`, `papel`, `acao`, `recurso`, `dataHora` (UTC) e `requestId`.
- **CA2** Não existe rota de API que altere ou apague registros de auditoria; qualquer tentativa de escrita direta no Firestore é negada.
- **CA3** O registro de auditoria não contém CPF completo, conteúdo de documento nem token.

### H21 · Consultar a trilha de auditoria — **Should** { #h21 }

*Como* gestor, *quero* filtrar a trilha por período, ação e recurso, *para* investigar uma decisão.

- **CA1** Dado um filtro de até 31 dias, quando consulto, então recebo resultados paginados de no máximo 50 itens por página, em ordem decrescente de data.
- **CA2** Dado um usuário `terceirizada` ou `porteiro`, quando consulta a trilha, então recebe 403.

---

### H22 · Exportar relatório de conformidade em CSV — **Could** { #h22 }

*Como* gestor, *quero* exportar o semáforo de um contrato em CSV, *para* anexar a relatórios.

- **CA1** Dado um contrato com até 500 trabalhadores, quando exporto, então recebo um CSV UTF-8 com uma linha por trabalhador e colunas nome, CPF mascarado, semáforo e próximo vencimento.

### H23 · Ver painel resumido do contrato — **Should** { #h23 }

*Como* gestor, *quero* ver, por contrato, quantos trabalhadores estão em cada cor e quantos documentos vencem nos próximos 30 dias, *para* priorizar ações.

- **CA1** Dado um contrato, quando abro o painel, então os totais por cor batem com a lista de trabalhadores do contrato.
- **CA2** O painel carrega em até 2 segundos (p95) com 500 trabalhadores de seed no contrato, lendo apenas o documento `contagens/{contratoId}`.

---

## Won't — fora deste ciclo { #wont }


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
| Um mesmo login de terceirizada atendendo vários clientes | No MVP, cada login pertence a um único tenant |
| Foto do trabalhador na consulta de portaria | Dado biométrico, aumenta o risco LGPD sem ser essencial ao MVP |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: histórias H01 a H06, com texto e prioridade mantidos. Do Tech Lead: histórias H07 a H23 propostas, critérios de aceite Dado/Quando/Então e lista Won't detalhada (C4, C9). |
| 1.1 | 01/11/2026 | @Lelafeta | Alinhamento de limites de upload (5 MB PDF/JPG), resposta da portaria e carga do painel ao Caderno. |
