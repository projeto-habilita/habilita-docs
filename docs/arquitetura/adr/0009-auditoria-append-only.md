# ADR 0009 — Trilha de auditoria append-only

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend |
| Consultados | PO |
| Relacionados | H20, H21, RN-14 |

## 1. Contexto

O produto se apresenta como "trilha de auditoria de decisões e acessos". Em um incidente (trabalhador inapto que entrou, documento aprovado por engano), a contratante precisa saber quem decidiu, o quê e quando. Um registro que pode ser alterado não serve como evidência.

## 2. Opções consideradas

1. **Logs de aplicação (Cloud Logging)** — já existem; mas têm retenção limitada, não são consultáveis por cliente no produto e não devem conter dados de negócio.
2. **Coleção no Firestore sem regra especial** — fácil; mas nada impede alteração por código futuro.
3. **Coleção append-only no Firestore**, escrita na mesma transação ou lote da operação de negócio, sem rota de alteração ou exclusão, e com Rules deny-all.
4. **Cadeia de hashes** (cada registro contém o hash do anterior) — detecta adulteração; é o desafio central do projeto Alçada, não do Habilita. Fica como evolução.

## 3. Decisão

- Coleção `tenants/{t}/auditoria/{id}` **somente inserção**: o repositório de auditoria expõe apenas `registrar()`; não há métodos de atualização ou exclusão nem rotas para isso.
- O registro é gravado **no mesmo lote atômico** da operação de negócio. Se a auditoria falha, a operação falha.
- Campos: `ator` (uid), `papel`, `acao` (ex.: `DOCUMENTO_APROVADO`), `recurso` (tipo e id), `detalhes` (sem dados pessoais além de ids), `dataHora` (UTC), `requestId`.
- Leitura pelo `admin_contratante` via `GET /api/auditoria`, paginada (H21).

## 4. Consequências

- **Positivas:** evidência consistente com o estado (mesmo lote); simples de implementar.
- **Negativas:** não detecta adulteração feita diretamente no banco por quem tem credencial de administrador (risco aceito no MVP; evolução: cadeia de hashes ou exportação periódica para bucket com retenção bloqueada).
- **Obrigatório:** todo caso de uso de escrita chama a auditoria; revisão de PR verifica.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Adulteração direta no banco por quem tem credencial de administrador | Aceito no MVP; evolução: cadeia de hashes ou exportação para bucket com retenção bloqueada |
| Caso de uso novo esquecer de auditar | Teste por caso de uso exigindo exatamente 1 registro; item no checklist de revisão |

## 6. Validação da decisão

- [ ] Teste de cada caso de uso de escrita confirmando exatamente 1 registro de auditoria (H20.CA1).
- [ ] Teste que confirma ausência de rotas `PUT`, `PATCH` e `DELETE` sob `/auditoria` no OpenAPI.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Trilha de auditoria append-only. Status: proposto. |
| 1.1 | 05/11/2026 | @osamuelleite | Leitura da auditoria pelo `admin_contratante`, no lugar de `gestor` (ADR 0002, 4 papéis). |
