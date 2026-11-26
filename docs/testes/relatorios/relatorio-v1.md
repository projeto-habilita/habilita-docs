# Relatório Consolidado de QA — v1 (Checkpoint 4)

| Campo | Valor |
|---|---|
| **Versão do Relatório** | `v1` — Checkpoint 4 (26/11/2026) |
| **Laudo por SHA** | [relatorio-a012ee3.md](../relatorio-a012ee3.md) |
| **Repositório / Branch** | `projeto-habilita/Habilita` · `test/hab-cp4-suites-qa` (`a012ee39ad99597315d14dab411220b7e56c564e`) |
| **Base de Código Auditada** | `origin/feat/hab-cp2-nucleo-backend` (`b8fef43`) + suítes QA CP3/CP4 em `apps/api/tests/` |
| **Matriz de Rastreabilidade** | [Matriz de Rastreabilidade — 100% Must (78 CAs)](../matriz.md) |
| **Artefatos Exportados (`[QA-10]`)** | [JUnit XML](junit-cp4.xml) · [Coverage XML](coverage-cp4.xml) · [k6 JSON](k6-resumo-cp4.json) · [Homologação E2E JSON](homologacao-e2e-cp4.json) |

---

## 1. Resumo dos Resultados por Categoria (CP4)

| Categoria | Suíte / Ferramenta | Resultado Real | Meta CP4 | Status |
|---|---|---|---|:---:|
| **1. Unidade, Integração & Limites** | `pytest` (`-m "not integration"`) + `pytest-cov` (`[QA-11]`, `[QA-13]`, `[QA-14]`, `[QA-15]`) | **345 passed**, **85 xfailed (`strict=True`)**, **0 failed**, **0 skipped** (`1 deselected`) · Domínio: **98,1%** (`1.026` stmts, `19` miss) · Global: **98,0%** (`1.409` stmts, `27` miss) | Domínio `>= 80%`, Global `>= 60%` | ✅ |
| **2. Contrato OpenAPI & RFC 9457** | `test_contrato_openapi.py` (`29` operações / `25` caminhos) | **27 passed**, **10 xfailed (`strict=True`)** (`8` ops J4/J5/Auditoria/Interno ausentes + `Pendencia.tipoNome` + `statusRevisao`) | Contrato sincronizado e RFC 9457 | ⚠️ |
| **3. Autorização & Multi-tenant (100% rotas)** | `test_autorizacao_rotas.py` + `test_h19_isolamento_tenant.py` (`[QA-09]` / `[QA-08]`) | **155 passed**, **68 xfailed (`strict=True`)** (`4` eixos: sem token `401`, inválido `401`, RBAC `403`, cross-tenant/terceirizada `404`; divergência `analista_sst` registrada) | 100% rotas testadas | ✅ |
| **4. Falhas e Timeouts** | `test_falhas_e_timeouts.py` | **30 passed**, **2 xfailed (`strict=True`)** (Firestore `503`, timeouts/erros Supabase Storage, payloads inválidos, regressões `#112` e `#114`) | Tratamento resiliente sem vazar stack trace | ✅ |
| **5. Carga Local (`k6` 20 VUs × 60 s)** | `carga/k6_carga_local.js` (ADR 0010 / ADR 0012) | **2.509 reqs** (`41,01 req/s`), **0,00% falhas**, `p95` escrita **`1.063,27 ms`** (`<= 1.500 ms` ✅), `p95` leitura **`1.507,45 ms`** (`> 800 ms` ⚠️) | `p95` leitura `<= 800 ms`, escrita `<= 1.500 ms`, erro `< 1%` | ⚠️ |
| **6. Homologação E2E via API (`[QA-05]`)** | `carga/executar_homologacao_e2e.py` (Docker Compose + Firebase Auth Emulator) | **38/38 verificações aprovadas** (`0` falhas) nos 4 perfis (`admin_contratante`, `analista_sst`, `terceirizada`, `porteiro`), RBAC e isolamento | Fluxos J1–J3 + isolamento e RBAC aprovados | ✅ |

---

## 2. Detalhamento da Cobertura de Código (`coverage-cp4.xml`)

