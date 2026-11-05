# ADR 0002 — Multi-tenant e autorização

| Campo | Valor |
|---|---|
| Status | **Aceito** — aprovado em PR (Tech Lead + 1 integrante) na Sprint 4, com os 4 papéis da ficha (feedback do Checkpoint 1, item 7) |
| Data | 14/10/2026 |
| Decisores | Tech Lead, Backend, Frontend |
| Consultados | PO |
| Relacionados | ADR 0001 (stack), ADR 0008 (consulta por CPF), RNF01, H19, H07, [Modelo de dados](../dados.md) |

## 1. Contexto

O Habilita atende várias empresas contratantes (tenants). Os dados de uma **nunca** podem aparecer para outra: é o requisito de segurança mais importante do produto e um item explícito do gate (teste de isolamento escrito já no CP1; Security Rules deny-all com testes no CP2; autorização em 100% das rotas no CP4, RNF01).

A ficha do Habilita define **quatro papéis** com permissões diferentes — **admin do contratante**, **analista de SST**, **prestadora** (terceirizada) e **portaria** — e diz que o analista **não edita a matriz de exigências**. Para a terceirizada há um segundo nível de isolamento: ela só vê os próprios trabalhadores dentro do cliente. O Guia do PO pede, para o Habilita, decisões explícitas de **multi-tenancy** e de **RBAC**; este ADR registra as duas.

## 2. Opções consideradas

### 2.1 Isolamento dos dados

| Opção | Prós | Contras |
|---|---|---|
| Um projeto Firebase por cliente | Isolamento máximo | Inviável no orçamento e na operação de um MVP |
| Coleções planas com campo `tenantId` | Simples | Um `where` esquecido vaza dados; isolamento depende de disciplina |
| **Tenant no caminho** (`tenants/{tenantId}/...`) | Isolamento estrutural; repositório só monta caminhos do próprio tenant | Consultas *collection group* atravessam tenants e exigem cuidado |

### 2.2 Autorização

| Opção | Prós | Contras |
|---|---|---|
| Navegador acessa o Firestore direto, autorização nas Security Rules | Menos código de API | Regras de RBAC + tenant + terceirizada complexas, difíceis de testar; o cliente vê a estrutura do banco |
| Papéis numa coleção `usuarios` lida a cada requisição | Flexível | Uma leitura extra por requisição (custo e latência) |
| **Custom claims no token + autorização centralizada na API + Rules deny-all** | Um ponto de decisão testável; zero leitura extra; Rules triviais | Mudança de papel só vale quando o token renova (até 1 h) |

## 3. Decisão

**Multi-tenancy**

- Todo dado de negócio fica sob **`tenants/{tenantId}/...`** no Firestore; arquivos no Storage seguem o mesmo prefixo.
- Repositórios recebem o `tenantId` **no construtor**, vindo da claim do token; nenhum método consegue ler outro tenant.
- Recurso de outro tenant pedido por ID retorna **404** (não revela que existe). *Divergência consciente do rascunho de escopo, que citava 403 — ver relatório de conciliação.*
- Consultas *collection group* só são permitidas no job interno (ADR 0007).

**Autenticação e RBAC**

- **Firebase Auth** (e-mail e senha no MVP).
- Claims customizadas: `tenantId`, `papel` (`admin_contratante` | `analista_sst` | `terceirizada` | `porteiro`) e, para terceirizada, `terceirizadaId`. Atribuídas só pelo script de seed ou por rotina administrativa com credencial de serviço. Os valores `terceirizada` e `porteiro` correspondem a "prestadora" e "portaria" da ficha; mantivemos os nomes já usados no código e no contrato.
- **Separação de funções:** só `admin_contratante` cria contratos, edita a **matriz de exigências** e o catálogo de tipos de documento; `analista_sst` revisa documentos (aprova ou rejeita) e acompanha semáforo e alertas, mas **não edita a matriz**. Quem define a regra não é o mesmo papel que, no dia a dia, julga o documento; o admin também pode revisar, para clientes pequenos com uma pessoa só no SST.
- A API **verifica o ID token** em toda rota protegida e autoriza em duas etapas: **papel permitido na rota** (senão 403) e **escopo do recurso** — tenant e, quando aplicável, terceirizada (senão 404).
- **Security Rules deny-all** no Firestore e no Storage: o navegador nunca acessa dados diretamente.
- A matriz rota × papel é declarada no `openapi.yaml` (extensão `x-papeis`) e testada automaticamente.

