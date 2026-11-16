# Demo Day

Roteiro: problema → hipótese de valor → jornada principal → critérios comprovados → segurança → qualidade → custo → defensibilidade.

Cena: trabalhador, documento, semáforo e portaria — incluindo reutilizar documento válido em outro contrato.

| Evidência | Link |
|---|---|
| Produção | |
| CI | |
| Testes e cobertura | |
| Lighthouse | |
| Custos | |
| Rollback | |

## Roteiro da demo

Passos executados no ambiente de demonstração com os usuários do seed do tenant `tnt_alfa` (papéis `admin_contratante`, `analista_sst`, `terceirizada` e `porteiro`, conforme a [ADR 0002](../arquitetura/adr/0002-tenant-autorizacao.md)), sem dados reais:

1. **Login da contratante — J1 (`admin_contratante` do seed, `tnt_alfa`):** entrar com o usuário administrador da contratante do seed, selecionar a terceirizada cadastrada (`ter_alfa_1`) e os tipos de documento (`ASO` e `NR-35`).
2. **Criação do contrato e da matriz — J1 (`admin_contratante`):** criar um contrato vigente vinculando a terceirizada e salvar a matriz de exigências com `ASO` e `NR-35` (H01).
3. **Cadastro e alocação do trabalhador — J2 (`terceirizada` do seed, `ter_alfa_1`):** entrar com o usuário da terceirizada do seed, cadastrar um trabalhador com CPF fictício válido e vinculá-lo ao contrato criado (H02, H10).
4. **Teste de erro no upload (> 5 MB) — J2 (`terceirizada`):** tentar enviar um arquivo com mais de 5 MB para o trabalhador; o sistema recusa o envio (HTTP 413) e exibe na tela a mensagem `"Envie um PDF ou JPG de até 5 MB"` (H02.CA6, H02.CA11).
5. **Envio de documentos válidos — J2 (`terceirizada`):** enviar os arquivos PDF de `ASO` e `NR-35` (até 5 MB, com marca de documento fictício) informando a data de validade `validoAte`; ambos entram na fila com status `PENDENTE` (H02.CA5).
6. **Rejeição com motivo obrigatório — J3 (`analista_sst`):** entrar como analista de SST, abrir a fila de pendentes, rejeitar o `NR-35` informando um motivo de 10 a 500 caracteres (H03) e verificar que o trabalhador permanece com semáforo **vermelho** no contrato.
7. **Reenvio e aprovação — J2 e J3 (`terceirizada` → `analista_sst`):** consultar o motivo da rejeição como terceirizada, enviar novo `NR-35` válido (H14) e, como analista de SST, aprovar o `ASO` e o novo `NR-35` confirmando `validoAte` (H13); o semáforo do trabalhador no contrato passa para **verde** (ou **amarelo**, se vencer em até 30 dias).
8. **Consulta na portaria e visualização do semáforo (`analista_sst` e `porteiro` do seed):** conferir no painel do analista de SST o semáforo atualizado do trabalhador no contrato (H16) e, com o usuário `porteiro` do seed, consultar o CPF fictício na portaria (H04), recebendo o resultado `LIBERADO` (ou `BLOQUEADO` antes da aprovação total) e registrando o acesso (H18).

## Roteiro do gate do CP3 (15/11)

No CP3 a demonstração roda localmente, pela API, como combinado com o tutor (exceção D5 da [ADR 0011](../arquitetura/adr/0011-divergencias-caderno.md)). As telas de J1 a J3 entram na S7. A API ainda não tem hospedagem: o Render exigiu cartão, o que o Plano Custo Zero proíbe, e a equipe vai escolher um host gratuito sem cartão até o CP4.

0. **Preparação:** rodar `docker compose up`, carregar o seed e conferir `/healthz` e `/readyz`. Quando a API tiver hospedagem, o aquecimento antes da demo segue o que o host escolhido exigir (a registrar na ADR 0011, I8).
1. Abrir o Swagger em `/docs` e autenticar com um usuário de cada papel do seed: `admin_contratante`, `analista_sst`, `terceirizada` e `porteiro`.
2. Executar os passos 1 a 8 do roteiro acima pelas rotas da API, incluindo o caso de erro do passo 4 (upload acima de 5 MB recusado com 413).
3. Mostrar o isolamento: o `admin_contratante` de outro tenant tenta abrir o contrato pelo ID e recebe 404.
4. Mostrar o preview do frontend: abrir um PR com uma mudança de texto e acessar o link de preview gerado. Depende do workflow de deploy do frontend (PR #51 do repositório de código, ainda aberto).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Roteiro e cena do Demo Day, com tabela de evidências a preencher. |
| 1.1 | 02/11/2026 | @Lelafeta | Inclusão da seção Roteiro da demo com passos numerados das jornadas J1 a J3, portaria e caso de erro de upload acima de 5 MB. |
| 1.2 | 15/11/2026 | @Lelafeta | Inclusão do roteiro do gate do CP3 pela API, com passo 0 de preparação e aquecimento do Render. |
| 1.3 | 16/11/2026 | @Lelafeta | Feedback do Checkpoint 3: papéis alinhados à ADR 0002 (`admin_contratante` e `analista_sst`), Render retirado do passo 0 e passo de preview do frontend. |
