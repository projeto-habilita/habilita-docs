# Ensaio do clone do zero (prova de reprodutibilidade)

| Campo | Valor |
|---|---|
| Status | Em uso a partir do Checkpoint 2 |
| Responsável | Tech Lead (organiza); executa **alguém que não escreveu o código** |
| Quando | Sexta antes de cada gate. CP2: **sex 30/10** (auditoria de configuração manual) e **sáb 31/10** (ensaio completo) |

**Regra de passagem do Caderno:** o gate passa quando a evidência é **reproduzível por outra pessoa da equipe** e está registrada no repositório. "Funciona na minha máquina" não conta.

## Parte A · Auditoria de configuração manual (planilha TL-15)

Antes do ensaio, o Tech Lead confere se **todo passo necessário está escrito**. Qualquer passo que alguém faz "de cabeça" vira uma linha no README do código ou em [Ambiente local](ambiente-local.md).

| Verificação | Onde deve estar escrito | OK? |
|---|---|---|
| Pré-requisitos com versão (Docker, Compose, Git; WSL2 no Windows) | README do código | ☐ |
| Subir tudo sem `.env` (`docker compose up --build`) | README do código | ☐ |
| Rodar o seed e saber **quais usuários existem** (um de cada um dos 4 papéis, nos 2 tenants) e suas senhas de dev | README do código / seed | ☐ |
| Rodar os testes localmente (mesmo comando do CI) | README do código | ☐ |
| Portas usadas (web, API, emuladores, UI dos emuladores) | README do código | ☐ |
| Configurações do GitHub (proteção, merge, Pages) | [Configuração do GitHub](configuracao-github.md) | ☐ |
| Nenhuma variável obrigatória fora do `.env.example` | `.env.example` | ☐ |
| Nada depende de recurso na nuvem (a demo do CP2 é local) | — | ☐ |

## Parte B · Ensaio de 15 minutos (Guia do CP2)

Quem executa: uma pessoa que **não** escreveu a parte testada, em uma pasta nova.

1. `git clone https://github.com/projeto-habilita/Habilita.git` em uma pasta vazia e `docker compose up --build`. **Tudo sobe sem ajuste manual?**
2. Rodar o seed e entrar com **um usuário de cada papel**:
    - J1 (`admin_contratante`): cria um contrato e a matriz de exigências.
    - J2 (`terceirizada`): cadastra um trabalhador e envia um documento.
    - J3 (`analista_sst`): aprova um documento e rejeita outro com motivo.
3. Recarregar a página: os dados continuam lá?
4. Forçar erros: arquivo acima de 5 MB, arquivo PNG ou EXE, campo vazio, API desligada (`docker compose stop api`). A tela mostra **mensagem amigável** e os três estados (carregando, vazio, erro)?
5. Com o usuário da empresa A, abrir um registro da empresa B pelo ID. **O sistema recusa (404)?**
6. Com `analista_sst`, tentar editar a matriz. **O sistema recusa (403)?**
7. Abrir o último PR: o CI rodou os testes e o `ci-ok` ficou verde?
8. Abrir o site dos docs: a [matriz de rastreabilidade](../testes/matriz.md) e o relatório de testes estão lá?

## Registro dos ensaios

| Data | Quem executou | Commit (`main`) | Resultado | Problemas → issue/BUG |
|---|---|---|---|---|
| 30/10/2026 | | | Parte A | |
| 31/10/2026 | | | Parte B | |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/10/2026 | @osamuelleite | Página criada a partir do Guia do CP2 (planilha TL-15 e TL-16). |