| Módulo | Statements | Miss | Cobertura |
|---|---:|---:|---:|
| `src/habilita/armazenamento/porta.py` | 50 | 0 | 100% |
| `src/habilita/armazenamento/supabase.py` | 93 | 0 | 100% |
| `src/habilita/cadastros/repositorio.py` | 248 | 1 | 99% |
| `src/habilita/cadastros/rotas.py` | 51 | 0 | 100% |
| `src/habilita/cadastros/schemas.py` | 76 | 0 | 100% |
| `src/habilita/trabalhadores/repositorio.py` | 397 | 16 | 96% |
| `src/habilita/trabalhadores/rotas.py` | 112 | 1 | 99% |
| `src/habilita/trabalhadores/schemas.py` | 92 | 1 | 99% |
| `src/habilita/comum/auth.py` | 38 | 1 | 97% |
| `src/habilita/comum/erros.py` | 64 | 0 | 100% |
| `src/habilita/comum/firestore.py` | 49 | 0 | 100% |
| `src/habilita/config.py` / `main.py` / `saude` / `sessao` | 132 | 0 | 100% |
| `src/habilita/__main__.py` | 7 | 7 | 0% |
| **Total Domínio** (`cadastros/*` + `trabalhadores/*` + `armazenamento/porta.py`) | **1.026** | **19** | **98,1%** *(98,15%)* |
| **Total Global** | **1.409** | **27** | **98,0%** *(98,08%)* |

---

## 3. Catálogo de Defeitos (`BUG-001`..`BUG-014`) e Triagem (`[PO-15]`)

