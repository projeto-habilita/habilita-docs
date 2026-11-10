# Requisitos Funcionais

!!! info "Sobre esta página"
    **Para que serve:** Listar **o que** o sistema faz. Cada requisito precisa de um critério de aceite mensurável e depois de um teste.

    **O que escrever:** Um RF por linha, ligado a uma jornada. Completar critérios de aceite.

    **Prazo:** CP1 · 18/10

| ID | Requisito | Jornada | Critério de aceite |
|---|---|---|---|
| RF01 | Criar contrato com matriz de documentos exigidos | J1 | Contrato salvo só com `dataInicio ≤ dataFim` (senão 400); matriz com 1 a 20 tipos de documento do próprio cliente; ao alterar a matriz, semáforo do contrato recalculado em até 60 s. Detalhe: [H01](historias-backlog.md#h01), [H08](historias-backlog.md#h08), [H09](historias-backlog.md#h09) |
| RF02 | Cadastrar trabalhador vinculado a uma terceirizada | J2 | CPF com dígitos verificadores válidos e único por cliente (senão 400 / 409); resposta mostra só o CPF mascarado; terceirizada só cadastra e vincula os próprios trabalhadores (senão 404). Detalhe: [H02](historias-backlog.md#h02), [H10](historias-backlog.md#h10) |
| RF03 | Enviar documento com data de validade | J2 | PDF ou JPG de até 5.000.000 bytes (5 MB) (acima → 413; conteúdo de outro tipo → 415); `validoAte` obrigatória e não anterior à `dataEmissao` (senão 400); documento criado como `PENDENTE`. Detalhe: [H02](historias-backlog.md#h02) |
| RF04 | Aprovar ou rejeitar documento, com motivo obrigatório na rejeição | J3 | Rejeição exige motivo de 10 a 500 caracteres (senão 400 e o documento continua `PENDENTE`); só documento `PENDENTE` pode ser decidido (senão 409); toda decisão gera registro de auditoria. Detalhe: [H03](historias-backlog.md#h03), [H12](historias-backlog.md#h12), [H13](historias-backlog.md#h13) |
| RF05 | Recalcular status e vencimentos sem duplicar efeitos | J4 | Executar o recálculo 2 vezes para a mesma data produz estado final idêntico e nenhum alerta duplicado; validade em D+31 → válido, D+30 e D+29 → a vencer, D−1 → vencido. Detalhe: [H15](historias-backlog.md#h15) |
| RF06 | Exibir semáforo por trabalhador e por contrato | J4 | Verde se todas as exigências estão válidas; amarelo se alguma vence em até 30 dias; vermelho se alguma falta ou venceu; contrato assume a pior cor, com contagem por cor. Detalhe: [H16](historias-backlog.md#h16) |
| RF07 | Alertar documentos próximos do vencimento | J4 | 1 alerta ao entrar na faixa de 30 dias e 1 ao vencer; nenhum alerta repetido para o mesmo documento e faixa. Detalhe: [H05](historias-backlog.md#h05) |
| RF08 | Reaproveitar documento válido em outro contrato | J2/J4 | Trabalhador já aprovado no contrato A, vinculado ao contrato B do mesmo cliente, tem as exigências comuns atendidas sem novo upload e semáforo calculado em até 60 s; nunca entre clientes. Detalhe: [H11](historias-backlog.md#h11) |
| RF09 | Consultar liberação na portaria e registrar acesso | J5 | Resposta `LIBERADO`, `BLOQUEADO` (com orientação genérica, sem dado de documento) ou `NAO_CADASTRADO` em p95 ≤ 800 ms; entrada só registrável a partir de consulta `LIBERADO` com até 10 minutos. Detalhe: [H04](historias-backlog.md#h04), [H18](historias-backlog.md#h18) |
| RF10 | Registrar trilha de auditoria de decisões e acessos | J3/J5 | Toda escrita de negócio e toda consulta de portaria gera exatamente 1 registro imutável com autor, ação, recurso e data/hora; nenhuma rota altera ou apaga registros. Detalhe: [H20](historias-backlog.md#h20), [H21](historias-backlog.md#h21) |

!!! note "RF10"
    Proposto pelo Tech Lead: a página [É / Não É](../estrategia/e-nao-e.md) inclui "uma trilha de auditoria de decisões e acessos", mas não havia RF correspondente. A PO valida.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: RF01 a RF09. Do Tech Lead: RF10 (trilha de auditoria) proposto e critérios mensuráveis ligados às histórias (C5). |
| 1.1 | 01/11/2026 | @Lelafeta | Alinhamento do limite de upload (5 MB PDF/JPG) e resposta da portaria ao Caderno. |
