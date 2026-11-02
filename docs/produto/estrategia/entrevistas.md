# Entrevistas e Evidências

!!! info "Sobre esta página"
    **Para que serve:** Provar que o problema existe de verdade. O CP1 exige **2 entrevistas** ou evidências secundárias (pesquisas, notícias, normas), rotuladas como tal.

    **O que escrever:** Uma linha por entrevista/evidência. **Nunca** colocar nome real, CPF ou empresa identificável.

    **Prazo:** CP1 · 18/10

| # | Perfil (sem nome) | Data | Tipo | Principais achados |
|---|---|---|---|---|
| E1 | Persona sintética: gerente de SST, indústria média | 18/10/2026 | Entrevista com persona sintética — **hipótese não validada** | Controle em planilha alimentada por e-mail das prestadoras; vencimentos descobertos tarde, às vezes na portaria; o mesmo ASO é pedido de novo a cada contrato. **Custo hoje:** horas da equipe em conferência manual e exposição a autuação, cuja multa cresce com a gravidade e o nº de empregados (ES1, ES5). **Decisor:** gestão de SST/Facilities, com aprovação da diretoria ou dos sócios (ES6). **Disposição a pagar:** referência de mercado de R$ 900/mês por plataforma de gestão de terceiros com 3 usuários (ES5); a persona considera viável algo abaixo disso para empresa média |
| E2 | Persona sintética: porteiro e coordenador de terceirizada | 18/10/2026 | Entrevista com persona sintética — **hipótese não validada** | Portaria confere lista impressa desatualizada e libera na dúvida; sem registro de quem entrou. A terceirizada reenvia os mesmos documentos para cada cliente, por e-mail e ligações, sem padrão (ES6). **Frequência:** a cada novo contrato e a cada vencimento. **Decisor e pagamento:** não decide nem paga; quem contrata é a empresa contratante |
| ES1 | Legislação federal | 14/10/2026 | Evidência secundária — **hipótese não validada** | A contratante responde pelas condições de segurança dos terceirizados em suas dependências (Lei nº 6.019/1974, art. 5º-A, § 3º) |
| ES2 | Normas regulamentadoras | 14/10/2026 | Evidência secundária — **hipótese não validada** | Documentos de SST têm periodicidades diferentes (ex.: NR-10 e NR-35 bienais); um prazo único não serve |
| ES3 | Modelos públicos de checklist de obra | 14/10/2026 | Evidência secundária — **hipótese não validada** | Conferência de ASO, treinamentos e EPI de terceirizados aparece em formulários manuais |
| ES4 | Estudo DIEESE sobre terceirização | 18/10/2026 | Evidência secundária — **hipótese não validada** | Afastamentos por acidente de trabalho típico: 9,6% em atividades tipicamente terceirizadas contra 6,1% nas tipicamente contratantes ([Fundação Perseu Abramo/DIEESE](https://fpabramo.org.br/2017/03/28/notas-fpa-politica-social-407)) |
| ES5 | NR-28 e preço público de concorrente | 18/10/2026 | Evidência secundária — **hipótese não validada** | Multas por descumprimento de NR variam pela gravidade (I1 a I4) e pelo nº de empregados ([CBIC, 27/04/2026](https://cbic.org.br/multas-por-descumprimento-das-normas-regulamentadoras-seguem-criterios-tecnicos-e-variam-conforme-gravidade-da-infracao/)); plataforma de gestão de terceiros custa R$ 900/mês com 3 usuários ([SoftExpert Store](https://store.softexpert.com/pt/gestao_terceiros/p)) |
| ES6 | Avaliações públicas de concorrente | 18/10/2026 | Evidência secundária — **hipótese não validada** | Usuários do Tertium Online relatam burocracia com documentos de terceiros, muitas ligações e e-mails sem padrão, e redução do tempo operacional após automatizar; avaliadores são sócios ou proprietários ([B2B Stack](https://www.b2bstack.com.br/product/tertium-online)) |

Detalhes e fontes de ES1 a ES3 em [Evidências secundárias](evidencias-secundarias.md).

## Como conduzir

- [Roteiro de entrevista](roteiro-entrevista.md): consentimento, 10 perguntas (P1 a P10) ligadas às hipóteses e ao [Ishikawa](ishikawa.md).
- [Modelo de registro](modelo-registro-entrevista.md): copiar para `entrevista-E01.md` e `entrevista-E02.md` nesta pasta (e adicionar ao `nav:` do `mkdocs.yml`).

## Regras

1. **Inventar depoimento é proibido.** Sem entrevista, o problema permanece marcado como "hipótese não validada".
2. Pedir **consentimento** antes de começar e registrar que foi obtido.
3. **Sem dados pessoais**: use códigos (`E01`) e descrições genéricas.
4. Gravações, se houver, ficam **fora** do repositório e são apagadas após a transcrição.

## Perfis-alvo (em ordem de prioridade)

1. Gerente ou técnico(a) de SST de empresa contratante com 3 ou mais contratos de terceirização.
2. Gerente de Facilities ou de manutenção predial.
3. Responsável por documentação em empresa terceirizada.
4. Supervisor(a) de portaria ou controle de acesso.

## Síntese das hipóteses (preencher após as entrevistas)

| Hipótese | Evidência a favor | Evidência contra | Conclusão |
|---|---|---|---|
| HP1 · Documentos de SST de terceirizados são controlados em planilha ou e-mail | | | |
| HP2 · Vencimentos passam despercebidos com frequência | | | |
| HP3 · A portaria não consegue verificar aptidão na hora | | | |
| HP4 · Reenviar o mesmo documento para cada contrato é uma dor real | | | |
| HP5 · Quem decide a compra é a gestão de SST ou Facilities | | | |

!!! note "HP1 a HP5 são hipóteses de pesquisa"
    O prefixo `HP` evita confusão com as histórias de usuário `H01`, `H02`… do [Backlog](../requisitos/historias-backlog.md).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: página e tabela de entrevistas. Do Tech Lead: evidências secundárias ES1 a ES3, perfis-alvo e hipóteses renomeadas para HP1 a HP5 (C10). |
| 1.1 | 18/10/2026 | @Lelafeta | Inclusão dos registros de entrevista com personas sintéticas (E1 e E2) e adição de novas evidências secundárias (ES4 a ES6). |
| 1.2 | 02/11/2026 | @Lelafeta | Correção de quebra de linha após a tabela de entrevistas e evidências. |
