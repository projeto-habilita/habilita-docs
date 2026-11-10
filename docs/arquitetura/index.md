# Arquitetura

| Campo | Valor |
|---|---|
| Status | Rascunho |
| Responsável | Tech Lead |
| Última atualização | 02/11/2026 |
| Notação | Modelo C4 (níveis 1 e 2) e diagramas de sequência em Mermaid |

## 1. Objetivos e restrições que moldam a arquitetura

| Força | Consequência na arquitetura |
|---|---|
| **Isolamento entre clientes** é o principal requisito de segurança | Tenant no caminho dos dados, tenant extraído do token, acesso a dados só pelo backend ([ADR 0002](adr/0002-tenant-autorizacao.md)) |
| **Teto de US$ 50** e uso intermitente | Serviços que escalam a zero (Cloud Run com `min-instances=0`, Firestore e Storage por uso) |
| **9 semanas e equipe pequena** | Monólito modular: uma API, um frontend, um banco. Nada de microsserviços ([ADR 0005](adr/0005-estilo-arquitetural.md)) |
| **Rodar tudo localmente até a Semana 4** | Docker Compose com emuladores do Firebase (Auth, Firestore, Storage) |
| **Resposta rápida na portaria** (p95 ≤ 800 ms) | Semáforo materializado; consulta por chave (hash do CPF) |
| **Documentos com dado de saúde** (ASO) | Bucket privado, download por link temporário, nenhum dado pessoal em log ([ADR 0006](adr/0006-armazenamento-de-documentos.md)) |

## 2. Contexto (C4 nível 1)

```mermaid
flowchart TB
    gestor["👤 Gestor de SST/Facilities<br/>(contratante)"]
    terc["👤 Responsável da terceirizada"]
    porteiro["👤 Porteiro"]
    habilita[["Habilita<br/>Plataforma de conformidade documental de SST"]]
    firebaseAuth["Firebase Authentication<br/>(identidade)"]
    email["Serviço de e-mail<br/>(Could · H17)"]

    gestor -- "configura contratos, aprova documentos,<br/>acompanha semáforo e alertas" --> habilita
    terc -- "cadastra trabalhadores,<br/>envia documentos" --> habilita
    porteiro -- "consulta liberação por CPF,<br/>registra acesso" --> habilita
    habilita -- "valida tokens e claims" --> firebaseAuth
    habilita -. "resumo diário de alertas" .-> email
```

## 3. Contêineres (C4 nível 2)

```mermaid
flowchart LR
    subgraph Navegador
        web["Web App (SPA React)<br/>3 áreas: Gestão, Terceirizada, Portaria"]
    end
    subgraph Firebase
        hosting["Firebase Hosting<br/>estáticos + rewrite /api/**"]
        auth["Firebase Auth<br/>login e custom claims"]
    end
    subgraph GCP
        api["API Habilita<br/>(Cloud Run, min-instances=0)<br/>camadas: aplicação, domínio, infra"]
        fs[("Firestore<br/>tenants/{tenantId}/…<br/>Rules: deny-all")]
        gcs[("Cloud Storage<br/>bucket privado<br/>Rules: deny-all")]
        sched["Cloud Scheduler<br/>03:00 America/Sao_Paulo"]
        sm["Secret Manager<br/>chave HMAC do CPF"]
    end

    web -- "login (SDK)" --> auth
    web -- "HTTPS /api/** + ID token" --> hosting
    hosting -- "rewrite" --> api
    api -- "verifica ID token (Admin SDK)" --> auth
    api -- "lê/escreve (Admin SDK)" --> fs
    api -- "grava arquivo / gera URL assinada" --> gcs
    sched -- "POST /internal/jobs/... (OIDC)" --> api
    api -- "lê segredo" --> sm
    web -. "baixa arquivo por URL assinada (5 min)" .-> gcs
```

**Regra de ouro:** o navegador **nunca** acessa Firestore ou Storage diretamente. As Security Rules negam tudo; só a API, com credencial de serviço, lê e escreve. Isso concentra autorização em um único lugar testável.

## 4. Camadas da API

A API segue a arquitetura em camadas estudada no lab, com a regra de dependência apontando para o domínio.

