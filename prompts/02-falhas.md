# Prompt 2 — few-shot orientado por falhas e riscos

A técnica **few-shot** fornece exemplos resolvidos para orientar o padrão das respostas. Aqui, as demonstrações mostram preparação, resultado verificável, observação da integração e limpeza.

## Versão inicial — ainda sem exemplos

> Gere casos de erro para a integração de uma API analítica com PostgreSQL. Considere autenticação, filtros inválidos e indisponibilidade.

A saída exploratória está em `casos/iniciais.md`. O pedido não fixa códigos de erro nem o modo de provocar falhas.

## Versão refinada — few-shot

> Atue como revisor de testes de integração com foco em falhas de comunicação e contratos. Use somente `docs/contrato.md` e `dados/registros.json`, fornecidos como contexto. Considere cliente HTTP → API → PostgreSQL de teste.
>
> Gere casos negativos para token ausente, expirado e de assinatura inválida, falta de permissão, filtros inválidos, período fora do histórico, banco indisponível e timeout. Inclua erros simultâneos para verificar a ordem autenticação → autorização → validação → consulta.
>
> Informe por caso: ID, risco, preparação, requisição ou alteração sobre uma requisição válida, status/código esperado, observações que comprovam a interação e restauração do ambiente. Para entradas rejeitadas, verifique que não houve consulta analítica ao banco com instrumentação de teste. Para 503, descreva como introduzir a falha sem substituir o resultado SQL por mock.
>
> Não invente retentativas, fallback, renovação automática de token, circuit breaker ou texto literal de erro. Não transforme falha de banco em sucesso com zero. Use tokens fictícios emitidos localmente. Campos de diagnóstico sensíveis não devem aparecer na resposta.
>
> Use tabela de variações quando várias entradas compartilham o mesmo resultado, com identificação individual. Diferencie caso gerado de teste executado e declare as lacunas de automação.

>
> **Exemplo resolvido 1 — requisição sem token**
>
> ID: EX01. Risco: acesso sem autenticação.
> Preparação: API e banco de teste disponíveis, fixture carregada e contador de consultas analíticas zerado.
> Requisição: POST /analytics/summary com corpo válido, sem Authorization.
> Esperado: HTTP 401, erro.codigo = NAO_AUTENTICADO e mensagem não vazia em português.
> Evidência: nenhuma consulta analítica ao PostgreSQL. Não contar health checks.
> Limpeza: restaurar instrumentação e encerrar conexões do cliente.
>
> **Exemplo resolvido 2 — banco indisponível**
>
> ID: EX02. Risco: falha de persistência ser apresentada como resultado vazio.
> Preparação: API inicialmente saudável, identidade de teste válida e conectividade com o PostgreSQL interrompida de forma controlada após a inicialização.
> Requisição: POST /analytics/summary com token autorizado e corpo válido.
> Esperado: HTTP 503, erro.codigo = DADOS_INDISPONIVEIS. Não retornar HTTP 200 com totais zerados.
> Evidência: tentativa real de comunicação com o banco e falha registrada no ambiente de teste, sem expor SQL ou credenciais ao cliente.
> Limpeza: restaurar a comunicação, descartar conexões defeituosas e aguardar prontidão dentro do limite do harness.
>
> **Sua tarefa:** seguindo o nível de detalhe desses exemplos, gere quatro casos adicionais que ainda não estejam em casos/falhas.md. Não repita os exemplos. Varie tipos de entrada e combinação de falhas sem inventar regras. Use IDs FS01–FS04 e explicite a relação entre a preparação, a resposta e a evidência de integração. Considere também o catálogo existente como contexto.

## Registro da aplicação

Os exemplos foram acrescentados nesta revisão dos prompts. O catálogo F01–F18 já existia e não é atribuído retroativamente ao few-shot. A nova saída está em [casos/few-shot.md](../casos/few-shot.md). EX01 e EX02 demonstram situações já presentes no catálogo anterior e não contam como novos casos.
