# Cotas dos planos gratuitos — Guia para o PO

| Campo | Valor |
|---|---|
| Responsável | @pedrolmelo (DevOps) |
| Público-alvo | Product Owner (PO), Tech Lead e equipe de desenvolvimento |
| Data | 31/10/2026 |
| Relacionados | [FinOps](index.md), [Registro de gasto (CP2)](gasto.md), [ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md) |

## Contexto

No **Plano Custo Zero** ([ADR 0010](../arquitetura/adr/0010-plano-custo-zero.md)), o projeto Habilita não utiliza recursos pagos nem cadastra cartões de crédito. A métrica de acompanhamento de capacidade da equipe deixa de ser o saldo financeiro em dólares e passa a ser o **consumo das cotas dos planos gratuitos**.

Esta página detalha, em linguagem acessível para o PO e gestores do produto, quais são as cotas gratuitas disponíveis em cada ferramenta e **como esses limites afetam a experiência do produto e os testes da equipe**.

## Tabela de cotas gratuitas e impactos no produto

| Serviço | Recurso | Cota do plano gratuito | Quando isso pode atrapalhar o Habilita (Impacto no Produto) |
|---|---|---|---|
| **Render** | Web Service (API FastAPI) | 750 horas/mês por workspace | **API dorme após 15 minutos sem uso.** O primeiro usuário que acessar a aplicação após esse período aguardará cerca de 1 minuto para a API acordar (cold start). Além disso, as 750 horas são por workspace: se a equipe subir mais de um serviço simultâneo, as horas se esgotam antes de fechar o mês. **Ação:** acordar a API 5 minutos antes de demonstrações (incluindo o Demo Day) e manter tela de carregamento amigável no frontend. |
| **Supabase Storage** | Armazenamento de Documentos de SST | 1 GB total armazenado, 5 GB de download/mês, arquivo individual de até 50 MB | **Capacidade máxima de ~500 documentos de 2 MB (ou ~200 documentos de 5 MB).** A aplicação deve limitar o upload a 5 MB por documento. O volume de seed para testes deve ser controlado (no máximo 3 documentos por trabalhador de teste). **Atenção:** o projeto Supabase pausa após 1 semana sem uso; a equipe mantém um workflow semanal de manutenção para garantir que ele permaneça ativo. |
| **Cloud Firestore (Firebase)** | Banco de Dados NoSQL | 1 GiB armazenado; 50.000 leituras/dia, 20.000 escritas/dia e 20.000 exclusões/dia (plano Spark) | **Se a cota diária estourar, o Firebase trava as operações até o dia seguinte (00:00 PST).** Não há cobrança, mas o sistema para. **Ação:** testes automatizados pesados, testes de carga com k6 e cargas volumosas de seed rodam exclusivamente contra os emuladores no Docker Compose local, nunca contra o Firestore de produção. A aplicação implementa paginação obrigatória por cursor (máximo 50 itens) e semáforo materializado para não multiplicar leituras. |
| **Firebase Hosting** | Hospedagem do Frontend SPA | 10 GB de armazenamento estático, 360 MB/dia de transferência | Suficiente para hospedar os arquivos estáticos compilados do React e canais de preview para pull requests. Manter as imagens da aplicação otimizadas para não inflar o tamanho do bundle. |
| **GitHub Actions** | Automação, CI/CD e Recálculo | 2.000 minutos/mês em repositório privado (ilimitado em repositório público) | Execuções excessivas de pipelines ou rotinas agendadas muito frequentes podem consumir os minutos de CI. O job de recálculo idempotente roda uma vez ao dia com tempo de execução curto. Pipelines de PR aproveitam cache de dependências. |
| **UptimeRobot** | Monitoramento de Disponibilidade | 50 monitores, intervalo de checagem a cada 5 minutos, gratuito e sem cartão | A checagem ocorre em intervalos de 5 minutos apontando para `/readyz`. Uma indisponibilidade pontual pode levar até 5 minutos para ser detectada pelo alerta de e-mail. |

---

> **Nota para o PO:** Esta página substitui integralmente a calculadora de preços do Google Cloud (GCP Pricing Calculator), que deixa de se aplicar ao projeto Habilita devido à adoção do Plano Custo Zero.

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 31/10/2026 | @pedrolmelo | Criação da página de cotas gratuitas orientada ao PO e à equipe, em substituição à calculadora do GCP. |

