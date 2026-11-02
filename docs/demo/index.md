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

Passos executados no ambiente de demonstração com os usuários do seed do tenant `tnt_alfa` (papéis `gestor`, `terceirizada` e `porteiro`), sem dados reais:

1. **Login da contratante — J1 (`gestor` do seed, `tnt_alfa`):** entrar com o usuário gestor do seed, selecionar a terceirizada cadastrada (`ter_alfa_1`) e os tipos de documento (`ASO` e `NR-35`).
2. **Criação do contrato e da matriz — J1 (`gestor`):** criar um contrato vigente vinculando a terceirizada e salvar a matriz de exigências com `ASO` e `NR-35` (H01).
3. **Cadastro e alocação do trabalhador — J2 (`terceirizada` do seed, `ter_alfa_1`):** entrar com o usuário da terceirizada do seed, cadastrar um trabalhador com CPF fictício válido e vinculá-lo ao contrato criado (H02, H10).
4. **Teste de erro no upload (> 5 MB) — J2 (`terceirizada`):** tentar enviar um arquivo com mais de 5 MB para o trabalhador; o sistema recusa o envio (HTTP 413) e exibe na tela a mensagem `"Envie um PDF ou JPG de até 5 MB"` (H02.CA6, H02.CA11).
5. **Envio de documentos válidos — J2 (`terceirizada`):** enviar os arquivos PDF de `ASO` e `NR-35` (até 5 MB, com marca de documento fictício) informando a data de validade `validoAte`; ambos entram na fila com status `PENDENTE` (H02.CA5).
6. **Rejeição com motivo obrigatório — J3 (`gestor`):** entrar como gestor, abrir a fila de pendentes, rejeitar o `NR-35` informando um motivo de 10 a 500 caracteres (H03) e verificar que o trabalhador permanece com semáforo **vermelho** no contrato.
7. **Reenvio e aprovação — J2 e J3 (`terceirizada` → `gestor`):** consultar o motivo da rejeição como terceirizada, enviar novo `NR-35` válido (H14) e, como gestor, aprovar o `ASO` e o novo `NR-35` confirmando `validoAte` (H13); o semáforo do trabalhador no contrato passa para **verde** (ou **amarelo**, se vencer em até 30 dias).
8. **Consulta na portaria e visualização do semáforo (`gestor` e `porteiro` do seed):** conferir no painel do gestor o semáforo atualizado do trabalhador no contrato (H16) e, com o usuário `porteiro` do seed, consultar o CPF fictício na portaria (H04), recebendo o resultado `LIBERADO` (ou `BLOQUEADO` antes da aprovação total) e registrando o acesso (H18).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Roteiro e cena do Demo Day, com tabela de evidências a preencher. |
| 1.1 | 02/11/2026 | @Lelafeta | Inclusão da seção Roteiro da demo com passos numerados das jornadas J1 a J3, portaria e caso de erro de upload acima de 5 MB. |