### 3.1 Matriz de papéis (resumo; a fonte é o `x-papeis` do `openapi.yaml`)

| Capacidade | `admin_contratante` | `analista_sst` | `terceirizada` | `porteiro` |
|---|:-:|:-:|:-:|:-:|
| Cadastrar terceirizadas, tipos de documento e contratos (J1) | ✅ | ❌ | ❌ | ❌ |
| **Editar a matriz de exigências** (J1) | ✅ | ❌ | ❌ | ❌ |
| Ver contratos, trabalhadores, documentos e semáforo | ✅ | ✅ | ✅ (só os seus) | ❌ |
| Cadastrar trabalhadores, vincular a contratos e enviar documentos (J2) | ✅ | ❌ | ✅ (só os seus) | ❌ |
| Aprovar ou rejeitar documento com motivo (J3) | ✅ | ✅ | ❌ | ❌ |
| Alertas de vencimento (J4) | ✅ | ✅ | ❌ | ❌ |
| Consultar liberação por CPF (J5) | ✅ | ✅ | ❌ | ✅ |
| Registrar entrada ou barramento (J5) | ❌ | ❌ | ❌ | ✅ |
| Trilha de auditoria | ✅ | ❌ | ❌ | ❌ |

Papel fora da lista da rota → **403**. Recurso de outro tenant ou de outra terceirizada → **404**.

## 4. Consequências

**Positivas**

- Isolamento estrutural, não dependente de filtro; exclusão ou exportação por cliente fica trivial.
- Um único ponto de decisão de acesso, testável; teste de isolamento simples de escrever.
- Rules de uma linha, fáceis de auditar.

**Negativas / custos**

- Mais código na API (toda leitura e escrita passa por ela).
- Revogação de papel não é imediata (até 1 h) sem revogar refresh tokens — aceito no MVP.
- Análises globais entre clientes ficam mais caras (não são necessárias no MVP).

**Obrigatório a partir desta decisão**

- Proibido montar caminho do Firestore fora dos repositórios.
- Proibido aceitar `tenantId` ou `terceirizadaId` vindos da requisição.
- Nenhuma rota nova sem papel declarado em `x-papeis` e sem teste de autorização.

## 5. Riscos e mitigação

| Risco | Mitigação |
|---|---|
| *Collection group* usado em rota de usuário vaza dados | Revisão de PR reprova `collectionGroup` fora do módulo do job; testes T-ISO |
| Rota nova esquecida na matriz de autorização | Suíte de autorização gerada a partir do `openapi.yaml`; rota sem `x-papeis` falha o CI |
| Claims erradas no seed | Seed versionado e revisado; teste que confere claims dos usuários de demonstração |

## 6. Validação da decisão

- [ ] Testes de isolamento T-ISO-01 a T-ISO-07 **escritos** até 18/10 (podem falhar) e **passando** até 01/11.
- [ ] Matriz de autorização cobrindo 100% das rotas com os **4 papéis** (CP2 e CP4), incluindo o caso "`analista_sst` tenta editar a matriz → 403".
- [ ] Seed com um usuário de cada um dos 4 papéis nos 2 tenants.
- [ ] Teste das Security Rules no emulador negando leitura e escrita diretas.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: tema e formato do ADR. Do Tech Lead: ADRs de multi-tenancy e RBAC fundidos, com 404 entre clientes (C8, C16). |
| 1.1 | 26/10/2026 | @osamuelleite | Feedback do CP1 (item 7): `gestor` separado em `admin_contratante` e `analista_sst`, como na ficha; matriz de papéis; o analista não edita a matriz. Status Aceito. |
