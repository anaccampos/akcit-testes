# Casos adicionais gerados com few-shot

Gerados após acrescentar duas demonstrações resolvidas ao prompt 2. **São especificações, não testes executados.** Os quatro casos abaixo são novos em relação a F01–F18; os exemplos EX01 e EX02 não são contados novamente.

Base: POST `/analytics/summary`, Content-Type JSON, corpo `{"competenciaInicio":"2020-01","competenciaFim":"2025-12","ufs":["SP"]}`. Salvo indicação contrária, usar token local autorizado, API e PostgreSQL de teste saudáveis e fixture carregada.

## FS01 — competência final com tipo numérico

- Risco: aceitar coerção silenciosa de tipo e produzir uma consulta diferente da solicitada.
- Preparação: ambiente-base e contador de consultas analíticas zerado.
- Requisição: substituir `competenciaFim` por `202512` (número).
- Esperado: HTTP 400, `erro.codigo = FILTRO_INVALIDO` e mensagem não vazia em português.
- Evidência: a requisição atravessa HTTP e validação da API, sem executar consulta analítica. O servidor não converte o número para uma competência válida.
- Limpeza: restaurar o corpo-base, a instrumentação e as conexões do cliente.

## FS02 — null no lugar de lista de UFs

- Risco: interpretar um tipo inválido como ausência de filtro e ampliar a consulta.
- Preparação: ambiente-base e contador de consultas analíticas zerado.
- Requisição: substituir `ufs` por `null`.
- Esperado: HTTP 400, `erro.codigo = FILTRO_INVALIDO`. Ausência do campo e lista vazia são válidas, mas null não é uma lista nem ausência.
- Evidência: rejeição na API sem consulta analítica; não retornar os totais de todas as UFs.
- Limpeza: restaurar corpo-base e instrumentação; encerrar conexões do cliente.

## FS03 — elemento de UF com tipo incorreto

- Risco: validar apenas que ufs é uma lista, sem validar seus elementos.
- Preparação: ambiente-base e contador de consultas analíticas zerado.
- Requisição: substituir `ufs` por `["SP", 42]`.
- Esperado: HTTP 400, `erro.codigo = FILTRO_INVALIDO`.
- Evidência: rejeitar a requisição inteira, sem consultar apenas SP e sem converter 42 para string. Não executar consulta analítica.
- Limpeza: restaurar corpo-base e instrumentação; encerrar conexões do cliente.

## FS04 — falta de autenticação com banco indisponível

- Risco: a API depender do banco antes de validar o acesso e responder com um erro de infraestrutura no lugar da rejeição de autenticação.
- Preparação: iniciar a API saudável, depois interromper a conectividade API–PostgreSQL. Manter a instrumentação capaz de observar tentativas de consulta, sem contabilizar health checks.
- Requisição: corpo-base válido, sem Authorization.
- Esperado: HTTP 401, `erro.codigo = NAO_AUTENTICADO`, e não HTTP 503.
- Evidência: a requisição é rejeitada antes de uma tentativa de consulta analítica, conforme a ordem de processamento definida no contrato.
- Limpeza: restaurar conectividade e instrumentação, encerrar conexões defeituosas e aguardar prontidão com limite no harness.

Em todos os erros: resposta JSON, mensagem não vazia em português e ausência de token, senha, stack trace ou SQL. Os mecanismos de instrumentação e injeção de falhas ainda precisam ser automatizados.
