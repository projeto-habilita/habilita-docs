# ADR 0010 — Plano Custo Zero (substituição de serviços GCP por equivalentes gratuitos)

| Campo | Valor |
|---|---|
| Status | **Aceito** no merge do PR #16 do Tech Lead: plano definido pelo monitor; exceções ao Caderno pendentes de registro da aprovação do professor (seção 6) |
| Data | 31/10/2026 · revisado em 03/11/2026 |
| Decisores | Tech Lead, DevOps |
| Consultados | PO e tutores AI Lab / Makers |
| Relacionados | [ADR 0001](0001-stack.md), [ADR 0003](0003-ci-cd.md), [ADR 0006](0006-armazenamento-de-documentos.md), [ADR 0007](0007-recalculo-idempotente.md), [FinOps](../../finops/index.md), Registro de gasto (gasto.md) e Cotas gratuitas (cotas-gratuitas.md) |
| Origem | Documento do monitor **"Plano Custo Zero — sem o voucher do GCP"** (@Pedro Henrique Pereira Santos, 14/10/2026), repassado à equipe pelo Tech Lead em 03/11/2026 |
| Aprovação do professor | `[pendente — registrar data, quem aprovou e onde (e-mail, mensagem ou ata)]` |

> **Atenção:** Esta ADR **substitui parcialmente** (no que diz respeito a hospedagem, armazenamento, agendamento e custo) as seguintes decisões: [ADR 0001](0001-stack.md) (stack principal), [ADR 0003](0003-ci-cd.md) (CI/CD e deploy), [ADR 0006](0006-armazenamento-de-documentos.md) (armazenamento de documentos) e [ADR 0007](0007-recalculo-idempotente.md) (recálculo idempotente).

## 1. Contexto

No planejamento original do ciclo B2B (registrado nas ADRs 0001 e 0003), a infraestrutura em nuvem dependia do Google Cloud Platform (GCP) com os serviços Cloud Run, Cloud Storage, Cloud Scheduler, Secret Manager e Artifact Registry, sob um teto de gastos de US$ 50 monitorado por alertas de orçamento (50%, 80% e 100%).

Entretanto, durante a execução inicial:
1. **Instrução direta dos tutores:** os tutores do AI Lab / Makers orientaram a equipe a **não usar o GCP**.
2. **Impossibilidade de faturamento:** a equipe não conseguiu ativar conta de faturamento no GCP sem cadastrar cartão de crédito pessoal.
3. **Ausência de créditos institucionais:** o voucher educacional do GCP não chegou.
4. **Regra de ouro contra risco financeiro:** a avaliação gratuita do GCP e a ativação de projetos vinculados exigem cartão de crédito. O time estabeleceu como princípio inegociável que nenhum integrante da equipe cadastrará cartão pessoal em qualquer plataforma de nuvem.
5. **Requisitos do Checkpoint 2 (prazo 01/11/2026):** o gate exige comprovação de (a) gasto financeiro registrado como US$ 0,00, (b) decisão de arquitetura devidamente formalizada em ADR e (c) registro detalhado em `docs/finops`.

Dessa forma, fez-se necessária a definição de uma estratégia de infraestrutura com custo estritamente zero, substituindo peças pagas por plataformas com planos gratuitos sem exigência de cartão.

## 2. Opções consideradas

| Critério | Opção A: Voucher / GCP original | **Opção B: Plano Custo Zero** | Opção C: Adiar a decisão |
|---|---|---|---|
| **Descrição** | Aguardar liberação de voucher para ativar o GCP original | Substituir Cloud Run, Storage e Scheduler por Render, Supabase e GitHub Actions | Manter a stack original no papel e postergar migração até o CP3 |
| **Custo financeiro** | Risco de cobrança em cartão pessoal (se ativado sem voucher) | **US$ 0,00 garantido** (nenhum cartão cadastrado em nenhum provedor) | Incerto |
| **Aderência ao Caderno** | 100% aderente à stack original (sem exceções) | Requer 6 exceções formais ao Caderno | Aderente apenas teoricamente |
| **Viabilidade imediata** | Bloqueada (sem voucher e sem cartão) | **Imediata** (plataformas gratuitas sem cartão ativáveis no ato) | Falsa impressão de avanço |
| **Impacto no CP2** | Reprovação no gate (sem gasto US$ 0 comprovável) | **Aprovado no gate** com ADR e evidências de plano gratuito | Reprovação no gate |
| **Prós** | Sem exceções de arquitetura; rewrite de mesma origem nativo | Risco financeiro nulo; desbloqueio do time; cortes automáticos por cotas | Nenhum esforço imediato de documentação |
| **Contras** | Bloqueio das entregas; risco financeiro para os alunos | Cold start no Render (~1 min); limites rígidos de cotas gratuitas | Inadimplência com os prazos do AI Lab |

