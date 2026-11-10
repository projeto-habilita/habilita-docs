# Modelo de dados e tenants

| Campo | Valor |
|---|---|
| Status | Rascunho |
| Responsável | Tech Lead (regras de negócio validadas pelo PO) |
| Última atualização | 02/11/2026 |
| Decisões relacionadas | [ADR 0002](adr/0002-tenant-autorizacao.md), [ADR 0006](adr/0006-armazenamento-de-documentos.md), [ADR 0007](adr/0007-recalculo-idempotente.md), [ADR 0008](adr/0008-consulta-liberacao-cpf.md), [ADR 0009](adr/0009-auditoria-append-only.md) |

## Modelo de domínio

```mermaid
erDiagram
    TENANT ||--o{ USUARIO : "tem"
    TENANT ||--o{ TERCEIRIZADA : "contrata"
    TENANT ||--o{ TIPO_DOCUMENTO : "define"
    TERCEIRIZADA ||--o{ CONTRATO : "executa"
    CONTRATO ||--o{ EXIGENCIA : "exige"
    TIPO_DOCUMENTO ||--o{ EXIGENCIA : "é exigido em"
    TERCEIRIZADA ||--o{ TRABALHADOR : "emprega"
    TRABALHADOR ||--o{ ALOCACAO : "é alocado"
    CONTRATO ||--o{ ALOCACAO : "aloca"
    TRABALHADOR ||--o{ DOCUMENTO : "possui"
    TIPO_DOCUMENTO ||--o{ DOCUMENTO : "classifica"
    DOCUMENTO ||--o{ ALERTA : "gera"
    TRABALHADOR ||--o{ CONSULTA_LIBERACAO : "é consultado"
    CONSULTA_LIBERACAO ||--o| ACESSO : "resulta em"
```

**Ponto central do design:** o **documento pertence ao trabalhador**, não ao contrato. O contrato apenas declara quais tipos exige. É isso que torna o reuso entre contratos (RN-08) automático, sem cópia de arquivo.

## Entidades

| Entidade | Descrição | Campos principais |
|---|---|---|
| **Tenant** | Empresa contratante (cliente) | `id`, `nome`, `criadoEm` |
| **Usuário** | Login no Firebase Auth com claims | `uid`, `tenantId`, `papel` (`gestor`, `terceirizada`, `porteiro`), `terceirizadaId` (só para papel `terceirizada`) |
| **Terceirizada** | Prestadora contratada pelo tenant | `id`, `cnpj`, `razaoSocial`, `ativa` |
| **TipoDocumento** | Catálogo do tenant | `id`, `nome`, `validadeMeses` (prazo padrão, usado para sugerir e conferir a validade) |
| **Contrato** | Vínculo tenant–terceirizada | `id`, `terceirizadaId`, `identificador`, `dataInicio`, `dataFim`, `exigencias[]` (ids de TipoDocumento), `versaoMatriz` |
| **Trabalhador** | Pessoa da terceirizada | `id`, `terceirizadaId`, `nome`, `cpfHash`, `cpfMascarado`, `ativo` |
| **Alocação** | Trabalhador em um contrato | `contratoId`, `trabalhadorId`, `semaforo`, `pendencias[]`, `calculadoEm` |
| **ContagemContrato** | Resumo do contrato para o painel | `contratoId`, `semaforo`, `totais` (`VERDE`, `AMARELO`, `VERMELHO`), `totalTrabalhadores`, `vencendoEm30Dias`, `dataReferencia`, `calculadoEm` |
| **Documento** | Arquivo de SST de um trabalhador | `id`, `trabalhadorId`, `tipoDocumentoId`, `dataEmissao`, `validoAte`, `statusRevisao`, `situacaoValidade`, `motivoRejeicao`, `arquivo` (`caminho`, `contentType`, `tamanhoBytes`, `sha256`), `enviadoPor`, `revisadoPor`, `revisadoEm` |
| **Alerta** | Aviso de vencimento | `id` determinístico, `documentoId`, `tipo` (`VENCE_EM_30_DIAS`, `VENCIDO`), `criadoEm`, `lidoEm` |
| **ConsultaLiberacao** | Consulta feita pela portaria | `id`, `trabalhadorId` (ou nulo), `resultado`, `porteiroUid`, `criadaEm`, `usadaEm` |
| **Acesso** | Entrada ou barramento registrado | `id`, `consultaId`, `decisao` (`ENTRADA`, `BARRADO`), `porteiroUid`, `registradoEm` |
| **RegistroAuditoria** | Evento imutável | `id`, `ator`, `papel`, `acao`, `recurso`, `detalhes`, `dataHora`, `requestId` |

