# Definition of Done

"Funciona na minha máquina" não é pronto. Estas listas são a mesma régua para toda a equipe.

## Definition of Ready — quando uma história pode entrar na sprint

- [ ] Escrita no formato *Como [persona], quero [ação], para [benefício]*, com ID `HNN` e RF de origem.
- [ ] Critérios de aceite mensuráveis (o QA consegue escrever o teste lendo a frase).
- [ ] Prioridade MoSCoW definida pela PO.
- [ ] Rotas e esquemas afetados identificados no `openapi.yaml`.
- [ ] Dependências e bloqueios conhecidos (outra história, ADR pendente, dado de seed).
- [ ] Cabe em até 3 dias de trabalho de uma pessoa; se não couber, quebrar.

## Definition of Done — quando uma história está concluída

Uma história só está concluída quando tem:

- [ ] Implementação
- [ ] Teste — **ligado a um critério de aceite** (ID `HNN.CAn` no nome do teste ou em comentário)
- [ ] Critério de aceite validado
- [ ] PR revisado — **por alguém diferente do autor**
- [ ] CI verde (incluindo `mkdocs build --strict`)
- [ ] Documentação atualizada no mesmo PR (incluindo `openapi.yaml` quando o contrato muda)
- [ ] Evidência vinculada na [matriz](../testes/matriz.md)

E também, conforme o Caderno B2B (seção 1.6):

- [ ] Nenhum segredo (senha, chave, token) no código ou no histórico.
- [ ] Telas com estados de carregando, vazio, erro e sucesso.
- [ ] Logs sem dados pessoais (sem CPF, nome, e-mail, conteúdo de documento, token).
- [ ] Nenhum recurso novo em nuvem sem ADR ou aprovação do Tech Lead.
- [ ] Teste de autorização para toda rota nova ou alterada (sem token, papel errado, tenant errado).
- [ ] Entrada e saída tipadas com Pydantic (`extra="forbid"`), mypy e `tsc` sem erros ([Qualidade](qualidade.md)).

## Definition of Done do gate

Um gate só passa quando a evidência é **reproduzível por outra pessoa da equipe** e está **registrada no repositório**. Ver [Checklist do Checkpoint 1](checkpoint-1.md).

## Histórico de versões

| Versão | Data | Autor | Alterações |
|---|---|---|---|
| 1.0 | 14/10/2026 | @osamuelleite | Versão inicial, conciliação PO + Tech Lead. Da PO: itens da DoD. Do Tech Lead: itens obrigatórios do Caderno (seção 1.6) e Definition of Ready (C12). |
| 1.1 | 31/10/2026 | @pedrolmelo | Ajuste do critério de novos recursos em nuvem para abranger serviços fora do GCP (Plano Custo Zero). |
