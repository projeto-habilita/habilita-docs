# ADR 0004 — Documentação do projeto com MkDocs + Material

| Campo | Valor |
|---|---|
| Status | **Aceito** — aprovado em PR (Tech Lead + 1 integrante) na Sprint 4 (feedback do Checkpoint 1, item 7) |
| Data | 14/10/2026 |
| Decisores | PO + equipe Habilita |
| Consultados | Tutores AI LAB / MAKERS |
| Relacionados | ADR 0001 (stack), ADR 0003 (CI/CD), Definition of Done |

## 1. Contexto

O ciclo B2B exige que **toda evidência esteja no repositório** e seja reproduzível por outra pessoa da equipe. A documentação não é opcional: ela é cobrada em todos os gates.

| Gate | O que precisa estar documentado |
|---|---|
| CP1 · 18/10 | Visão do produto, Ishikawa, backlog MoSCoW, entrevistas, 4 ADRs, OpenAPI v0.1, DoD, estratégia de testes |
| CP2 · 01/11 | Matriz requisito → aceite → teste → resultado; primeiro relatório em `/docs` |
| CP3 · 15/11 | Operação (healthz/readyz, logs, alertas), segurança de infraestrutura |
| CP4 · 29/11 | JUnit e cobertura em `/docs/testes`, rollback ensaiado, relatório de custos |
| Demo · 05/12 | Links para produção, CI, testes, cobertura, Lighthouse, custos, rollback e documentação |

Além disso, a DoD define que uma história só está concluída com **documentação atualizada e evidência vinculada ao relatório**. Portanto a documentação precisa:

1. Viver no mesmo repositório do código e mudar no mesmo PR (docs-as-code).
2. Ser revisada no PR e validada pelo CI, como o código.
3. Renderizar OpenAPI, diagramas e relatórios de teste sem trabalho manual.
4. Custar zero do teto de US$ 50.
5. Ser simples para uma equipe de 9 semanas, sem curva de aprendizado alta.

## 2. Opções consideradas

| Critério | **MkDocs + Material** | Docusaurus | Markdown puro no GitHub | Wiki / Notion / Google Docs |
|---|---|---|---|---|
| Docs-as-code (mesmo PR) | Sim | Sim | Sim | **Não** |
| Validação no CI (links quebrados, nav) | Sim (`--strict`) | Sim | Não | Não |
| Busca e navegação | Sim | Sim | Fraca | Sim |
| Mermaid (arquitetura, estados) | Nativo no tema | Plugin | Sim (GitHub) | Parcial |
| Renderizar OpenAPI | Plugin (Swagger UI) | Plugin | Não | Não |
| Esforço de setup | Baixo (1 YAML) | Médio (Node/React) | Nenhum | Nenhum |
| Custo | US$ 0 | US$ 0 | US$ 0 | US$ 0 / licença |
| Rastreabilidade de evidência | Alta | Alta | Média | **Baixa** |
| Saúde do projeto em out/2026 | **Atenção** (ver §5) | Ativo | — | — |

Wiki/Notion/Google Docs ficam fora porque quebram a regra de evidência no repositório e não passam por PR. Markdown puro é a alternativa mínima, mas não valida links nem organiza navegação para os tutores no Demo Day. Docusaurus é sólido, porém adiciona uma segunda toolchain Node só para docs e mais configuração do que o ganho justifica neste ciclo.

## 3. Decisão

Adotar **MkDocs 1.6.1 + Material for MkDocs 9.7.x**, com versões fixadas, fonte em `/docs` e configuração em `mkdocs.yml` na raiz do repositório.

Regras associadas:

- **Docs-as-code:** a pasta `/docs` é a fonte única; nada de evidência apenas em chat, e-mail ou drive.
- **Mesmo PR:** mudança de comportamento, contrato ou infraestrutura atualiza a página correspondente no mesmo PR (item da DoD e do template de PR).
- **CI bloqueante:** `mkdocs build --strict` roda em todo PR; link quebrado ou página fora da navegação **falha o build**.
- **ADRs:** formato desta página, numerados (`NNNN-titulo.md`), imutáveis depois de aceitos — mudanças geram ADR novo que *substitui* o anterior.
- **OpenAPI:** o arquivo `openapi.yaml` é a fonte; a página `arquitetura/api.md` apenas o renderiza (sem cópia manual).
- **Relatórios de teste:** JUnit, cobertura e Lighthouse gerados no CI são publicados em `docs/testes/relatorios/` como artefatos versionados por gate.
- **Publicação:** GitHub Pages (custo zero, fora do orçamento GCP). Preview opcional via artefato do CI.
- **Sem dados sensíveis:** nenhum dado real de trabalhador, CPF ou documento entra na documentação; usar apenas a massa de seed.

## 4. Consequências

**Positivas**

- Evidência de cada gate fica navegável num único site, com histórico Git.
- Tutores e avaliadores encontram tudo pelo menu, sem pedir links.
- Documentação desatualizada vira falha de CI, não descoberta na véspera do gate.
- Zero impacto no teto de US$ 50.

**Negativas / custos**

- Python como dependência extra apenas para docs (isolada em `requirements-docs.txt`).
- Disciplina de atualizar docs no PR aumenta levemente o tempo de cada história.

## 5. Riscos e mitigação

O ecossistema MkDocs está em transição e isso precisa estar explícito:

| Risco | Situação | Mitigação |
|---|---|---|
| MkDocs 1.x sem novas versões | A versão 1.6.1 não recebe atualizações há mais de 18 meses; o MkDocs 2.0 em desenvolvimento **não** constrói projetos 1.x | Fixar `mkdocs==1.6.1`; não atualizar para 2.0 durante o ciclo |
| Fim de vida do Material | O Material for MkDocs tem fim de vida previsto para **05/11/2026**, no meio do ciclo (entre CP2 e CP3) | Fixar `mkdocs-material==9.7.*`; o site continua funcionando após o EOL — apenas não recebe correções |
| Dependência sem patch de segurança | Gerador estático, roda só no CI e produz HTML estático — superfície de ataque baixa | Varredura de dependências do CP4 inclui `requirements-docs.txt`; exceção registrada se houver alerta |
| Necessidade de migrar | Baixa dentro de 9 semanas | Caminho de saída: **Zensical** (dos mesmos autores) lê `mkdocs.yml` existente. Manter a config compatível: só extensões Markdown padrão e plugins mínimos |

**Gatilho de revisão:** se algum alerta crítico/alto de segurança surgir nas dependências de docs, ou se o build quebrar sem correção, abrir ADR substituta avaliando Zensical ou Markdown puro.

## 6. Validação da decisão

- [ ] `mkdocs build --strict` verde no CI até 18/10.
- [ ] Os 4 ADRs do CP1 publicados no site.
- [ ] Item "documentação atualizada" presente no template de PR e na DoD.
- [ ] Site publicado e link no README.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial da PO: documentação com MkDocs + Material, versões fixadas. Verificada pelo Tech Lead na conciliação. |
| 1.1 | 26/10/2026 | @osamuelleite | Status Aceito (feedback do CP1, item 7). URLs do site e do repositório atualizadas para a organização `projeto-habilita`. |