| Camada | Responsabilidade | Exemplos no Habilita | Pode depender de |
|---|---|---|---|
| **Apresentação** (web) | Telas por papel, estados de carregando, vazio, erro e sucesso | Painel com semáforo, fila de pendentes, tela de portaria | API (via contrato OpenAPI) |
| **Aplicação** (API) | Rotas HTTP, validação de entrada, autenticação, autorização, orquestração dos casos de uso | `RejeitarDocumento`, `ConsultarLiberacao`, middleware de token/tenant/papel | Domínio, portas de infraestrutura |
| **Domínio** | Regras de negócio puras, sem I/O | Cálculo de faixa (RN-04), semáforo (RN-06), reuso (RN-08), validação de CPF/CNPJ | Nada (100% testável em unidade) |
| **Dados / Infra** | Adaptadores de Firestore, Storage, Secret Manager, relógio, logger | Repositórios por tenant, gerador de URL assinada | SDKs externos |

```mermaid
flowchart TB
    A["Aplicação<br/>(HTTP, auth, casos de uso)"] --> D["Domínio<br/>(regras puras)"]
    A --> P["Portas (interfaces)<br/>RepositórioDocumentos, Armazenamento, Relógio"]
    I["Infra<br/>(Firestore, Storage, Secret Manager)"] -. implementa .-> P
```

O domínio não conhece o Firestore. Isso permite testar todas as regras com dados em memória e cumprir a meta de ≥ 80% de cobertura no domínio.

## 5. Fluxos principais

### 5.1 Consulta de liberação na portaria (J5)

```mermaid
sequenceDiagram
    autonumber
    actor P as Porteiro
    participant W as Web (Portaria)
    participant A as API
    participant F as Firestore
    P->>W: digita CPF
    W->>A: POST /api/liberacoes/consultas {cpf} + ID token
    A->>A: valida token, papel = porteiro, tenantId da claim
    A->>A: valida dígitos do CPF, calcula HMAC(cpf)
    A->>F: get tenants/{t}/indiceCpf/{hash}
    alt CPF não encontrado
        A->>F: grava consulta (NAO_CADASTRADO) + auditoria
        A-->>W: 200 {resultado: NAO_CADASTRADO, consultaId}
    else encontrado
        A->>F: lê alocações do trabalhador em contratos vigentes
        A->>A: aplica RN-10
        A->>F: grava consulta + auditoria (lote atômico)
        A-->>W: 200 {resultado: LIBERADO|BLOQUEADO, contratos, pendencias, consultaId}
    end
    P->>W: confirma ENTRADA ou BARRADO
    W->>A: POST /api/acessos {consultaId, decisao}
    A-->>W: 201
```

### 5.2 Envio e aprovação de documento (J2 e J3)

```mermaid
sequenceDiagram
    autonumber
    actor T as Terceirizada
    actor G as Gestor
    participant A as API
    participant S as Cloud Storage
    participant F as Firestore
    T->>A: POST /api/trabalhadores/{id}/documentos (multipart, ≤ 5 MiB, validoAte)
    A->>A: autoriza (RN-02), valida tamanho e assinatura de bytes
    A->>S: grava arquivo em caminho privado + SHA-256
    A->>F: cria documento PENDENTE + auditoria
    A-->>T: 201
    G->>A: GET /api/documentos?status=PENDENTE
    G->>A: POST /api/documentos/{id}/url-download
    A->>S: gera URL assinada (5 min)
    A-->>G: {url, expiraEm}
    G->>A: POST /api/documentos/{id}/aprovacao {validoAte?}
    A->>F: transação: PENDENTE→APROVADO, confirma validoAte (RN-09)
    A->>F: recalcula alocações do trabalhador (RN-13) + auditoria
    A-->>G: 200
```

### 5.3 Recálculo diário (J4)

```mermaid
sequenceDiagram
    autonumber
    participant C as Cloud Scheduler
    participant A as API (rota interna)
    participant F as Firestore
    C->>A: POST /internal/jobs/recalculo-vencimentos {dataReferencia} (token OIDC)
    A->>A: valida que o chamador é a conta de serviço do agendador
    A->>F: marca execucoesJob/{data} = EM_ANDAMENTO
    loop lotes de até 400 documentos aprovados
        A->>A: calcula faixa (RN-04) — função pura
        A->>F: grava situação e alerta com ID determinístico (RN-12)
        A->>F: recalcula alocações afetadas
    end
    A->>F: marca execucoesJob/{data} = CONCLUIDO com contagens
    A-->>C: 200
```