## Coleções no Firestore

Todo dado de negócio vive sob `tenants/{tenantId}` ([ADR 0002](adr/0002-tenant-autorizacao.md)).

```
tenants/{tenantId}
├── terceirizadas/{terceirizadaId}
├── tiposDocumento/{tipoDocumentoId}
├── contratos/{contratoId}
│   └── alocacoes/{trabalhadorId}          ← semáforo materializado por contrato
├── contagens/{contratoId}                 ← resumo materializado do contrato (painel H23, 1 leitura)
├── trabalhadores/{trabalhadorId}
│   └── documentos/{documentoId}
├── indiceCpf/{cpfHash}                    ← { trabalhadorId }  garante unicidade e busca O(1)
├── alertas/{documentoId_tipo}             ← ID determinístico (idempotência)
├── consultasLiberacao/{consultaId}
├── acessos/{acessoId}
└── auditoria/{registroId}                 ← append-only
execucoesJob/{dataReferencia}              ← controle do job diário (fora dos tenants)
```

**Arquivos** no Cloud Storage (bucket privado):
`tenants/{tenantId}/trabalhadores/{trabalhadorId}/documentos/{documentoId}/{uuid}.{pdf|jpg}`

### Contagens do contrato (painel H23)

- **Para que serve:** O painel do contrato consulta **um único documento** em vez de varrer a lista de alocações de trabalhadores, atendendo ao critério de aceite H23.CA2 (painel com 500 trabalhadores carrega em até 2 segundos).
- **Documento:** `tenants/{tenantId}/contagens/{contratoId}`

| Campo | Tipo | Descrição |
|---|---|---|
| `contratoId` | string | Identificador do contrato a que a contagem se refere. |
| `semaforo` | string ou nulo | Pior cor entre os trabalhadores alocados ativos (`VERDE`, `AMARELO`, `VERMELHO`), ou nulo quando o contrato não tem trabalhadores alocados (RN-07). |
| `totais.VERDE` | inteiro (≥ 0) | Quantidade de trabalhadores com semáforo verde no contrato. |
| `totais.AMARELO` | inteiro (≥ 0) | Quantidade de trabalhadores com semáforo amarelo no contrato. |
| `totais.VERMELHO` | inteiro (≥ 0) | Quantidade de trabalhadores com semáforo vermelho no contrato. |
| `totalTrabalhadores` | inteiro (≥ 0) | Quantidade total de trabalhadores alocados no contrato (`VERDE + AMARELO + VERMELHO`). |
| `vencendoEm30Dias` | inteiro (≥ 0) | Quantidade de documentos aprovados de trabalhadores alocados que vencem nos próximos 30 dias (faixa `A_VENCER` da RN-04). |
| `dataReferencia` | string (`YYYY-MM-DD`) | Data utilizada como referência temporal para o cálculo dos vencimentos e semáforos. |
| `calculadoEm` | string (ISO 8601) | Carimbo de data e hora em que o documento de contagem foi gerado. |

- **Quem atualiza:** **Somente a API** com credenciais administrativas do Firebase Admin SDK. O navegador nunca lê nem escreve diretamente nessa coleção, em conformidade com as Security Rules deny-all ([ADR 0002](adr/0002-tenant-autorizacao.md)). Toda leitura e escrita realizada pela API filtra obrigatoriamente pelo `tenantId` contido na claim do token JWT (RN-01).
- **Quando atualiza:** A contagem é recalculada:
  1. No recálculo **por evento** (RN-13): imediatamente após ações de aprovação de documento, rejeição de documento, vínculo ou desvínculo de trabalhador ao contrato, alteração na matriz de exigências ou inativação de trabalhador.
  2. No **job diário de recálculo** ([ADR 0007](adr/0007-recalculo-idempotente.md)), que reavalia diariamente todos os contratos cadastrados.
