# ADR 0008 — Consulta de liberação por CPF com HMAC e POST

| Campo | Valor |
|---|---|
| Status | **Proposto** (aprovar até o Checkpoint 2 — 01/11/2026) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend |
| Consultados | PO |
| Relacionados | H02, H04, H18, RN-10, AM-03, AM-06 |

## 1. Contexto

A portaria identifica o trabalhador pelo CPF. O CPF é dado pessoal, e o produto precisa (a) encontrar o trabalhador rapidamente (p95 ≤ 800 ms), (b) garantir unicidade por tenant, (c) minimizar a exposição do CPF (LGPD) e (d) registrar cada consulta.

O rascunho de escopo previa `GET /api/liberacao?cpf=...`. Parâmetros de URL costumam ser gravados em logs de proxy, histórico do navegador e ferramentas de monitoramento, o que espalharia CPFs por lugares fora do nosso controle.

## 2. Opções consideradas

**Transporte do CPF**
1. `GET /liberacao?cpf=...` — simples; CPF na URL e, portanto, em logs e histórico.
2. **`POST /liberacoes/consultas` com CPF no corpo** — o corpo não é registrado por padrão; a operação também cria um registro (a consulta), o que combina com POST.

**Armazenamento do CPF**
1. CPF em texto — busca simples; exposição total em caso de vazamento.
2. CPF criptografado (reversível) — protege em repouso; mas buscar exige descriptografar ou um índice auxiliar.
3. Hash simples (SHA-256) — o espaço de CPFs é pequeno (cerca de 10⁹ combinações) e pode ser revertido por força bruta.
4. **HMAC-SHA256 com chave secreta** — busca exata por igualdade; sem a chave, o hash não pode ser revertido por força bruta. Guardar também uma versão **mascarada** para exibição.

## 3. Decisão

- Rota **`POST /api/liberacoes/consultas`** com `{ "cpf": "..." }` no corpo. O CPF nunca aparece em URL.
- Persistir **apenas** `cpfHash = HMAC-SHA256(chave, cpfNormalizado)` e `cpfMascarado` (`***.456.789-**`). O CPF completo **não é guardado** no MVP.
- Índice `tenants/{t}/indiceCpf/{cpfHash}` → `{ trabalhadorId }` garante unicidade (H02.CA3) e busca por chave em uma leitura.
- Chave HMAC em **variável protegida do Render** (e secret do GitHub, se o CI precisar), conforme a ADR 0010, e em `.env` local (valor de desenvolvimento, nunca o de produção). Nunca fixa no código.
- Toda consulta grava `consultasLiberacao` e auditoria. `POST /api/acessos` registra entrada ou barramento a partir do `consultaId` (RN-11).
- CPF de outro tenant resulta em `NAO_CADASTRADO`, sem dado algum do outro tenant (H04.CA3).
- **A portaria não vê nenhum dado de documento** (ficha do Habilita; feedback do CP1, item 3). A resposta traz só `resultado` (`LIBERADO`, `BLOQUEADO` ou `NAO_CADASTRADO`), nome e CPF mascarado do trabalhador, uma **orientação genérica** e o prazo para registrar o acesso. Exemplo para `BLOQUEADO`: "Acesso não liberado. Oriente o trabalhador a procurar o SST da contratante". **Não** devolve tipo de documento, motivo, vencimento, semáforo ou contrato: dizer "NR-35 vencido" já revelaria parte do conteúdo. O detalhe das pendências fica nas telas do `admin_contratante` e do `analista_sst`.
- Quem consulta: `porteiro`, `admin_contratante` e `analista_sst` (matriz da ADR 0002). Para os dois últimos, a resposta da rota é a mesma; o detalhe eles veem na tela do trabalhador.
- **Rate limit** na rota de consulta por usuário (proposta: 30 consultas por minuto) para dificultar varredura de CPFs.

## 4. Consequências

- **Positivas:** CPF fora de URLs e logs; vazamento do banco não revela CPFs sem a chave; busca O(1).
- **Negativas:** sem o CPF completo, não é possível exibi-lo nem exportá-lo (aceito; ver H22, que exporta mascarado); trocar a chave HMAC exige recalcular todos os hashes a partir de nova coleta de CPF (procedimento a documentar se necessário).
- **Obrigatório:** normalizar CPF (somente dígitos) antes do HMAC; validar dígitos verificadores antes de consultar; nunca registrar o corpo dessa rota em log.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| Vazamento da chave HMAC | Chave só em variável protegida do Render; gitleaks no CI; rotação documentada |
| Varredura de CPFs pela portaria | Rate limit (30/min por usuário) e auditoria de cada consulta |
| CPF registrado por engano em log | Teste automatizado que falha ao encontrar 11 dígitos seguidos nos logs |

## 6. Validação da decisão

- [ ] Teste que inspeciona os logs de uma consulta e falha se encontrar 11 dígitos seguidos.
- [ ] Teste de isolamento T-ISO-04 (CPF existente só no tenant B consultado pelo porteiro do tenant A).
- [ ] Teste de rate limit (31ª consulta no mesmo minuto recebe 429).
- [ ] Teste de contrato: a resposta de `BLOQUEADO` não contém `pendencias`, `contratos`, `semaforo`, tipo de documento nem datas de validade.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial: ADR criado pelo Tech Lead na conciliação. Consulta de liberação por CPF com HMAC e POST (C15). Status: proposto. |
| 1.1 | 26/10/2026 | @osamuelleite | Feedback do CP1 (item 3): a portaria recebe só o resultado e uma orientação genérica, sem pendências nem dados de documento; papéis atualizados para os 4 da ficha. |
| 1.2 | 03/11/2026 | @osamuelleite | Plano Custo Zero (ADR 0010): chave HMAC em variável protegida do Render no lugar do Secret Manager. |
