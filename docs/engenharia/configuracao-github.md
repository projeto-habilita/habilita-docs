# Configuração do GitHub (organização, proteção da `main` e merge)

| Campo | Valor |
|---|---|
| Status | Em vigor a partir de 26/10/2026 |
| Responsável | Tech Lead (executa quem é admin do repositório) |
| Relacionados | [Como contribuir](contribuir.md), [ADR 0003](../arquitetura/adr/0003-ci-cd.md), feedback do CP1 (itens 2 e 10) |

Esta página documenta **toda configuração manual** feita no GitHub, para que ninguém dependa de lembrar o que foi clicado (regra do CP2: "nenhuma configuração manual não documentada").

!!! warning "Estado em 03/11/2026 (conferido pelo DevOps em [Como contribuir](contribuir.md), DO-07)"
    A transferência para a organização (seção 2) já foi feita e a `main` dos dois repositórios está protegida por **ruleset** (Settings → Rules → Rulesets → `main`), sem bypass. **Ainda falta**, para cumprir a regra da equipe: **2 aprovações** nos dois repositórios (hoje 1 no código e 0 nos docs), *Dismiss stale approvals*, *Require branches to be up to date*, *Require conversation resolution*, merge **só por squash** e *Automatically delete head branches*. Quem é admin aplica as mesmas opções da seção 4 dentro do ruleset e atualiza os prints em `docs/engenharia/evidencias/`.

## 1. Onde ficam os repositórios

| Repositório | Conteúdo | Check obrigatório |
|---|---|---|
| [projeto-habilita/habilita-docs](https://github.com/projeto-habilita/habilita-docs) | Documentação (MkDocs), ADRs, `openapi.yaml` | `build` |
| [projeto-habilita/Habilita](https://github.com/projeto-habilita/Habilita) | Código (API, web, seed, emuladores, CI) | `ci-ok` |

Site publicado: <https://projeto-habilita.github.io/habilita-docs/>

## 2. Transferir o `habilita-docs` para a organização (feito)

> **Quem pode:** o dono atual (`@osamuelleite`), desde que tenha permissão para **criar repositórios** na organização `projeto-habilita`. Se não tiver, um owner da organização precisa liberar isso primeiro (Organization → Settings → Member privileges → Repository creation).

1. Abra <https://github.com/osamuelleite/habilita-docs> e clique em **Settings** (aba de engrenagem, no topo do repositório).
2. Na página **General**, role até o fim, até a caixa vermelha **Danger Zone**.
3. Clique em **Transfer** (linha *Transfer ownership*).
4. Em *New owner*, escolha **Select one of my organizations** e selecione **`projeto-habilita`**.
5. Digite o nome pedido na caixa de confirmação (`osamuelleite/habilita-docs`) e clique em **I understand, transfer this repository**.
6. Confira o novo endereço: <https://github.com/projeto-habilita/habilita-docs>. O GitHub redireciona o endereço antigo para `git clone`/`git push`, mas **o site do GitHub Pages muda de endereço** e não tem redirecionamento.
7. Em **Settings → Pages**, confira se *Source* continua em **GitHub Actions**. Depois, em **Actions → docs → Run workflow**, rode o workflow na `main` para publicar o site no endereço novo.
8. Em **Settings → Collaborators and teams**, dê acesso de escrita (**Write**) às 6 pessoas da equipe (ou a um time da organização). **Quem está no CODEOWNERS precisa ter Write**, senão a revisão do dono não é exigida.
9. Cada pessoa atualiza o remoto no próprio computador (opcional, porque o redirecionamento funciona):
   `git remote set-url origin https://github.com/projeto-habilita/habilita-docs.git`

## 3. Ajustes de merge (nos dois repositórios)

**Settings → General → Pull Requests:**

- [x] **Allow squash merging** (desmarque *Allow merge commits* e *Allow rebase merging*).
- [x] **Always suggest updating pull request branches**.
- [x] **Automatically delete head branches**.

## 4. Proteger a `main` (nos dois repositórios)

> **Quem pode:** um **admin** do repositório. Antes de começar, abra um PR qualquer (ou use um recente) para o check obrigatório já ter rodado na última semana; só assim ele aparece na busca do passo 6.

1. No repositório, vá em **Settings → Branches**.
2. Clique em **Add classic branch protection rule**.
3. **Branch name pattern:** `main`
4. Marque **Require a pull request before merging** e, dentro dele:
    - [x] **Require approvals** → *Required number of approvals before merging*: **2**
    - [x] **Dismiss stale pull request approvals when new commits are pushed**
    - [x] **Require review from Code Owners**
    - [x] **Require approval of the most recent reviewable push**
5. Marque **Require status checks to pass before merging** e, dentro dele:
    - [x] **Require branches to be up to date before merging**
6. Na busca de checks, digite e selecione o check do repositório:
    - código (`Habilita`): **`ci-ok`** (o job que consolida segredos, higiene, qualidade, testes, build, Dockerfiles e compose);
    - docs (`habilita-docs`): **`build`** (`mkdocs build --strict`).
7. Marque também:
    - [x] **Require conversation resolution before merging**
    - [x] **Require linear history**
    - [x] **Do not allow bypassing the above settings** (vale também para admins)
8. **Deixe desmarcados** *Allow force pushes* e *Allow deletions*.
9. Clique em **Create** e confirme a senha, se o GitHub pedir.
10. **Evidência para o tutor:** tire um print da regra salva (tela inteira, mostrando as opções marcadas) e salve em:
    - `docs/engenharia/evidencias/protecao-main-docs.png`
    - `docs/engenharia/evidencias/protecao-main-codigo.png`

### Como saber se funcionou (2 minutos)

1. Tente `git push origin main` direto: o GitHub precisa **recusar**.
2. Abra um PR: o botão de merge só fica verde com o check obrigatório verde **e** 2 aprovações, sendo pelo menos 1 de um dono do CODEOWNERS.

## 5. Cuidados conhecidos

| Situação | O que acontece | Como evitamos |
|---|---|---|
| Autor é o único dono de uma pasta no CODEOWNERS | O PR nunca recebe a revisão do dono e fica travado | Todo caminho tem **2 donos** nos dois CODEOWNERS |
| Workflow com filtro de caminhos (`paths`) no `pull_request` | PR que não mexe nesses caminhos fica "Expected — Waiting for status" | O `docs.yml` roda `build` em **todo** PR |
| PR do Dependabot | Também precisa de 2 aprovações | Revisão semanal no controle da equipe |
| Pessoa sem acesso Write | A aprovação dela não conta | Passo 2.8 |

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 26/10/2026 | @osamuelleite | Versão inicial: transferência para a org, ajustes de merge e proteção da `main` com 2 aprovações (planilha TL-01, TL-02 e TL-15; feedback do CP1, itens 2 e 10). |
| 1.1 | 03/11/2026 | @osamuelleite | Estado atual da proteção (ruleset) e o que falta ativar; transferência marcada como feita. |
