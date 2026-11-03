# ADRs

Decisões de arquitetura numeradas e imutáveis após aceitas. Mudanças geram uma ADR nova que substitui a anterior.

| ADR | Título | Status | Gate |
|---|---|---|---|
| [0001](0001-stack.md) | Stack principal | Proposto — Python + FastAPI decidido em 14/10 | CP1 |
| [0002](0002-tenant-autorizacao.md) | Multi-tenant e autorização | Proposto | CP1 |
| [0003](0003-ci-cd.md) | CI/CD e deploy | Proposto | CP1 |
| [0004](0004-documentacao-mkdocs.md) | Documentação com MkDocs | Proposto | CP1 |
| [0005](0005-estilo-arquitetural.md) | Monólito modular em camadas | Proposto | CP2 |
| [0006](0006-armazenamento-de-documentos.md) | Armazenamento de documentos (Storage privado, upload pela API, URL assinada) | Proposto | CP2 |
| [0007](0007-recalculo-idempotente.md) | Recálculo diário idempotente de vencimentos | Proposto | CP2 |
| [0008](0008-consulta-liberacao-cpf.md) | Consulta de liberação por CPF com HMAC e POST | Proposto | CP2 |
| [0009](0009-auditoria-append-only.md) | Trilha de auditoria append-only | Proposto | CP2 |
| [0010](0010-plano-custo-zero.md) | Plano Custo Zero (substituição de serviços GCP por equivalentes gratuitos) | Proposto | CP2 |

**Cobertura do Checkpoint 1:** as 4 ADRs gerais (0001 a 0004). Os ADRs específicos do Habilita pedidos pelo Guia do PO — **multi-tenancy** e **RBAC** — estão registrados juntos na ADR 0002. As ADRs 0005 a 0009 detalham os desafios de engenharia da ficha (upload seguro, recálculo idempotente, portaria) e podem ser aprovadas até o CP2.

## Ciclo de vida

```
Proposto ──revisão em PR──▶ Aceito ──nova decisão──▶ Substituído pela ADR NNNN
                       └──▶ Rejeitado
```

- Uma ADR passa a **Aceito** quando aprovada em PR pelo Tech Lead e por mais um integrante.
- ADR aceita **não é editada** para mudar a decisão: cria-se uma nova, e a antiga recebe "Substituído pela ADR NNNN".
- Escreva uma ADR quando a decisão afeta mais de um módulo, é cara de reverter, gera custo no GCP (regra da DoD) ou foi discutida mais de uma vez.
- ADR nova: arquivo `NNNN-titulo.md` nesta pasta **e** linha no `nav:` do `mkdocs.yml`.

## Modelo

Contexto · Opções consideradas · Decisão · Consequências · Riscos e mitigação · Validação — exemplo completo na [ADR 0004](0004-documentacao-mkdocs.md).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Índice dos ADRs 0001 a 0004 (temas da PO) e 0005 a 0009 (propostos pelo Tech Lead). |
| 1.1 | 31/10/2026 | @pedrolmelo | Inclusão da ADR 0010 (Plano Custo Zero). |
