# Saídas iniciais exploratórias

Estas saídas foram geradas pela IA nesta atividade antes de detalhar os casos finais. São esboços baseados nos prompts iniciais, não resultados de execução.

## Prompt 1 — contrato, versão inicial

1. Consultar um intervalo válido e verificar os totais retornados.
2. Consultar apenas uma UF e verificar se o resultado exclui as demais.
3. Consultar um período sem ocorrências e verificar se o resumo é vazio ou zerado, conforme o contrato.
4. Consultar um único mês e os extremos do histórico.

**Limitações encontradas:** não há valores esperados nem fixture; “vazio ou zerado” exige uma decisão; não está claro se o PostgreSQL é real ou substituído por mock.

## Prompt 2 — falhas, versão inicial

1. Enviar uma requisição sem token e verificar a rejeição.
2. Enviar um intervalo invertido e verificar erro de validação.
3. Indisponibilizar o banco e verificar uma resposta de erro.
4. Simular lentidão e verificar o tratamento de timeout.

**Limitações encontradas:** faltam status/códigos, método de injeção de falha, ordem de validação e observações sobre a comunicação entre componentes.

## Decisões adotadas no refinamento

Fixar fixture e respostas; retornar zeros para período coberto sem ocorrências; rejeitar período fora do histórico; usar PostgreSQL de teste real; definir 401/403/400/503; especificar prioridade das validações e separar login real do escopo. Essas são decisões do toy problem, não funcionalidades comprovadas de um sistema existente.