Detalhes da idempotência em [ADR 0007](adr/0007-recalculo-idempotente.md).

## 6. Como a arquitetura atende os requisitos não funcionais

Os requisitos são definidos em [Requisitos Não Funcionais](../produto/requisitos/nao-funcionais.md). Esta tabela mostra **onde** cada um é atendido e **como** é verificado.

| RNF | Onde a arquitetura atende | Como verificar |
|---|---|---|
| RNF01 · Autorização por tenant e papel | Tenant no caminho, claims no token, Rules deny-all (ADR 0002) | Suíte de isolamento e matriz de autorização no CI |
| RNF02 · Upload seguro | Validação por assinatura de bytes, bucket privado, URL assinada (ADR 0006) | Testes T-UPL |
| RNF03 · Zero vulnerabilidade crítica/alta | Dependências mínimas; varredura no pipeline (ADR 0003) | Varredura de dependências, imagens e segredos |
| RNF04 · p95 leitura ≤ 800 ms / escrita ≤ 1,5 s | Semáforo materializado; consulta por chave (ADR 0007, ADR 0008) | Logs de latência por rota; k6 |
| RNF05 · Cobertura | Domínio puro, sem I/O (ADR 0005) | Relatório de cobertura em `docs/testes/relatorios` |
| RNF06 · Lighthouse ≥ 90 | SPA enxuta no Hosting com CDN | Lighthouse CI |
| RNF07 · Operação | `/healthz`, `/readyz`, logs JSON com `requestId` | Inspeção dos logs; alertas 5xx e p95 |
| RNF08 · Custo | Escala a zero, `max-instances` baixo, paginação | [FinOps](../finops/index.md) |
| RNF09 · Sem dado real em teste e demo | Seed fictício reproduzível | Revisão do seed |
| RNF10 · Disponibilidade na demo | Smoke test pós-deploy; rollback (ADR 0003) | Checklist do Demo Day |
| RNF11 · Nenhum dado pessoal em log | CPF só como HMAC; corpo da rota de portaria não registrado (ADR 0008) | Teste sobre a saída de log |
| RNF12 · Reprodutibilidade | Docker Compose + emuladores | Subida em máquina limpa por outro integrante |
| RNF13 · Portaria no celular | Tela de um campo, cor acompanhada de texto | Revisão de UI e Lighthouse |

## 7. Ambientes

| Ambiente | Quando | Composição | Dados |
|---|---|---|---|
| **Local** | S1 a S4 (e sempre para desenvolvimento) | Docker Compose: web, API, emuladores Auth/Firestore/Storage | Seed fictício reproduzível |
| **Preview** | A partir da S6, por PR | Canal de preview do Hosting + revisão do Cloud Run com tag | Seed fictício |
| **Produção** | A partir da S6, após merge na `main` | Hosting + Cloud Run + Firestore + Storage | Seed fictício para a demo |

## 8. Riscos técnicos

| Risco | Impacto | Mitigação |
|---|---|---|
| **Cold start** do Cloud Run com `min-instances=0` estoura o p95 de 800 ms na primeira consulta | Médio | Medir cedo; imagem pequena; inicialização preguiçosa de SDKs; se necessário, `min-instances=1` só no dia da demo, registrado em ADR e no relatório de custo |
| **Emulador de Storage** pode não reproduzir URLs assinadas como em produção | Médio | *Spike* na S3 para validar; plano B: download servido pela API em modo local, por configuração |
| Consultas *collection group* sem filtro de tenant expõem dados | Alto | Restritas ao job interno; revisão de código obrigatória; teste de isolamento |
| Custo de leituras do Firestore no job diário | Baixo/Médio | Ler só documentos aprovados; lotes; volume de demo pequeno; alerta de orçamento |
| Rota síncrona (SDK `firebase-admin`) declarada como `async def` travar o servidor | Médio | Rotas com SDK síncrono declaradas com `def`; item no checklist de revisão (ADR 0001) |

## 9. Decisões pendentes

| Decisão | ADR | Prazo |
|---|---|---|
| Política de retenção de documentos | A criar | 15/11 |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Visão da arquitetura em camadas e diagramas de sequência, adaptados à estrutura da PO. |
| 1.1 | 02/11/2026 | @pedrolmelo | Ajuste do limite de upload no diagrama de sequência para ≤ 5 MiB (PO-06). |