- **Idempotência (obrigatório, ADR 0007):** O documento de contagem é **recalculado por inteiro e gravado com operação `set`** a partir do estado final das alocações do contrato; **nunca é incrementado nem decrementado**. Para as mesmas entradas e a mesma data de referência, a saída gerada é idêntica e determinística.
- **Consistência (H23.CA1 — totais batem com a lista):** O fluxo de atualização grava primeiro as alocações afetadas e, **por último**, o documento de contagens, calculado a partir do estado final consolidado dessas alocações. Se o volume de trabalhadores exigir divisão em múltiplos lotes de escrita, o documento de contagens é sempre a última escrita efetuada. Caso ocorra falha no meio do processo, a próxima execução (por ser idempotente) restaura a integridade total dos dados.
- **Como o contrato OpenAPI usa:** A operação `obterConformidadeContrato` (`GET /contratos/{contratoId}/conformidade`) preenche os campos `semaforo`, `totais` e `vencendoEm30Dias` diretamente a partir deste documento; o array `trabalhadores` continua sendo obtido da subcoleção `alocacoes`, com suporte a paginação por cursor. O esquema da resposta no contrato OpenAPI não é alterado.
- **Custo e FinOps:** O painel realiza apenas 1 leitura no Firestore para obter o resumo consolidado, em vez de ler até 500 alocações por acesso, protegendo a cota diária de leituras do plano gratuito Spark. O recálculo gera apenas 1 escrita adicional por contrato.
- **Observação para decisão futura (não é decisão):** Atualmente o parâmetro `limite` possui valor mínimo de 1, de modo que o painel recebe sempre a primeira página de trabalhadores junto ao resumo. Caso a equipe decida no futuro que o painel inicial deva consumir unicamente os totais sem trabalhadores, uma alternativa seria permitir `limite=0`; tal mudança requer alteração contratual no OpenAPI e não faz parte deste PR.

### Índices previstos

| Consulta | Coleção | Campos |
|---|---|---|
| Fila de pendentes (H12) | `documentos` (collection group, filtrada por tenant no caminho) | `statusRevisao` ASC, `enviadoEm` ASC |
| Alertas não lidos (H05) | `alertas` | `lidoEm` ASC, `criadoEm` DESC |
| Trilha por período (H21) | `auditoria` | `acao` ASC, `dataHora` DESC |
| Documentos a recalcular (H15) | `documentos` (collection group) | `statusRevisao` ASC, `validoAte` ASC |

> Consultas de *collection group* atravessam tenants por natureza. **Regra:** só o job interno pode usá-las sem filtro de tenant; toda rota de usuário filtra por `tenantId` da claim. Isso é verificado em revisão de código e em teste de isolamento.

## Ciclo de vida do documento

O documento tem dois eixos independentes: **revisão** (decisão humana) e **validade** (cálculo por data).

```mermaid
stateDiagram-v2
    [*] --> PENDENTE: upload (H02)
    PENDENTE --> APROVADO: gestor aprova (H13)
    PENDENTE --> REJEITADO: gestor rejeita com motivo (H03)
    APROVADO --> [*]
    REJEITADO --> [*]
    note right of APROVADO
        Situação de validade (RN-04),
        recalculada todo dia:
        VALIDO / A_VENCER / VENCIDO
    end note
```

Estados de revisão são **finais**: aprovado ou rejeitado não volta para pendente. Corrigir significa enviar um novo documento (H14), preservando o histórico.

## Regras de negócio