A **Opção B (Plano Custo Zero)** foi a única opção viável e segura para a equipe e para o cumprimento dos marcos do projeto.

## 3. Decisão

Adotar integralmente o **Plano Custo Zero** para a hospedagem, armazenamento, agendamento e observabilidade da plataforma Habilita, respeitando as seguintes diretrizes:

### 3.1 Regras de ouro do time

1. **Nunca cadastrar cartão** em nenhum serviço (Google Cloud, Firebase, Render, Supabase, GitHub).
2. **Nunca clicar em Upgrade/Blaze** nem vincular conta de faturamento ao projeto Firebase.
3. **Testes pesados só no emulador local** (k6, testes automatizados e seed grande rodam no Docker Compose, nunca contra o Firestore real).
4. **Um projeto Firebase por equipe**, criado por um integrante e compartilhado com os demais como membros.
5. **Conta de serviço com mínimo de permissão**; a chave fica apenas nos secrets do GitHub e nas variáveis do Render, nunca no repositório (gitleaks no CI continua obrigatório).
6. **Registrar em `/docs/finops` a print de cada serviço** mostrando plano gratuito e nenhuma forma de pagamento (substitui os alertas de orçamento como prova ao tutor).
7. **Em caso de dúvida, não ativar**; qualquer serviço novo precisa de ADR e aprovação do Tech Lead.

### 3.2 Mapa de substituição de componentes

| Peça original (GCP) | Substituto gratuito | Limite do plano gratuito |
|---|---|---|
| **Cloud Run (API)** | **Render** (web service gratuito) | 750 horas/mês por workspace; dorme após 15 min de inatividade e leva cerca de 1 min para acordar (cold start). |
| **Cloud Storage** | **Supabase Storage** (plano gratuito) | 1 GB total armazenado, 5 GB de download/mês, arquivo de até 50 MB; o projeto pausa após 1 semana sem uso. Limite da aplicação fixado em 5 MB = **5.000.000 bytes** (ADR 0006). Expurgo de 90 dias feito pelo job diário, contado de quando o arquivo deixa de ser necessário (ADR 0006 §3.4; ADR 0011, I2). |
| **Cloud Scheduler** | **GitHub Actions** (`schedule` cron) | 2.000 min/mês em repositório privado, ilimitado em público; workflow chama rota protegida da API. |
| **Secret Manager** | **GitHub Secrets** e **variáveis de ambiente do Render** | Variáveis protegidas e mascaradas nativas das ferramentas. |
| **Artifact Registry e retenção de 5 imagens** | **Build nativo do Render** a partir do Git | Rollback limitado aos 2 deploys anteriores; não há registro de imagens para limpar. |
| **Workload Identity Federation** | **Chave da conta de serviço em variável protegida** | Variáveis secretas no GitHub Actions e no Render; gitleaks ativo. (Exceção ao Caderno). |
| **Cloud Logging e Cloud Monitoring** | **Logs do Render** e **UptimeRobot** | UptimeRobot gratuito (50 monitores, checagem a cada 5 min sem cartão); alerta "/readyz fora do ar" por e-mail. Health check do Render em `/healthz`. |
| **Alertas de orçamento 50/80/100%** | **Prova de plano gratuito sem faturamento** | Prints do plano Spark / Free Tier em `docs/finops` comprovando ausência de cartão e gasto US$ 0. |
| **Preview de backend em PR** | **Apenas testes no PR** | Backend não tem preview em nuvem; roda suíte de testes no PR. Frontend continua com preview no Firebase Hosting. |

### 3.3 O que continua inalterado

- **Firebase Hosting, Firebase Authentication e Cloud Firestore** permanecem no plano **Spark** (gratuito).
- Cotas do Firestore no plano Spark: 1 GiB armazenado; 50.000 leituras, 20.000 escritas e 20.000 exclusões por dia. No plano Spark não há como ser cobrado: quando a cota diária é atingida, o Firebase interrompe as operações e retorna erro até a virada do dia (meia-noite PST).
- **Estilo arquitetural:** Monólito modular em camadas, Firebase Auth com custom claims injetadas, autorização centralizada na API FastAPI e Security Rules do Firestore e Storage com regra padrão `deny-all`.

### 3.4 Decisão sobre o ambiente local

No ambiente local, **o Storage do Supabase roda como serviço do próprio Docker Compose** (sem dependência da CLI externa do Supabase e sem `supabase start`), enquanto o Firebase Emulator Suite continua em execução exclusiva para Auth e Firestore.

### 3.5 Mudanças nos gates (versão Custo Zero)

