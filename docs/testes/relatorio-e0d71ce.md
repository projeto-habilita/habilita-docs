# Relatório de Execução e Laudo de QA — CP4 (`e0d71ce`)

| Campo | Valor |
|---|---|
| **Marco / Gate** | Checkpoint 4 — Qualidade, Segurança, Carga e Homologação E2E |
| **Data da Execução** | 26/11/2026 (`2026-11-26T00:26:21-03:00`) |
| **Repositório Alvo** | `projeto-habilita/Habilita` (`apps/api`) |
| **Branch das Suítes QA** | `test/hab-cp4-suites-qa` (base `origin/main` @ `4a6dd7a` (PR #32 mesclado)) |
| **Commit Curto / Completo** | `e0d71ce` / `e0d71ce32cf2145dce56eff301e35af3e5331767` |
| **Entregas Vinculadas** | `[QA-05]` (#83), `[QA-06]` (#34), `[QA-07]` (#35), `[QA-08]` (#84), `[QA-09]` (#85), `[QA-10]` (#36), `[QA-11]` (#86/#115), `[QA-13]` (#88), `[QA-14]` (#89), `[QA-15]` (#90), `[PO-15]` (#28) |
| **Relatório Consolidado** | [Relatório v1 (CP4)](relatorios/relatorio-v1.md) |
| **Matriz de Rastreabilidade** | [Matriz 100% Must (78 CAs)](matriz.md) |

---

## 1. Sumário Executivo e Parecer de QA

A execução da suíte consolidada do Checkpoint 4 no commit `e0d71ce` atingiu **100% de aprovação nos testes automatizados ativos** (**345 passed**, **85 xfailed estritos (`strict=True`)**, **0 failed**, **0 skipped**, além de 1 teste marcado `@pytest.mark.integration` deselecionado na rodada sem emulador live), com **98,1% de cobertura no domínio** (`1.007/1.026` stmts, `98,15%`, meta `>= 80%`) e **98,0% de cobertura global** (meta `>= 60%`).

No ambiente Docker Compose completo (API + Firebase Auth/Firestore Emulators + Supabase Storage local), a homologação E2E via API nos 4 perfis canônicos (`[QA-05]`) aprovou **38/38 verificações** sem falhas, e o teste de carga `k6` (`20 VUs × 60 s`) processou **2.509 requisições com 0,00% de erro**, aprovando o `p95` de escrita (**1.063,27 ms** `<= 1.500 ms`), mas identificando **violação do `p95` de leitura** (**1.507,45 ms** `> 800 ms`) devido a consultas `.stream()` sem `.limit()` sob container limitado a `0.5` CPU.

Para a triagem `[PO-15]` (#28), há **0 defeitos de severidade Crítica abertos**, **3 defeitos de severidade Alta abertos** (`#112`, `#113` e divergência RBAC `analista_sst`, além de 5 Médios e 1 Baixo), bem como **escopo pendente dependente de terceiros** (telas Web `PR #55`, 8 operações J4/J5/Auditoria/Interno na API e URL pública Cloud Run para smoke `[QA-08]`).

---

## 2. Resultados das 5 Categorias do CP4

### 2.1 Categoria 1 — Unidade, Integração, Limites de Upload e Cobertura (`[QA-11]`, `[QA-13]`, `[QA-14]`, `[QA-15]`)

- **Comando executado:** `uv run --with pytest-cov pytest -m "not integration" --cov=habilita --cov-report=term-missing --cov-report=xml:/tmp/coverage-cp4.xml --junitxml=/tmp/junit-cp4.xml` (com `HABILITA_OPENAPI_PATH` apontando para o contrato `openapi.yaml` da raiz).
- **Totais:** **430 testes executados** (`345 passed`, `85 xfailed` com `strict=True`, `0 failed`, `0 skipped`, `1 deselected`) em `7,223 s`.
- **Verificações estáticas:** `uv run ruff check tests/` (0 erros), `uv run ruff format --check tests/` (21 arquivos conformes), `uv run lint-imports` (`1 kept, 0 broken`), `MYPYPATH=src uv run mypy` nas 8 suítes novas/atualizadas (0 erros).

| Suíte | Total | Passed | Xfailed (`strict=True`) | Escopo |
|---|---:|---:|---:|---|
| `test_armazenamento_memoria.py` | 20 | 20 | 0 | Validação de caminhos, traversal e bucket em memória |
| `test_autorizacao_rotas.py` | 216 | 150 | 66 | Matriz 100% rotas × papéis × multi-tenant (`[QA-09]` / `[QA-08]`) |
| `test_cobertura_dominio_qa11.py` | 10 | 10 | 0 | Ramos descobertos de domínio, repositórios e storage (`[QA-11]`) |
| `test_contrato_openapi.py` | 37 | 27 | 10 | Conformidade OpenAPI e RFC 9457 (`[QA-09]`) |
| `test_cors.py` | 6 | 6 | 0 | Política CORS e preflight |
| `test_falhas_e_timeouts.py` | 32 | 30 | 2 | Falhas Firestore/Supabase, timeouts e regressões `#112`/`#114` |
| `test_firestore_repositorios.py` | 5 | 5 | 0 | Repositórios Firestore e HMAC CPF |
| `test_h19_isolamento_tenant.py` | 7 | 5 | 2 | Isolamento multi-tenant e Rules deny-all (`H19`) |
| `test_j1_contratos_cadastros.py` | 11 | 11 | 0 | Jornada J1 (terceirizadas, tipos, contratos, matriz) |
| `test_j1_j3_criterios_complementares.py` | 21 | 20 | 1 | Critérios complementares J1–J3 e reuso `H11` (só `H01.CA6` `/auditoria` em `xfail`) |
| `test_j2_j3_trabalhadores_documentos.py` | 17 | 17 | 0 | Jornadas J2 e J3 (trabalhadores, upload, fila, revisão) |
| `test_j4_j5_especificacao_qa14_qa15.py` | 4 | 0 | 4 | Especificação J4/J5 (`[QA-14]` idempotência job e `[QA-15]` sigilo portaria) |
| `test_pr32_revisao_criterios.py` | 17 | 17 | 0 | Critérios de revisão J1–J3 e `H07.CA4` |
| `test_saude_sessao_e_erros.py` | 6 | 6 | 0 | `/healthz`, `/readyz`, `/me`, paginação e `Problem Details` |
| `test_upload_limites_qa13.py` | 21 | 21 | 0 | BVA upload: 4,9 MB / 4.999.999 / 5.000.000 (`201`), `5.000.001` / 5,1 MB (`413`), 0 B (`400`), magic bytes (`415`) |
| **Total** | **430** | **345** | **85** | **0 falhas, 0 skips** |

#### Cobertura de Código (`coverage-cp4.xml`)

| Módulo | Statements | Miss | Cobertura |
|---|---:|---:|---:|
| `src/habilita/__init__.py` | 0 | 0 | 100% |
| `src/habilita/__main__.py` | 7 | 7 | 0% |
| `src/habilita/armazenamento/__init__.py` | 3 | 0 | 100% |
| `src/habilita/armazenamento/porta.py` | 50 | 0 | 100% |
| `src/habilita/armazenamento/supabase.py` | 93 | 0 | 100% |
| `src/habilita/cadastros/repositorio.py` | 248 | 1 | 99% |
| `src/habilita/cadastros/rotas.py` | 51 | 0 | 100% |
| `src/habilita/cadastros/schemas.py` | 76 | 0 | 100% |
| `src/habilita/comum/auth.py` | 38 | 1 | 97% |
| `src/habilita/comum/erros.py` | 64 | 0 | 100% |
| `src/habilita/comum/firestore.py` | 49 | 0 | 100% |
| `src/habilita/config.py` | 54 | 0 | 100% |
| `src/habilita/main.py` | 32 | 0 | 100% |
| `src/habilita/saude/__init__.py` | 0 | 0 | 100% |
| `src/habilita/saude/rotas.py` | 28 | 0 | 100% |
| `src/habilita/sessao/rotas.py` | 15 | 0 | 100% |
| `src/habilita/trabalhadores/repositorio.py` | 397 | 16 | 96% |
| `src/habilita/trabalhadores/rotas.py` | 112 | 1 | 99% |
| `src/habilita/trabalhadores/schemas.py` | 92 | 1 | 99% |
| **Domínio (`cadastros/*` + `trabalhadores/*` + `armazenamento/porta.py`)** | **1.026** | **19** | **98,1%** *(98,15%, meta >= 80%)* |
| **Global (`TOTAL`)** | **1.409** | **27** | **98,0%** *(98,08%, meta >= 60%)* |

---

### 2.2 Categoria 2 — Contrato OpenAPI (`test_contrato_openapi.py`)

- Validação das **29 operações (25 caminhos)** do `openapi.yaml` contra `/openapi.json` e chamadas HTTP reais com verificação de **RFC 9457** (`application/problem+json`: `type`, `title`, `status`, `detail`, `instance`): **27 passed, 10 xfailed (`strict=True`)**.
- **Divergências detectadas:**
  - `8` operações de J4/J5/Auditoria/Interno ainda não registradas em `b8fef43` (`GET /contratos/{contratoId}/conformidade`, `GET /painel/contratos`, `GET /alertas`, `POST /alertas/{alertaId}/leitura`, `POST /liberacoes/consultas`, `POST /acessos`, `GET /auditoria`, `POST /internal/jobs/recalculo-vencimentos`).
  - Schema `Pendencia` retorna `tipoNome` em vez de `tipoDocumentoNome` (`BUG-011`).
  - `GET /documentos` aceita query param `statusRevisao` em vez de `status` (`BUG-012`).

---

### 2.3 Categoria 3 — Matriz 100% Rotas × Autorização / Multi-tenant (`test_autorizacao_rotas.py` + `test_h19_isolamento_tenant.py`)

- **216 casos parametrizados** cobrindo todas as **29 operações** nos 4 eixos obrigatórios (`sem token -> 401` exceto `/healthz` e `/readyz`, `token inválido/expirado -> 401`, `papel não autorizado -> 403`, `cross-tenant / cross-terceirizada -> 404` conforme ADR 0002): **150 passed, 66 xfailed (`strict=True`)**.
- **Prontidão `[QA-08]` (#84):** suporte nativo a `HABILITA_BASE_URL` via `httpx.Client` para validação pós-deploy no Cloud Run assim que a URL pública estiver disponível.
- **Divergência detectada:** `POST /trabalhadores` e `POST /trabalhadores/{trabalhadorId}/documentos` aceitam `analista_sst` na API, contrariando `x-papeis: [admin_contratante, terceirizada]` no `openapi.yaml` (`BUG-010`).

---

### 2.4 Categoria 4 — Falhas, Timeouts e Resiliência (`test_falhas_e_timeouts.py`)

- **32 casos executados** (`30 passed, 2 xfailed strict=True`):
  - Indisponibilidade e timeouts do Firestore no `/readyz` (`503` sem vazar stack trace) e propagação segura nos repositórios;
  - Bloqueio de fallback em memória fora de `app_env == "local"` (`ci` e `prod`);
  - Falhas em `SupabaseArmazenamento`: `httpx.TimeoutException`, `httpx.ConnectError`, HTTP `500/502/403`, JSON inválido/truncado em signed URL e redirecionamento inesperado;
  - Payloads malformados, CPFs inválidos, datas inválidas e `Content-Type` incorreto tratados em `application/problem+json` sem stack trace.
  - **2 regressões `xfail(strict=True)` confirmadas abertas:** `#112` (`BUG-006`) e `#114` (`BUG-008`).

---

### 2.5 Categoria 5 — Carga Local `k6` (ADR 0010 / ADR 0012)

- Execução contra Docker Compose local (`http://127.0.0.1:8000`) com **20 VUs concorrentes por 60 s**, autenticação real via Firebase Auth Emulator e guardrails ADR 0012 ativos (`ALVO=local` obrigatório; `ALVO=producao` limitado a `<= 5 VUs`, `<= 5 req/s`, `<= 500 reqs`):

| Métrica | Resultado Medido | Meta (RNF04 / Estratégia) | Status |
|---|---:|---:|:---:|
| Requisições totais / Iterações | `2.509` / `471` | — | ℹ️ |
| Vazão média (`rps`) | `41,01 req/s` | — | ℹ️ |
| Taxa de falha HTTP (`http_req_failed`) | `0,00%` (`0/2.509`) | `< 1,00%` | ✅ |
| Checks aprovados | `2.503/2.503` (`100%`) | `100%` | ✅ |
| `p95` escrita (`tipo:escrita`) | **`1.063,27 ms`** (`avg: 520,15 ms`, `max: 1.551,21 ms`) | `<= 1.500 ms` | ✅ |
| `p95` leitura (`tipo:leitura`) | **`1.507,45 ms`** (`avg: 426,02 ms`, `max: 2.585,49 ms`) | `<= 800 ms` | ⚠️ **Violado (`BUG-013`)** |
| `p95` global | `1.462,30 ms` (`avg: 448,25 ms`) | — | ℹ️ |

---

## 3. Homologação E2E via API nos 4 Perfis (`[QA-05]` #83)

Executada em Docker Compose completo via `apps/api/tests/carga/executar_homologacao_e2e.py` com tokens reais do Firebase Auth Emulator (`127.0.0.1:9099`):

- **Resultado geral:** **38/38 verificações aprovadas (`0` falhas)** ([homologacao-e2e-cp4.json](relatorios/homologacao-e2e-cp4.json)).
- **Cobertura validada ponta a ponta:**
  - `/healthz` (`200`), `/readyz` (`200`) e `GET /me` nos 4 perfis (`admin_contratante`, `analista_sst`, `terceirizada`, `porteiro`).
  - **J1 (`admin_contratante`):** criação de terceirizada, tipo de documento, contrato e definição de exigências (`versaoMatriz=1`).
  - **J2 (`terceirizada`):** cadastro de trabalhador com CPF mascarado (`***.663.337-**`), alocação (`VERMELHO`, 2 pendências), uploads PDF fictícios no Supabase Storage local + download via signed URL (`200`, `sha256_match=True`).
  - **J3 (`analista_sst`):** listagem da fila (`total_pendentes=6`), bloqueio de motivo `< 10` chars (`400`), rejeição válida (`200`), conflito `409` ao aprovar documento rejeitado, aprovação de ASO/NR10 e recálculo de semáforo (`VERMELHO -> VERDE`).
  - **RBAC e Isolamento:** `porteiro` bloqueado (`403`) em rotas de gestão; `terceirizada` bloqueada (`403`) em aprovação; cross-tenant (`tnt_beta -> tnt_alfa`) e cross-terceirizada (`ter_alfa_2 -> ter_alfa_1`) retornando `404` em todas as rotas; sonda J5 `POST /liberacoes/consultas` retornando `404` (`j5PortariaImplementada: false` em `b8fef43`).

---

## 4. Catálogo Padronizado de Defeitos (`BUG-001`..`BUG-014`) e Triagem (`[PO-15]` #28)

Severidades classificadas conforme [Estratégia de Testes §11](estrategia.md#11-severidade-de-defeitos): **Crítica** (bloqueia merge/deploy), **Alta** (corrigir na sprint corrente), **Média** (priorizar no backlog), **Baixa** (backlog).

### 4.1 Resumo Executivo para Triagem (`[PO-15]`)

| Severidade | Resolvidos | Abertos | Bloqueia Demo Day / Deploy? |
|---|---:|---:|---|
| **Crítica** | 0 | **0** | Não há defeito crítico aberto (isolamento multi-tenant, upload privado e CPF protegidos) |
| **Alta** | 3 | **3** | **Atenção prioritária na sprint:** `#112` (`BUG-006`), `#113` (`BUG-007`) e RBAC `analista_sst` (`BUG-010` / `#119`) |
| **Média** | 2 | **5** | Priorizar: `#114` (`BUG-008`), `#116` (`BUG-009`), contrato `Pendencia`/`status` (`BUG-011` / `#120`, `BUG-012` / `#121`) e `p95` leitura `k6` (`BUG-013` / `#122`) |
| **Baixa** | 0 | **1** | Backlog: ordenação fila por `id` / fallback local 15 min (`BUG-014` / `#123`) |
| **Total** | **5** | **9** | **14 registros catalogados (`BUG-001`..`BUG-014`)** |

### 4.2 Tabela Consolidada (`BUG-001`..`BUG-014`)

| ID | Severidade | Componente / Jornada | Resumo | Status | Issue / Commit / Regressão |
|---|---|---|---|---|---|
| `BUG-001` | **Alta** | `cadastros/repositorio.py` · J1 (`H01.CA3`) | `criar_contrato` fixava `vigente=True` ignorando `dataFim` no passado | ✅ **Resolvido** | Corrigido em `b8fef43` · `test_h01_ca3_contrato_com_data_fim_no_passado_nao_fica_vigente` |
| `BUG-002` | **Média** | `cadastros/repositorio.py` · J1 (`H08.CA3`) | Cadastro de terceirizada não verificava CNPJ duplicado no tenant (`201` vs `409`) | ✅ **Resolvido** | Corrigido em `b8fef43` · `test_h08_ca3_cnpj_duplicado_no_mesmo_tenant_retorna_409` |
| `BUG-003` | **Média** | `trabalhadores/repositorio.py` · J2 (`H10.CA3`) | Alocação duplicada no mesmo contrato retornava `201` em vez de `409` | ✅ **Resolvido** | Corrigido em `b8fef43` · `test_h10_ca3_vincular_trabalhador_duplicado_no_mesmo_contrato_retorna_409` |
| `BUG-004` | **Alta** | `trabalhadores/rotas.py` · J3 (`H13.CA3`) | Aprovação aceitava `validoAte < dataEmissao` sem rejeitar com `400` | ✅ **Resolvido** | Corrigido em `b8fef43` · `test_h13_ca3_aprovar_documento_com_valido_ate_anterior_data_emissao_rejeitado` |
| `BUG-005` | **Alta** | `trabalhadores/schemas.py` · J2/J3 (`H02.CA8`, `H02.CA10`, `H03.CA2`) | Upload aceitava `dataEmissao` futura e `validoAte < dataEmissao`; rejeição aceitava só espaços | ✅ **Resolvido** | Corrigido em `b7da8bf` · `test_j1_j3_criterios_complementares.py` e `test_pr32_revisao_criterios.py` |
| `BUG-006` | **Alta** | `trabalhadores/repositorio.py` · J2 (`H02`) | `criar_documento` engole `ArmazenamentoError` quando `app_env == "local"` e grava metadado órfão | 🔴 **Aberto** | Issue `#112` · `test_regressao_112_criar_documento_nao_engole_armazenamento_error_nem_grava_orfao` (`xfail`) |
| `BUG-007` | **Alta** | `trabalhadores/repositorio.py` · J3 (`H13`, `H03`) | Aprovação/rejeição e recálculo de semáforo sem transação atômica (`WriteBatch`) | 🔴 **Aberto** | Issue `#113` · risco de inconsistência parcial documento × alocação |
| `BUG-008` | **Média** | `comum/auth.py` · Transversal (`H07.CA3`) | `extrair_usuario` expõe mensagem crua de exceção Firebase no campo `detail` do HTTP `401` | 🔴 **Aberto** | Issue `#114` · `test_regressao_114_auth_401_nao_expoe_mensagem_crua_de_excecao_firebase` (`xfail`) |
| `BUG-009` | **Média** | `.github/workflows/job-diario.yml` · J4 (`H15`) | Timeout/keep-alive ausente no workflow diário (risco com cold start / pausa Supabase) | 🔴 **Aberto** | Issue `#116` · mitigado por `workflow_dispatch` |
| `BUG-010` | **Alta** | `trabalhadores/rotas.py` · RBAC (`H19.CA2`) | `POST /trabalhadores` e `POST /trabalhadores/{id}/documentos` aceitam `analista_sst`, divergindo do `openapi.yaml` | 🔴 **Aberto** | Issue `#119` · `test_eixo3_perfil_nao_autorizado_retorna_403[...negado_analista_sst]` (`xfail`) |
| `BUG-011` | **Média** | `trabalhadores/schemas.py` · Contrato J2/J3 | Schema `Pendencia` retorna `tipoNome` em vez de `tipoDocumentoNome` (`openapi.yaml`) | 🔴 **Aberto** | Issue `#120` · `test_contrato_schema_pendencia_usa_tipo_documento_nome` (`xfail`) |
| `BUG-012` | **Média** | `trabalhadores/rotas.py` · Contrato J3 | `GET /documentos` usa query param `statusRevisao` em vez de `status` (`openapi.yaml`) | 🔴 **Aberto** | Issue `#121` · `test_contrato_get_documentos_filtra_por_query_param_status` (`xfail`) |
| `BUG-013` | **Média** | `trabalhadores/repositorio.py` · Desempenho (`RNF04`) | `p95` leitura em `k6` atingiu `1.507,45 ms` (`> 800 ms`) por `.stream()` sem `.limit()` sob `cpus: 0.5` | 🔴 **Aberto** | Issue `#122` · evidência `k6-resumo-cp4.json` (0% erros HTTP; escrita OK `1.063,27 ms`) |
| `BUG-014` | **Baixa** | `trabalhadores/repositorio.py` · J3 (`H12.CA1`, `H12.CA3`) | Fila ordena por `id` em vez de `enviadoEm`; fallback local assina por 15 min (Supabase usa `<= 300 s`) | 🔴 **Aberto** | Issue `#123` · evidência `test_h03_ca1_analista_revisa_fila_e_gera_url_download` |

### 4.3 Detalhamento Técnico dos Defeitos (`BUG-001`..`BUG-014`)

- **`BUG-001` [Alta · Resolvido (`b8fef43`)]:** Em `cadastros/repositorio.py` (J1), `criar_contrato` persistia `vigente=True` mesmo com `dataFim < hoje`. Passos: `POST /contratos` com `dataInicio` e `dataFim` no passado. Esperado: `vigente == false` (RN-10). Obtido antes da correção: `vigente == true`. Regressão: `test_h01_ca3_contrato_com_data_fim_no_passado_nao_fica_vigente` (aprovado).
- **`BUG-002` [Média · Resolvido (`b8fef43`)]:** Em `cadastros/repositorio.py` (J1), `POST /terceirizadas` com mesmo CNPJ no mesmo tenant criava registro duplicado (`201`) em vez de `409`. Regressão: `test_h08_ca3_cnpj_duplicado_no_mesmo_tenant_retorna_409` (aprovado).
- **`BUG-003` [Média · Resolvido (`b8fef43`)]:** Em `trabalhadores/repositorio.py` (J2), `POST /contratos/{id}/alocacoes` para par já alocado retornava `201` em vez de `409`. Regressão: `test_h10_ca3_vincular_trabalhador_duplicado_no_mesmo_contrato_retorna_409` (aprovado).
- **`BUG-004` [Alta · Resolvido (`b8fef43`)]:** Em `trabalhadores/rotas.py` (J3), `POST /documentos/{id}/aprovacao` aceitava `validoAte < dataEmissao`. Esperado: HTTP `400` RFC 9457 (RN-09). Regressão: `test_h13_ca3_aprovar_documento_com_valido_ate_anterior_data_emissao_rejeitado` (aprovado).
- **`BUG-005` [Alta · Resolvido (`b7da8bf`)]:** Em `trabalhadores/schemas.py` (J2/J3), validações de `dataEmissao` futura (`H02.CA8`), `validoAte < dataEmissao` (`H02.CA10`) e motivo apenas com espaços (`H03.CA2`) falhavam no PR #102; corrigidas em `b7da8bf` e aprovadas em `test_j1_j3_criterios_complementares.py` e `test_pr32_revisao_criterios.py`.
- **`BUG-006` [Alta · Aberto (`#112`)]:** Em `src/habilita/trabalhadores/repositorio.py:691-702` (J2), ao enviar documento com `app_env == "local"` e erro no storage (`ArmazenamentoError`), a exceção é engolida e o metadado é gravado no Firestore/memória sem arquivo correspondente. Esperado: propagar falha (5xx) e não persistir metadado órfão. Regressão: `test_regressao_112_criar_documento_nao_engole_armazenamento_error_nem_grava_orfao` (`xfail strict=True`).
- **`BUG-007` [Alta · Aberto (`#113`)]:** Em `trabalhadores/repositorio.py` (J3), aprovação/rejeição atualiza o documento e recalcula alocações em chamadas separadas sem `WriteBatch`/transação atômica. Falha intermediária deixa o semáforo divergente do status do documento.
- **`BUG-008` [Média · Aberto (`#114`)]:** Em `src/habilita/comum/auth.py:59-60` (Transversal), `extrair_usuario` concatena `str(exc)` da exceção do Firebase Admin SDK no campo `detail` do HTTP `401`. Esperado: mensagem genérica segura sem detalhes internos de infraestrutura. Regressão: `test_regressao_114_auth_401_nao_expoe_mensagem_crua_de_excecao_firebase` (`xfail strict=True`).
- **`BUG-009` [Média · Aberto (`#116`)]:** Em `.github/workflows/job-diario.yml` (J4), falta configuração explícita de timeout/retry/keep-alive para evitar falha por cold start ou suspensão gratuita do Supabase.
- **`BUG-010` [Alta · Aberto (`#119`)]:** Em `src/habilita/trabalhadores/rotas.py:95,211`, as rotas `POST /trabalhadores` e `POST /trabalhadores/{trabalhadorId}/documentos` permitem `analista_sst`, enquanto `openapi.yaml` (`x-papeis`) restringe ambas a `["admin_contratante", "terceirizada"]`. Regressão: `test_autorizacao_rotas.py` (`xfail strict=True`).
- **`BUG-011` [Média · Aberto (`#120`)]:** Em `src/habilita/trabalhadores/schemas.py:68`, `Pendencia` expõe `tipoNome` em vez de `tipoDocumentoNome` definido no `openapi.yaml`. Regressão: `test_contrato_schema_pendencia_usa_tipo_documento_nome` (`xfail strict=True`).
- **`BUG-012` [Média · Aberto (`#121`)]:** Em `src/habilita/trabalhadores/rotas.py:289`, `GET /documentos` declara query param `statusRevisao` em vez de `status` definido no `openapi.yaml`. Regressão: `test_contrato_get_documentos_filtra_por_query_param_status` (`xfail strict=True`).
- **`BUG-013` [Média · Aberto (`#122`)]:** Sob 20 VUs × 60 s no Compose (`cpus: 0.5`), `GET /documentos` e `GET /trabalhadores/{id}` usam `.stream()` sem `.limit()`/`select` no Firestore, elevando `p95` de leitura para `1.507,45 ms` (`> 800 ms`, `k6` exit code 99).
- **`BUG-014` [Baixa · Aberto (`#123`)]:** Fila de pendentes ordena em memória por `id` e não por `enviadoEm` (`H12.CA1`), e o adapter local em memória gera URL simulada de 15 min enquanto `SupabaseArmazenamento` respeita `<= 300 s` (`H12.CA3`).

---

## 5. Escopo Pendente Dependente de Terceiros

1. **Interface Web (`Habilita PR #55` — Frontend):** telas de J1–J3 e portaria necessárias para validar `H02.CA11` (mensagem `"Envie um PDF ou JPG de até 5 MB"`), `H09.CA3` (sugestão de validade e aviso ao aprovar acima do padrão), `H12.CA2` (estado vazio `"Nenhum documento aguardando análise"`), `H07.CA1` (tela inicial do `porteiro`), E2E Playwright e Lighthouse `>= 90`.
2. **Rotas de Backend J4, J5, Auditoria e Job Interno (8 de 29 operações OpenAPI):** `/contratos/{contratoId}/conformidade` (`H16.CA4`, `H23`), `/painel/contratos` (`H23`), `/alertas` e `/alertas/{alertaId}/leitura` (`H05`), `/liberacoes/consultas` (`H04`), `/acessos` (`H18`), `/auditoria` (`H01.CA6`, `H20`) e `/internal/jobs/recalculo-vencimentos` (`H15`). Todas já possuem suítes de contrato, autorização e especificação escritas com `@pytest.mark.xfail(strict=True)`.
3. **Deploy Público da API no Cloud Run (`[QA-08]` #84):** aguardando URL pública para executar `HABILITA_BASE_URL=<url> pytest tests/test_autorizacao_rotas.py` e smoke de carga sob os tetos da [ADR 0012](../arquitetura/adr/0012-teto-teste-carga-producao.md).

---

## 6. Artefatos Exportados (`[QA-10]` #36)

- **Relatório JUnit XML:** [junit-cp4.xml](relatorios/junit-cp4.xml)
- **Relatório de Cobertura Cobertura/XML:** [coverage-cp4.xml](relatorios/coverage-cp4.xml)
- **Resumo de Carga `k6` (JSON):** [k6-resumo-cp4.json](relatorios/k6-resumo-cp4.json)
- **Evidência Homologação E2E 4 Perfis (JSON):** [homologacao-e2e-cp4.json](relatorios/homologacao-e2e-cp4.json)
- **Relatório Consolidado CP4 (v1):** [relatorio-v1.md](relatorios/relatorio-v1.md)

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/11/2026 | @CauaRVeloso | Laudo completo do Checkpoint 4 sobre o commit `e0d71ce` (`test/hab-cp4-suites-qa`), cobrindo as 5 categorias, homologação E2E `[QA-05]`, catálogo `BUG-001`..`BUG-014` (`[PO-15]`) e links dos artefatos (`[QA-10]`). |
