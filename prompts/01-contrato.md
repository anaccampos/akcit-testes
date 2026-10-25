# Prompt 1 — zero-shot orientado por contrato

A técnica **zero-shot** solicita a tarefa sem apresentar exemplos resolvidos de casos de teste. O contrato, as fixtures e o formato de saída continuam sendo fornecidos: contexto não é o mesmo que uma demonstração da resposta.

## Versão inicial

> Gere casos de teste de integração para uma API do DataSUS que recebe filtros de período e UF e consulta um resumo de indicadores no PostgreSQL. Inclua situações normais e de limite. Use dados fictícios.

A saída exploratória está em `casos/iniciais.md`. A solicitação não determina cobertura, fixture, respostas nem fronteiras substituídas.

## Versão refinada

> Atue como analista de testes de integração. Considere exclusivamente o contrato em `docs/contrato.md` e os registros fictícios em `dados/registros.json`, fornecidos como contexto junto deste prompt.
>
> Gere oito casos positivos e de limite para POST /analytics/summary. Exercite cliente HTTP, API e PostgreSQL de teste. A API e o acesso ao banco devem ser reais; apenas a identidade será fornecida pelo emissor local descrito no contrato.
>
> Cubra intervalo completo, filtro por UF, um único mês, extremos inferior e superior, período sem ocorrências, ausência de ufs e lista vazia. Calcule respostas exatas a partir da fixture. Não confunda período sem ocorrências com período fora do histórico.
>
> Para cada caso, informe ID, objetivo, precondições, requisição, status e JSON esperado, componentes envolvidos e limpeza necessária. Use token de teste válido com a permissão exigida. Não adicione campos ou regras ao contrato. Se faltar uma decisão, registre a lacuna em vez de inventar o resultado.
>
> Organize a resposta em casos rastreáveis. Não declare que os testes foram executados: a tarefa é gerar especificações de teste.