- **Checkpoint 1 (Fundação):** Mantido, com emulador local de Storage substituído pelo serviço Supabase Storage dentro do Compose.
- **Checkpoint 2 (Núcleo funcional — 01/11/2026):** Teto de gasto ajustado de ≤ US$ 5 para **gasto = US$ 0,00**, com decisão formalizada nesta ADR 0010 e registro de evidências em `docs/finops`.
- **Checkpoint 3 (Nuvem e integração — 15/11/2026):** De Hosting + Cloud Run com rewrite `/api/**` para **Hosting + Render com CORS restrito ao domínio da equipe**; deploy da API pelo Render após CI verde no merge da `main`; health check `/healthz` (Render) e `/readyz` (UptimeRobot); gasto mantido em US$ 0,00.
- **Checkpoint 4 (Estabilização — 29/11/2026):** Rollback operacionalizado pelo painel do Render (últimos 2 deploys) e pelo histórico do Firebase Hosting; auditoria de cotas consumidas (Firestore, Render, Supabase, Actions) no lugar do congelamento financeiro em US$ 40.
- **Demo Day (05/12/2026):** Como o Render leva cerca de 1 minuto para acordar do estado dormente, a equipe adotará o procedimento de **acordar a API 5 minutos antes da apresentação** (aquecimento via ping HTTP), além de manter o estado de carregamento amigável no frontend.

## 4. Consequências

**Positivas:**
- Garantia matemática de risco financeiro nulo para todos os membros da equipe e para a instituição.
- Atendimento pleno às diretrizes pedagógicas dos tutores do AI Lab / Makers.
- Desbloqueio imediato da infraestrutura de integração contínua e publicação para os gates.
- O plano Spark corta o tráfego quando o limite é atingido, impedindo faturas surpresa.

**Negativas / Restrições operacionais:**
- *Cold start* no Render: após 15 minutos sem receber requisições, o contêiner dorme e demora ~60 segundos para responder à primeira chamada.
- Cota de 750 horas/mês no Render: permite manter apenas 1 serviço ativo por workspace (um mês tem no máximo 744 horas). Não é possível subir múltiplos ambientes simultâneos.
- Pausa no Supabase: o projeto gratuito pausa após 7 dias de inatividade, exigindo workflow de manutenção.
- Perda do rewrite de mesma origem: a separação entre Firebase Hosting e Render impõe a configuração e manutenção de CORS rigoroso.
- Rollback restrito: o Render limita o rollback imediato aos 2 últimos deploys realizados.
- Exceções ao Caderno que demandam aprovação formal do professor.

## 5. Riscos e mitigação

| Risco | Impacto | Mitigação planejada |
|---|---|---|
| **Cota diária do Firestore estourar** | Alto (API indisponível até o dia seguinte) | Testes automatizados, k6 e cargas de seed rodam 100% no Compose local com emuladores; paginação obrigatória por cursor (máx 50); leitura apenas de documentos aprovados; semáforo materializado; proibido rodar testes de carga na véspera do Demo Day. |
| **Render dormindo no Demo Day** | Médio (atraso de 1 min na apresentação) | Procedimento de aquecimento: chamar a rota `/healthz` 5 minutos antes da banca avaliadora; tela de carregamento visual no frontend informando status. |
| **750 horas do Render acabarem** | Alto (API suspensa até o mês seguinte) | Manter rigorosamente um único web service por workspace; suspender instâncias experimentais; monitorar consumo no painel. |
| **Supabase pausar por inatividade** | Médio (falha de upload/download de arquivos) | Workflow semanal no GitHub Actions que executa operação mínima no bucket, mantendo o projeto ativo. |
| **Chave da conta de serviço vazar** | Crítico (comprometimento do Firebase) | Conta de serviço com privilégios mínimos necessários; credencial armazenada apenas em segredos protegidos (GitHub Secrets e variáveis de ambiente do Render); verificação obrigatória de gitleaks no CI; procedimento documentado de exclusão e rotação imediata no console em caso de suspeita. |
| **Exceções ao Caderno não aceitas pela banca** | Alto (penalização em nota no gate) | Submeter o plano para aprovação formal do professor antes do Checkpoint 2 (01/11/2026); registrar todas as 6 exceções com justificativas e mitigações nesta ADR. |

## 6. Exceções ao Caderno

Para cumprir a diretriz de não utilizar o GCP sem cartão, a equipe solicita formalmente a aprovação das seguintes 6 exceções aos requisitos do Caderno:

1. **API hospedada no Render em vez do Cloud Run:**
   - *Justificativa:* O Cloud Run exige ativação de faturamento no GCP com cartão de crédito. O Render oferece hospedagem de contêineres Docker com camada gratuita sem exigir cartão.
   - *Mitigação:* Contêiner empacotado da mesma forma que rodaria no Cloud Run, com endpoint de verificação `/healthz`.