Classificação de severidade conforme [Estratégia de Testes §11](../estrategia.md#11-severidade-de-defeitos): **0 Críticos abertos**, **3 Altos abertos**, **5 Médios abertos**, **1 Baixo aberto** e **5 Resolvidos** em `b7da8bf`/`b8fef43`.

| ID | Severidade | Componente / Jornada | Descrição / Passos / Esperado vs Obtido | Regressão / Evidência | Status |
|---|---|---|---|---|---|
| `BUG-001` | **Alta** | `cadastros/repositorio.py` · J1 (`H01.CA3`) | `criar_contrato` fixava `vigente=True` para `dataFim` no passado (esperado `vigente=False`, RN-10) | `test_h01_ca3_contrato_com_data_fim_no_passado_nao_fica_vigente` | ✅ **Resolvido (`b8fef43`)** |
| `BUG-002` | **Média** | `cadastros/repositorio.py` · J1 (`H08.CA3`) | `POST /terceirizadas` permitia CNPJ duplicado no tenant (`201` em vez de `409`) | `test_h08_ca3_cnpj_duplicado_no_mesmo_tenant_retorna_409` | ✅ **Resolvido (`b8fef43`)** |
| `BUG-003` | **Média** | `trabalhadores/repositorio.py` · J2 (`H10.CA3`) | `POST /contratos/{id}/alocacoes` permitia vínculo duplicado (`201` em vez de `409`) | `test_h10_ca3_vincular_trabalhador_duplicado_no_mesmo_contrato_retorna_409` | ✅ **Resolvido (`b8fef43`)** |
| `BUG-004` | **Alta** | `trabalhadores/rotas.py` · J3 (`H13.CA3`) | `POST /documentos/{id}/aprovacao` aceitava `validoAte < dataEmissao` (`200` em vez de `400`) | `test_h13_ca3_aprovar_documento_com_valido_ate_anterior_data_emissao_rejeitado` | ✅ **Resolvido (`b8fef43`)** |
| `BUG-005` | **Alta** | `trabalhadores/schemas.py` · J2/J3 (`H02.CA8`, `H02.CA10`, `H03.CA2`) | Upload aceitava `dataEmissao` futura e `validoAte < dataEmissao`; rejeição aceitava só espaços | `test_j1_j3_criterios_complementares.py` e `test_pr32_revisao_criterios.py` | ✅ **Resolvido (`b7da8bf`)** |
| `BUG-006` | **Alta** | `trabalhadores/repositorio.py:691-702` · J2 (`H02`) | `criar_documento` engole `ArmazenamentoError` com `app_env == "local"` e grava metadado órfão | `test_regressao_112_criar_documento_nao_engole_armazenamento_error_nem_grava_orfao` (`xfail`) | 🔴 **Aberto (`#112`)** |
| `BUG-007` | **Alta** | `trabalhadores/repositorio.py` · J3 (`H13`, `H03`) | Aprovação/rejeição e recálculo de semáforo sem `WriteBatch` atômico | Análise de código / Issue `#113` | 🔴 **Aberto (`#113`)** |
| `BUG-008` | **Média** | `comum/auth.py:59-60` · Transversal (`H07.CA3`) | `extrair_usuario` expõe mensagem crua de exceção Firebase no `detail` do HTTP `401` | `test_regressao_114_auth_401_nao_expoe_mensagem_crua_de_excecao_firebase` (`xfail`) | 🔴 **Aberto (`#114`)** |
| `BUG-009` | **Média** | `.github/workflows/job-diario.yml` · J4 (`H15`) | Timeout/keep-alive ausente no workflow diário contra cold start / pausa Supabase | Issue `#116` | 🔴 **Aberto (`#116`)** |
| `BUG-010` | **Alta** | `trabalhadores/rotas.py:95,211` · RBAC (`H19.CA2`) | `POST /trabalhadores` e `POST /trabalhadores/{id}/documentos` aceitam `analista_sst` (diverge de `x-papeis` no `openapi.yaml`) | `test_autorizacao_rotas.py` (`xfail strict=True`) | 🔴 **Aberto (CP4 Cat. 3)** |
| `BUG-011` | **Média** | `trabalhadores/schemas.py:68` · Contrato J2/J3 | `Pendencia` retorna `tipoNome` em vez de `tipoDocumentoNome` do `openapi.yaml` | `test_contrato_schema_pendencia_usa_tipo_documento_nome` (`xfail`) | 🔴 **Aberto (CP4 Cat. 2)** |
| `BUG-012` | **Média** | `trabalhadores/rotas.py:289` · Contrato J3 | `GET /documentos` usa query param `statusRevisao` em vez de `status` do `openapi.yaml` | `test_contrato_get_documentos_filtra_por_query_param_status` (`xfail`) | 🔴 **Aberto (CP4 Cat. 2)** |
| `BUG-013` | **Média** | `trabalhadores/repositorio.py` · Carga (`RNF04`) | `p95` leitura `1.507,45 ms` (`> 800 ms`) com 20 VUs por `.stream()` sem `.limit()` sob `cpus: 0.5` | [k6-resumo-cp4.json](k6-resumo-cp4.json) | 🔴 **Aberto (CP4 Cat. 5)** |
| `BUG-014` | **Baixa** | `trabalhadores/repositorio.py` · J3 (`H12.CA1`, `H12.CA3`) | Fila ordena por `id` em vez de `enviadoEm`; fallback em memória usa 15 min (Supabase usa `<= 300 s`) | `test_h03_ca1_analista_revisa_fila_e_gera_url_download` | 🔴 **Aberto (CP3/J3)** |

---

## 4. Escopo Pendente Dependente de Terceiros e Links para Artefatos

- **Frontend Web (`PR #55`):** validação visual e E2E Playwright de `H02.CA11`, `H09.CA3`, `H12.CA2`, `H07.CA1` e Lighthouse `>= 90`.
- **Rotas de Backend Pendentes (8/29 operações OpenAPI):** `/contratos/{contratoId}/conformidade`, `/painel/contratos`, `/alertas`, `/alertas/{alertaId}/leitura`, `/liberacoes/consultas`, `/acessos`, `/auditoria`, `/internal/jobs/recalculo-vencimentos`.
- **Cloud Run (`[QA-08]` #84):** execução de `test_autorizacao_rotas.py` com `HABILITA_BASE_URL` e smoke de carga conforme [ADR 0012](../../arquitetura/adr/0012-teto-teste-carga-producao.md) após publicação da URL.
- **Artefatos brutos (`[QA-10]`):** [junit-cp4.xml](junit-cp4.xml) · [coverage-cp4.xml](coverage-cp4.xml) · [k6-resumo-cp4.json](k6-resumo-cp4.json) · [homologacao-e2e-cp4.json](homologacao-e2e-cp4.json).

> Veja a análise detalhada e os passos completos de reprodução de cada defeito em [relatorio-a012ee3.md](../relatorio-a012ee3.md).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/11/2026 | @CauaRVeloso | Consolidação v1 do Checkpoint 4 (`a012ee3`) com métricas das 5 categorias, homologação E2E `[QA-05]`, catálogo `BUG-001`..`BUG-014` (`[PO-15]`) e artefatos exportados (`[QA-10]`). |
