# Relatório de QA — Checklist do Checkpoint 1 e Isolamento Multi-tenant

| Campo | Valor |
|---|---|
| **Marco** | Checkpoint 1 — Fundação (Fim da Semana 2 — 18/10/2026) |
| **Papel Responsável** | Quality Assurance (QA) |
| **Histórias Vinculadas** | [H19 — Isolar dados por cliente e papel](../../produto/requisitos/historias-backlog.md#h19) e [H07 — Entrar no sistema com o meu papel](../../produto/requisitos/historias-backlog.md#h07) |
| **Requisito Não Funcional** | [RNF01 — Autorização por cliente e papel em 100% das rotas](../../produto/requisitos/nao-funcionais.md) |
| **Decisões de Arquitetura** | [ADR 0002](../../arquitetura/adr/0002-tenant-autorizacao.md), [ADR 0006](../../arquitetura/adr/0006-armazenamento-de-documentos.md) e [ADR 0011 (I6)](../../arquitetura/adr/0011-divergencias-caderno.md) |
| **Repositório do Código** | `apps/api/tests/test_h19_isolamento_tenant.py` e `apps/api/tests/conftest.py` ([Habilita #18](https://github.com/projeto-habilita/Habilita/pull/18), mesclado na `main`) |
| **Status da Suíte (`main`)** | ✅ 2 passed, 15 xfailed (`strict=True`, 17 testes em `test_h19_isolamento_tenant.py` — [CI api-testes](https://github.com/projeto-habilita/Habilita/actions/runs/42757579962/job/129635500826)) |

---

## 1. Visão Geral e Contexto

Conforme o documento oficial *Checkpoint 1 — Guia para os POs* (seções *Checklist do resto do time: QA* e *Itens específicos de cada projeto*):

> **QA: começa já**  
> *"Teste de isolamento de tenant escrito (nos projetos com várias empresas clientes). Ele verifica que um usuário da empresa A recebe erro 403 ou 404 ao acessar dados da empresa B. O teste pode falhar agora, porque o sistema ainda não existe, mas ele precisa estar escrito."*  
>  
> **Habilita:**  
> *"ADRs de multi-tenancy e de RBAC; primeiros testes de isolamento de tenant. Em palavras simples: Como separar os dados de cada empresa e quem pode fazer o quê."*

Este relatório consolida a execução do checklist de QA para o marco inicial, a especificação e codificação da suíte automatizada de isolamento multi-tenant (casos **T-ISO-01** a **T-ISO-10** e provas adicionais de autenticação/RBAC mescladas no [Habilita #18](https://github.com/projeto-habilita/Habilita/pull/18)) e as evidências de conformidade com as regras arquiteturais do repositório.

---

## 2. Checklist de QA (Checkpoint 1)

| Item de Verificação | Critério no Guia do PO | Status | Evidência / Implementação |
|---|---|:---:|---|
| **Teste de isolamento de tenant escrito** | Escrito na fundação (mesmo falhando) | ✅ **Concluído** | Arquivo `apps/api/tests/test_h19_isolamento_tenant.py` implementado com 17 casos automatizados usando `pytest` e `FastAPI TestClient` (mesclado na `main` em [Habilita #18](https://github.com/projeto-habilita/Habilita/pull/18)). |
| **Verificação de erro 404 / 403 entre tenants** | Usuário de A recebe 404/403 ao acessar dados de B | ✅ **Concluído** | Coberto nos casos `T-ISO-01`, `T-ISO-02`, `T-ISO-04`, `T-ISO-05`, `T-ISO-07`, `T-ISO-09`, `T-ISO-10` e `analista_sst_altera_exigencias`. Conforme ADR 0002, recurso de outro tenant responde `404` (não revela existência). |
| **Isolamento de Terceirizadas (intra-tenant)** | Usuário de terceirizada X não acessa dados da terceirizada Y | ✅ **Concluído** | Caso `T-ISO-05` valida que a Terceirizada 1 de Alfa recebe `404` ao consultar trabalhador alocado à Terceirizada 2 de Alfa. |
| **Padrão de nomenclatura da DoD** | Nome do teste referencia o critério de aceite | ✅ **Concluído** | Nomes no padrão `test_h19_ca<N>_...` / `test_h19_...` estabelecido em `AGENTS.md` (ex.: `test_h19_ca1_t_iso_01_gestor_acessa_recursos_outro_tenant_retorna_404`). |
| **Isolamento por claims e proibição de `tenantId` na requisição** | `tenantId` vem do token; rejeita injeção | ✅ **Concluído** | Casos `T-ISO-03a` a `T-ISO-03d` validam que tentativas de injeção de `tenantId="tnt_beta"` via corpo, query param, cabeçalho ou cabeçalho simulado divergente são neutralizadas. |
| **Segurança das regras deny-all (Firestore e Storage)** | Acesso direto ao Firestore/Storage sem API é negado | ✅ **Concluído** | Casos `T-ISO-06a` (Firestore emulator) e `T-ISO-06b` (bucket privado no Supabase Storage sem política para `anon`, conforme ADR 0006 e ADR 0011 I6) executados e aprovados no CI. |

---

## 3. Matriz de Testes de Isolamento (História H19 e H07)

A suíte implementada traduz a Seção 5 da [Estratégia de Testes](../estrategia.md) e o mapeamento de cenários do seed determinístico:

| ID | Critério | Descrição do Cenário | Dados do Seed Utilizados | Esperado | Status na `main` (`Habilita #18`) |
|---|---|---|---|---|:---:|
| **T-ISO-01** | `H19.CA1` | Gestor de A acessa por ID recursos pertencentes exclusivamente a B (`/contratos/{id_B}`, `/trabalhadores/{id_B}`, `/documentos/{id_B}`). | `tnt_alfa` (`uid_gestor_alfa`) vs `ctr_beta_1`, `trb_beta_1`, `doc_beta_1_aso` | HTTP **404 Problem Details** (RFC 9457), corpo sem dados de B. | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-02** | `H19.CA1` | Gestor de A lista contratos, trabalhadores, documentos, alertas e auditoria. | `tnt_alfa` (`uid_gestor_alfa`) vs coleções de `tnt_beta` | HTTP **200** trazendo apenas itens de `tnt_alfa`. Nenhum item de B. | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-03a..03d** | `H19.CA5` | Gestor de A tenta injetar `tenantId: "tnt_beta"` no corpo (`03a`), query (`03b`), header (`03c`) e `X-Simulated-Tenant-Id` divergente (`03d`) ao criar contrato. | `uid_gestor_alfa` enviando `tenantId="tnt_beta"` | HTTP **400/422** (rejeitado por `extra="forbid"`) ou **201** criado em A. Nada criado em B. | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-04** | `H19.CA1` | Portaria de A consulta via POST CPF que só existe em B (`trb_beta_1`). | Porteiro de A vs CPF `20200000160` | HTTP **200** com `resultado: "NAO_CADASTRADO"`, sem expor nome ou dados de B. | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-05** | `H19.CA3` | Terceirizada 1 de A tenta acessar trabalhador da Terceirizada 2 de A. | `uid_terc_alfa_1` (`ter_alfa_1`) vs `trb_alfa_y1` (`ter_alfa_2`) | HTTP **404 Problem Details** (isolamento horizontal intra-tenant). | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-06a** | `H19.CA4` | Tentativa de leitura e escrita direta no Cloud Firestore sem passar pela API. | Prova em tempo de execução contra o emulador local do Firestore (`:8080`) | HTTP **403** (`PERMISSION_DENIED`) pelas Security Rules deny-all. | **PASSED** |
| **T-ISO-06b** | `H19.CA4` | Tentativa de listagem, leitura e escrita direta no Storage sem passar pela API (anônima ou com chave `anon`). | Bucket privado `documentos` no Supabase Storage local (ADR 0006 / ADR 0011 I6; fallback Firebase Storage `:9199`) | HTTP **400/401/403/404** (sem política pública para `anon`). | **PASSED** |
| **T-ISO-07** | `H19.CA1` | Gestor de A tenta gerar URL assinada de download para documento de B. | `uid_gestor_alfa` vs `doc_beta_1_aso` | HTTP **404 Problem Details** (bloqueio de geração de URL assinada para outro tenant). | **XFAIL** (`strict=True`, rotas S4) |
| **T-ISO-08** | `H07.CA2` / `H19` | Requisições a rotas protegidas sem token de autenticação. | Cliente anônimo sem header `Authorization` | HTTP **401 Unauthorized** com payload Problem Details (RFC 9457). | **XFAIL** (`strict=True`, auth S4) |
| **T-ISO-09** | `H19.CA2` | Usuário com papel inadequado tenta executar ação restrita (RBAC). | Porteiro criando contrato ou Terceirizada aprovando documento | HTTP **403 Forbidden** com payload Problem Details (RFC 9457). | **XFAIL** (`strict=True`, RBAC S4) |
| **T-ISO-10** | `H19.CA1` | Acesso a recursos com IDs semelhantes, colisões de chaves ou path traversal. | `tnt_alfa` vs `ctr_beta_1`, prefixos e `../tnt_beta/...` | HTTP **404 Problem Details** sem vazamento por correspondência parcial. | **XFAIL** (`strict=True`, rotas S4) |
| **Token inválido / expirado** | `H07.CA3` / `H19` | Chamada com token JWT com assinatura inválida ou expirado. | Header `Authorization` adulterado ou expirado | HTTP **401 Unauthorized** com payload Problem Details (RFC 9457). | **XFAIL** (`strict=True`, auth S4) |
| **`analista_sst` na matriz** | `H19.CA2` | Usuário `analista_sst` tenta editar matriz de exigências de contrato (ação exclusiva de `admin_contratante`). | `analista_sst` em `PUT /contratos/{id}/exigencias` | HTTP **403 Forbidden** com payload Problem Details (ADR 0002). | **XFAIL** (`strict=True`, RBAC S4) |

!!! info "Nota sobre os testes com status XFAIL e prevenção de falsos positivos"
    Conforme orientações de QA e da ADR 0002, testes que validam regras de isolamento em rotas ainda não implementadas na `main` utilizam `@pytest.mark.xfail(strict=True)` (e `xfail_strict = true` no `pyproject.toml`). Isso evita falsos positivos (como o FastAPI retornar 404 genérico de rota não registrada em vez do 404/401/403 com `application/problem+json` emitido pela camada de autorização/guard de tenant). Com a entrega das rotas de negócio (como no PR `Habilita #32`), os testes deixam o estado `xfail` e passam diretamente na suíte.

---

## 4. Evidências de Execução Automática

### 4.1 Execução da Suíte de Testes no CI (`Habilita #18` mesclado na `main`)

Execução via Docker Compose no job `api-testes` (`docker compose exec -T api pytest tests -v`) — [Run 42757579962 / Job 129635500826](https://github.com/projeto-habilita/Habilita/actions/runs/42757579962/job/129635500826):

- **`test_h19_isolamento_tenant.py` — 17 testes coletados (`2 passed, 15 xfailed` com `xfail_strict=true`):**
  1. `test_h19_ca1_t_iso_01_gestor_acessa_recursos_outro_tenant_retorna_404` → `XFAIL`
  2. `test_h19_ca1_t_iso_02_gestor_lista_recursos_nao_enxerga_outro_tenant` → `XFAIL`
  3. `test_h19_ca5_t_iso_03a_injecao_tenant_id_no_corpo_rejeitada_ou_isolada` → `XFAIL`
  4. `test_h19_ca5_t_iso_03b_injecao_tenant_id_na_query_rejeitada_ou_isolada` → `XFAIL`
  5. `test_h19_ca5_t_iso_03c_injecao_tenant_id_no_header_rejeitada_ou_isolada` → `XFAIL`
  6. `test_h19_ca5_t_iso_03d_header_simulado_com_token_divergente_rejeitado_ou_isolado` → `XFAIL`
  7. `test_h19_ca1_t_iso_04_portaria_consulta_cpf_outro_tenant_retorna_nao_cadastrado` → `XFAIL`
  8. `test_h19_ca3_t_iso_05_terceirizada_acessa_trabalhador_outra_terceirizada_retorna_404` → `XFAIL`
  9. `test_h19_ca4_t_iso_06a_acesso_direto_firestore_sem_api_rejeitado_deny_all` → **`PASSED`**
  10. `test_h19_ca4_t_iso_06b_acesso_direto_storage_sem_api_rejeitado_deny_all` → **`PASSED`**
  11. `test_h19_ca1_t_iso_07_gestor_gera_url_download_documento_outro_tenant_retorna_404` → `XFAIL`
  12. `test_h19_t_iso_08_requisicao_sem_token_retorna_401_problem_details` → `XFAIL`
  13. `test_h19_ca2_t_iso_09_papel_inadequado_retorna_403_problem_details` → `XFAIL`
  14. `test_h19_ca1_t_iso_10_ids_parecidos_ou_colidentes_entre_tenants_retorna_404` → `XFAIL`
  15. `test_h19_token_invalido_retorna_401_problem_details` → `XFAIL`
  16. `test_h19_token_expirado_retorna_401_problem_details` → `XFAIL`
  17. `test_h19_ca2_analista_sst_altera_exigencias_retorna_403` → `XFAIL`

### 4.2 Verificações de Tipagem, Lint e Arquitetura

- **Pipeline CI do `Habilita #18`:** `api-qualidade` (`ruff check`, `ruff format --check`, `mypy`, `lint-imports`), `api-testes`, `seed-qualidade`, `dockerfiles`, `compose-smoke`, `segredos`, `higiene`, `web-qualidade`, `web-build` e `ci-ok` concluídos com sucesso.

---

## 5. Conclusão e Próximos Passos

Com a suíte mesclada na `main` pelo [Habilita #18](https://github.com/projeto-habilita/Habilita/pull/18):
1. O **gate do Checkpoint 1 para o papel de QA está cumprido**, respeitando as regras de isolamento multi-tenant da ADR 0002 e o armazenamento privado no Supabase Storage (ADR 0006 / ADR 0011 I6).
2. Com a integração das rotas de negócio (em revisão no `Habilita #32`), os testes marcados com `xfail(strict=True)` em `test_h19_isolamento_tenant.py` passam a validar diretamente a aplicação real e previnem regressões de segurança.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 17/10/2026 | @CauaRVeloso | Versão inicial do relatório de QA do Checkpoint 1 e testes de isolamento `T-ISO-01` a `T-ISO-07`. |
| 1.1 | 17/11/2026 | @CauaRVeloso | Remoção da referência inexistente `H19.CA6` (mapeada para `H19.CA2` e `H07.CA2`/`CA3`), atualização do `T-ISO-06b` para Supabase Storage (ADR 0011 I6) e sincronização dos 17 testes mesclados no `Habilita #18`. |
