# Cotas dos planos gratuitos — Guia para o PO

| Campo | Valor |
|---|---|
| Responsável | @pedrolmelo (DevOps) |
| Público-alvo | Product Owner (PO), Tech Lead e equipe de desenvolvimento |
| Data | 31/10/2026 (atualizada em 15/11/2026) |
| Relacionados | [FinOps](index.md), [Registro de gasto (CP2 e CP3)](gasto.md), [ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md) |

## Contexto

No **Plano Custo Zero** ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)), o projeto Habilita não utiliza recursos pagos nem cadastra cartões de crédito. A métrica de acompanhamento de capacidade da equipe deixa de ser o saldo financeiro em dólares e passa a ser o **consumo das cotas dos planos gratuitos**.

Esta página detalha, em linguagem acessível para o PO e gestores do produto, quais são as cotas gratuitas disponíveis em cada ferramenta e **como esses limites afetam a experiência do produto e os testes da equipe**.

## Tabela de cotas gratuitas e impactos no produto

| Serviço | Recurso | Cota do plano gratuito | Quando isso pode atrapalhar o Habilita (Impacto no Produto) |
|---|---|---|---|
| **Firebase Hosting** | Hospedagem do Frontend SPA (`habilita-eb7ca`) | 10 GB de armazenamento estático, 360 MB/dia de transferência | Suficiente para hospedar os arquivos estáticos compilados do React (`https://habilita-eb7ca.web.app` e `https://habilita-eb7ca.firebaseapp.com`) e canais de preview para pull requests (com expiração automática em 7 dias). Manter as imagens otimizadas para não inflar o bundle. |
| **Cloud Firestore (Firebase)** | Banco de Dados NoSQL (`us-central1`) | 1 GiB armazenado; 50.000 leituras/dia, 20.000 escritas/dia e 20.000 exclusões/dia (plano Spark) | **Se a cota diária estourar, o Firebase trava as operações até o dia seguinte (00:00 PST).** Não há cobrança, mas o sistema para. **Ação:** testes automatizados pesados, testes de carga com k6 e cargas volumosas de seed rodam exclusivamente contra os emuladores no Docker Compose local, nunca contra o Firestore de produção. A aplicação implementa paginação obrigatória por cursor (máximo 50 itens) e semáforo materializado para não multiplicar leituras. |
| **Supabase Storage** | Armazenamento de Documentos de SST | **1 GB total armazenado**, 5 GB de download/mês, arquivo individual de até 50 MB | **Capacidade máxima de ~500 documentos de 2 MB (ou ~200 documentos no teto de 5 MB = 5.000.000 bytes).** A aplicação limita o upload a **5.000.000 bytes** ([ADR 0006](../arquitetura/adr/0006-armazenamento-de-documentos.md)) por documento e programa o expurgo em 90 dias após o arquivo deixar de ser necessário. O volume de seed para testes é controlado (no máximo 3 documentos por trabalhador). **Atenção:** o projeto gratuito do Supabase pausa após 1 semana sem uso; a execução periódica do job diário mantém o bucket ativo. |
| **GitHub Actions** | Automação, CI/CD e Recálculo Diário | 2.000 minutos/mês em repositório privado (ilimitado em repositório público) | Execuções excessivas de pipelines ou rotinas agendadas muito frequentes podem consumir os minutos de CI. Os workflows usam cache de dependências (`uv` e `npm`), o deploy de preview ocorre apenas em PRs do próprio repositório e o job de recálculo (`job-diario.yml`) roda uma vez ao dia às 03:00 BRT. |
| **UptimeRobot** | Monitoramento de Disponibilidade | 50 monitores (plano Free 50), intervalo de checagem a cada 5 minutos (300 s), gratuito e sem cartão | A checagem ocorre em intervalos de 5 minutos com envio de alerta por e-mail. No CP3, monitora a disponibilidade do frontend publicado no Firebase Hosting (`https://habilita-eb7ca.web.app`). |
| **Render** *(previsto na ADR 0010; débito técnico no CP3)* | Web Service (API FastAPI) | 750 horas/mês por workspace (dorme após 15 min de inatividade; ~1 min de *cold start*) | **Nota de cota das 750 h/mês:** como um mês tem até 744 horas, a franquia comporta no máximo **1 único serviço** acordado por workspace. **Estado no CP3:** não ativado porque o Render (assim como o Hugging Face Spaces) exigiu cadastro de cartão de crédito/plano pago na criação do serviço. A demonstração oficial ocorre localmente via `docker compose up` ([ADR 0011, D5](../arquitetura/adr/0011-divergencias-caderno.md)). |

---

> **Nota para o PO:** Esta página substitui integralmente a calculadora de preços do Google Cloud (GCP Pricing Calculator), que deixa de se aplicar ao projeto Habilita devido à adoção do Plano Custo Zero.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 31/10/2026 | @pedrolmelo | Criação da página de cotas gratuitas orientada ao PO e à equipe, em substituição à calculadora do GCP. |
| 1.1 | 02/11/2026 | @pedrolmelo | Ajuste na menção ao limite de upload de documentos para 5 MiB (5.242.880 bytes). |
| 1.2 | 15/11/2026 | @pedrolmelo | Atualização para o Checkpoint 3: registro das cotas de Supabase Storage (1 GB e limite de 5.000.000 bytes conforme ADR 0006 v3), Firebase Hosting (`habilita-eb7ca`), UptimeRobot (Free 50) e nota sobre as 750 h do Render e o débito técnico da API pública. |