| ID | Regra |
|---|---|
| **RN-01** | Todo dado pertence a exatamente um tenant. O `tenantId` vem sempre da claim do token, nunca da requisição. |
| **RN-02** | Um usuário `terceirizada` só enxerga e altera trabalhadores, documentos e contratos da sua própria `terceirizadaId`. |
| **RN-03** | Rejeição exige motivo com 10 a 500 caracteres após remover espaços nas pontas. |
| **RN-04** | **Faixas de validade.** `dias = validoAte − dataReferência`, em dias corridos, no fuso `America/Sao_Paulo`. Se `dias < 0` → `VENCIDO`; se `0 ≤ dias ≤ 30` → `A_VENCER`; se `dias > 30` → `VALIDO`. O documento vale até o fim do dia `validoAte`. |
| **RN-05** | **Documento que atende uma exigência.** Para cada tipo exigido, considera-se o documento `APROVADO` do trabalhador, daquele tipo, com o maior `validoAte`. Documentos `PENDENTE` e `REJEITADO` não atendem exigência. |
| **RN-06** | **Semáforo trabalhador × contrato.** **Vermelho** se alguma exigência não tem documento que a atenda (RN-05) ou se esse documento está `VENCIDO`. **Amarelo** se todas são atendidas e ao menos uma está `A_VENCER`. **Verde** se todas estão `VALIDO`. Um documento rejeitado não anula outro aprovado e válido do mesmo tipo (questão Q1 em aberto na Visão e Escopo). |
| **RN-07** | **Semáforo do contrato** = pior cor entre os trabalhadores alocados ativos, acompanhado da contagem por cor. Contrato sem trabalhadores mostra "sem trabalhadores", sem cor (materializado em `contagens/{contratoId}`). |
| **RN-08** | **Reuso.** Um documento aprovado atende a exigência do mesmo tipo em qualquer contrato do **mesmo tenant**. O reuso nunca atravessa tenants. |
| **RN-09** | **Data de validade (RF03).** A terceirizada informa `validoAte` (obrigatória) e `dataEmissao` (opcional) ao enviar o documento. Regras: `dataEmissao` não pode estar no futuro; `validoAte` não pode ser anterior a `dataEmissao`. Na aprovação, o gestor confere e pode corrigir `validoAte`. O `validadeMeses` do tipo serve para sugerir a data na tela (`dataEmissao + validadeMeses`) e para sinalizar ao gestor quando a validade informada ultrapassa o padrão do tipo. |
| **RN-10** | **Liberação na portaria.** `LIBERADO` se o trabalhador está ativo e tem semáforo verde ou amarelo em ao menos um contrato **vigente** (`dataInicio ≤ hoje ≤ dataFim`). `BLOQUEADO` se está cadastrado mas não cumpre a regra. `NAO_CADASTRADO` se o CPF não existe no tenant (questão Q2 em aberto). |
| **RN-11** | **Registro de acesso.** `ENTRADA` só pode ser registrada a partir de consulta `LIBERADO` feita há no máximo 10 minutos; cada consulta gera no máximo um registro. |
| **RN-12** | **Alertas.** Um alerta por par (documento, faixa). O ID é `{documentoId}_{tipo}`, o que impede duplicidade em reexecuções. |
| **RN-13** | **Recálculo.** O semáforo é recalculado (a) pelo job diário para todos os documentos aprovados e (b) por evento, após aprovação, rejeição, vínculo e alteração de matriz. O resultado é função pura de (documentos, matriz, data de referência). |
| **RN-14** | **Auditoria.** Toda escrita de negócio e toda consulta de liberação gera um registro de auditoria, que nunca é alterado nem apagado pela aplicação. |

## Exemplo de documento persistido (dados fictícios)

```json
{
  "id": "doc_01J9ZK7Q",
  "trabalhadorId": "trb_01J9ZK2A",
  "tipoDocumentoId": "tdoc_aso",
  "dataEmissao": "2026-03-10",
  "validoAte": "2027-03-10",
  "statusRevisao": "APROVADO",
  "situacaoValidade": "VALIDO",
  "motivoRejeicao": null,
  "arquivo": {
    "caminho": "tenants/tnt_demo/trabalhadores/trb_01J9ZK2A/documentos/doc_01J9ZK7Q/8f3c…e1.pdf",
    "contentType": "application/pdf",
    "tamanhoBytes": 482113,
    "sha256": "9b74c9897bac770ffc029102a200c5de…"
  },
  "enviadoPor": "uid_terceirizada_demo",
  "enviadoEm": "2026-10-20T13:02:11Z",
  "revisadoPor": "uid_gestor_demo",
  "revisadoEm": "2026-10-20T14:40:03Z"
}
```

## Considerações de modelagem

- **Semáforo materializado** em `alocacoes` em vez de calculado a cada leitura: a consulta de portaria lê poucos documentos e cabe na meta de p95 ≤ 800 ms. O custo é manter a materialização correta, garantida pelo recálculo por evento e pelo job diário (RN-13).
- **Datas** de negócio (`dataEmissao`, `validoAte`) são datas sem hora (`YYYY-MM-DD`). Carimbos de tempo são UTC em ISO 8601.
- **IDs** gerados pela aplicação (ex.: ULID) para não expor contagem e permitir ordenação.
- **Exclusão:** no MVP não há exclusão física de documentos e trabalhadores, apenas inativação (`ativo = false`). A política de retenção (questão Q4) definirá a exclusão definitiva.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Modelo de dados por tenant e regras de negócio RN-01 a RN-12, ajustadas à validade informada no envio (C7). |
| 1.1 | 02/11/2026 | @pedrolmelo | Remoção do formato PNG do caminho de arquivos (PO-06) e adição da coleção contagens/{contratoId} para o painel H23 (PO-02). |
