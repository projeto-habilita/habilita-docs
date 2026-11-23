## O que muda

<!-- Uma ou duas frases. Por que esta mudança existe? -->

**História:** H
**Tipo:** feat / fix / docs / test / refactor / chore

## Critérios de aceite atendidos

<!-- Liste os critérios cobertos e o teste de cada um -->

| Critério | Teste |
|---|---|
| HNN.CA1 | |

## Como testar

<!-- Passos para o revisor reproduzir localmente -->

1. `docker compose up`
2.

## Evidências

<!-- Prints dos estados de tela (carregando, vazio, erro, sucesso), saída de teste, link do CI -->

## Checklist do autor (Definition of Done)

- [ ] Teste novo ligado a um critério de aceite
- [ ] Teste de autorização para rotas novas ou alteradas (papel errado, tenant errado, sem token)
- [ ] Telas com estados de carregando, vazio, erro e sucesso
- [ ] Nenhum segredo no código; nenhum dado pessoal em log
- [ ] `openapi.yaml` atualizado (se o contrato mudou)
- [ ] Documentação atualizada no mesmo PR (página em `docs/`, ADR, modelo de dados) e `mkdocs build --strict` sem erro
- [ ] Nenhum recurso novo no GCP sem ADR ou aprovação do Tech Lead
- [ ] Valor diferente do Caderno ou da ficha? Registrado na [ADR 0011](docs/arquitetura/adr/0011-divergencias-caderno.md) no mesmo PR
- [ ] Dependência nova justificada abaixo (nome conferido no registro oficial)

## Dependências novas

<!-- Nome, motivo, alternativa considerada. Apague se não houver. -->

## Checklist do revisor

- [ ] Domínio sem dependência de infraestrutura (ADR 0005)
- [ ] Tenant e terceirizada vindos da claim, nunca da requisição (ADR 0002)
- [ ] Entrada e saída com modelos Pydantic (`extra="forbid"`, limites de tamanho, `response_model`)
- [ ] Rotas que usam SDK síncrono (`firebase-admin`) declaradas com `def`, não `async def`
- [ ] Rate limit aplicado em rota nova de escrita ou de consulta sensível
- [ ] Testes cobrem bordas, não só o caminho feliz
- [ ] Nenhuma divergência do Caderno sem linha na ADR 0011
- [ ] São 2 aprovações, incluindo um dono do CODEOWNERS; o autor não conta
