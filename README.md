# Habilita — Documentação

> **Documentação em dia. Acesso sob controle.**

Repositório de **documentação** do Habilita: plataforma B2B de conformidade documental de Segurança e Saúde no Trabalho (SST) para empresas que contratam terceirizados. Projeto 01 do ciclo B2B do AI LAB / Makers (04/10 a 05/12/2026).

📖 **Site publicado:** https://projeto-habilita.github.io/habilita-docs/

O código da aplicação fica em um repositório separado, na mesma organização: **[projeto-habilita/Habilita](https://github.com/projeto-habilita/Habilita)** (API FastAPI, frontend React, emuladores e CI).

| Repositório | O que tem | Check obrigatório na `main` |
|---|---|---|
| [projeto-habilita/habilita-docs](https://github.com/projeto-habilita/habilita-docs) | Documentação, ADRs, contrato OpenAPI | `build` (mkdocs `--strict`) |
| [projeto-habilita/Habilita](https://github.com/projeto-habilita/Habilita) | Código da aplicação | `ci-ok` |

Regras de proteção da `main` (2 aprovações + CI verde): [Configuração do GitHub](docs/engenharia/configuracao-github.md).

## Onde encontrar cada evidência

| Gate | Data | Onde |
|---|---|---|
| Checkpoint 1 · Fundação | 18/10 | [Checklist do CP1](docs/engenharia/checkpoint-1.md) |
| Checkpoint 2 · Núcleo funcional | 01/11 | [Matriz](docs/testes/matriz.md) |
| Checkpoint 3 · Nuvem e integração | 15/11 | [Operação](docs/operacao/index.md) |
| Checkpoint 4 · Estabilização | 29/11 | [Relatórios](docs/testes/relatorios/index.md) |
| Demo Day | 05/12 | [Demo Day](docs/demo/index.md) |

## Estrutura

```
habilita-docs/
├── docs/                 # páginas do site (um .md por página)
├── mkdocs.yml            # navegação e configuração do site
├── openapi.yaml          # contrato da API (renderizado em docs/arquitetura/api.md)
├── requirements-docs.txt # dependências fixadas do MkDocs (ADR 0004)
└── .github/
    ├── workflows/docs.yml        # build --strict em todo PR; publica no Pages após merge na main
    └── PULL_REQUEST_TEMPLATE.md
```

## Rodar localmente

```bash
python -m pip install -r requirements-docs.txt
cp openapi.yaml docs/openapi.yaml      # Windows: copy openapi.yaml docs\openapi.yaml
python -m mkdocs serve                 # http://127.0.0.1:8000
```

Antes de abrir um PR, rode `python -m mkdocs build --strict`: link quebrado ou página fora do `nav:` falha o build (e o CI).

Guia completo em [Primeiros passos](docs/engenharia/primeiros-passos.md) e [Como contribuir](docs/engenharia/contribuir.md).

## Publicação

O workflow [`docs.yml`](.github/workflows/docs.yml) roda `mkdocs build --strict` em todo PR e publica no GitHub Pages a cada push na `main` (Settings → Pages → Source = "GitHub Actions"). Páginas com `!!! todo "CPx"` indicam o gate em que precisam estar preenchidas.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Apresentação do repositório e instruções para rodar a documentação. |