2. **CORS restrito no lugar de rewrite de mesma origem no Firebase Hosting:**
   - *Justificativa:* O rewrite de `/api/**` no Hosting só direciona tráfego para Cloud Run ou Cloud Functions dentro do mesmo projeto GCP. Com a API no Render, o frontend e o backend residem em domínios distintos.
   - *Mitigação:* Configuração explícita de CORS no FastAPI com allowlist restrita ao domínio oficial do Firebase Hosting e a `localhost` em desenvolvimento, bloqueando qualquer outra origem.
3. **Chave de conta de serviço em variável protegida no lugar de Workload Identity Federation (WIF):**
   - *Justificativa:* A federação de identidade do GCP exige configuração vinculada a projeto com faturamento ativo.
   - *Mitigação:* A conta de serviço do Firebase Admin possui papéis estritamente limitados ao escopo do app; o JSON da chave é injetado como segredo criptografado nas GitHub Actions e no Render; gitleaks bloqueia commits com segredos; em caso de suspeita, revoga-se a chave no console do Firebase e gera-se uma nova.
4. **Supabase Storage no lugar do Cloud Storage:**
   - *Justificativa:* O Cloud Storage do Firebase exige plano Blaze ou conta de faturamento GCP. O Supabase fornece 1 GB gratuito com bucket privado sem necessidade de cartão.
   - *Mitigação:* Upload multipart intermediado e validado pela API com limite de 5 MB por arquivo; URLs assinadas temporárias para download; acesso direto ao bucket bloqueado.
5. **GitHub Actions no lugar do Cloud Scheduler:**
   - *Justificativa:* O Cloud Scheduler requer faturamento ativado no GCP.
   - *Mitigação:* Workflow do GitHub Actions agendado via `schedule` (cron) diário às 03:00, que invoca a rota protegida `/internal/jobs/recalculo-vencimentos` enviando token de autorização.
6. **UptimeRobot no lugar do Cloud Monitoring / Cloud Logging:**
   - *Justificativa:* O monitoramento de alertas e métricas do Cloud Monitoring exige ambiente GCP integrado.
   - *Mitigação:* Logs estruturados da aplicação disponíveis no painel do Render; monitoramento externo gratuito pelo UptimeRobot realizando pings a cada 5 minutos no endpoint `/readyz` com alertas enviados para o e-mail da equipe.

## 7. Reversibilidade

Caso a equipe receba um voucher institucional do Google Cloud Platform durante o ciclo, a reversibilidade para a stack original é simples e foi preservada:
- **Camadas estáveis:** Firebase Authentication, Cloud Firestore e Firebase Hosting já residem no ecossistema Google/Firebase e não sofrem qualquer migração.
- **API:** O código da API permanece desacoplado e empacotado em contêiner Docker padrão (FastAPI). Para retornar ao Cloud Run, basta redirecionar o workflow de CI/CD para compilar a imagem e implantá-la no Cloud Run, reativando o rewrite `/api/**` no `firebase.json` e o WIF.
- **Storage:** O módulo de armazenamento da API utiliza o padrão de portas e adaptadores. Para retornar ao Cloud Storage, substitui-se o adaptador Supabase pelo adaptador nativo do Google Cloud Storage.
- **Agendador:** O acionamento do job via Cloud Scheduler pode ser reativado com a mesma rota `/internal/jobs/recalculo-vencimentos`.

## 8. Validação da decisão

- [x] Print do console do Firebase evidenciando plano Spark e ausência de forma de pagamento registrada em `docs/finops/evidencias/spark-plano.png`.
- [x] Registro de gasto em `docs/finops/gasto.md` preenchido e declarado como US$ 0,00 em 31/10/2026.
- [ ] Adaptação do Docker Compose local com Supabase Storage validada (pendente de merge na `main` da aplicação).
- [ ] Configuração de CORS restrito ao domínio de produção implementada e validada antes do CP3.
- [ ] Monitoramento via UptimeRobot apontando para `/readyz` e health check do Render em `/healthz` configurados no CP3.
- [ ] Auditoria de cotas gratuitas consumidas realizada no CP4 em substituição ao limite de US$ 40.
- [ ] Aprovação do professor para as 6 exceções da seção 6 registrada no cabeçalho desta ADR.
- [ ] Workflow do job diário (`job-diario.yml`) chamando a rota protegida com `JOB_TOKEN` (ADR 0007) no CP3.

## 9. Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 31/10/2026 | @pedrolmelo | Versão inicial: proposta do Plano Custo Zero com substituição de Cloud Run, Cloud Storage, Cloud Scheduler por serviços gratuitos sem cartão. |
| 1.1 | 03/11/2026 | @osamuelleite | Status Aceito com a referência ao documento do monitor; limite de 5.000.000 bytes e expurgo pelo job diário (ADR 0006 v3, ADR 0007 v1.3); interpretações registradas na ADR 0011. |
| 1.2 | 05/11/2026 | @osamuelleite | Status aponta para o PR #16, sem data fixa de merge. |
