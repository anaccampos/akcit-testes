# Casos gerados pelo prompt orientado por falhas

**Estado: especificados, não executados contra uma API.**

Base válida: `{"competenciaInicio":"2020-01","competenciaFim":"2025-12","ufs":["SP"]}`. Enviar POST `/analytics/summary` com Content-Type JSON. Salvo alteração explícita, usar token local válido com `indicadores:consultar` e banco com fixture inicial.

Em todas as respostas de erro, verificar o status/código indicado, mensagem não vazia em português e ausência de SQL, stack trace, tokens e credenciais. Não comparar a frase literal. Não aceitar corpo de sucesso junto com erro.

## Identidade e autorização

| ID | Preparação/entrada | Resultado esperado | Observação de integração |
|---|---|---|---|
| F01 | Omitir Authorization | 401 / NAO_AUTENTICADO | Nenhuma consulta analítica |
| F02 | Token local expirado em relação ao relógio fixado | 401 / NAO_AUTENTICADO | Nenhuma consulta analítica |
| F03 | Token assinado por chave não confiável | 401 / NAO_AUTENTICADO | Nenhuma consulta analítica |
| F04 | Token válido sem indicadores:consultar | 403 / SEM_PERMISSAO | Nenhuma consulta analítica |
| F05 | Sem token e inicio = 2025-13 | 401 / NAO_AUTENTICADO | Autenticação precede validação dos filtros |
| F06 | Sem permissão e intervalo invertido | 403 / SEM_PERMISSAO | Autorização precede validação dos filtros |

Em F05, “inicio” significa o campo `competenciaInicio`. O corpo permanece JSON sintaticamente válido.

Preparação comum: emitir os tokens apenas no ambiente local de testes. Instrumentar a execução de consultas analíticas com contador ou log de teste, mantendo o acesso real ao banco; excluir conexões de inicialização e health checks da contagem. Limpeza: restaurar relógio e instrumentação entre casos, descartar tokens e encerrar o emissor de teste ao final.

## Validação e limites

| ID | Alteração no corpo válido | Resultado esperado |
|---|---|---|
| F07 | competenciaInicio = 2025-13 | 400 / FILTRO_INVALIDO |
| F08 | competenciaInicio = 2025-12, competenciaFim = 2020-01 | 400 / FILTRO_INVALIDO |
| F09 | Remover competenciaInicio | 400 / FILTRO_INVALIDO |
| F10 | ufs = "SP" em vez de lista | 400 / FILTRO_INVALIDO |
| F11 | ufs = ["XX"] | 400 / FILTRO_INVALIDO |
| F12 | ufs = ["SP", "SP"] | 400 / FILTRO_INVALIDO |
| F13 | competenciaInicio = 2019-12 | 400 / PERIODO_FORA_DO_HISTORICO |
| F14 | competenciaFim = 2026-01 | 400 / PERIODO_FORA_DO_HISTORICO |
| F15 | competenciaInicio = " 2020-01" | 400 / FILTRO_INVALIDO |

F07–F12 e F15 devem ser rejeitados antes de qualquer consulta analítica. F13–F14 podem consultar o catálogo de cobertura, mas não devem executar a agregação nem recortar o intervalo. A fixture deve permanecer inalterada. Restaurar a requisição-base e zerar a instrumentação em cada caso.

## Comunicação API–PostgreSQL

### F16 — conexão indisponível

- Preparação: iniciar instância isolada da API com endpoint PostgreSQL local sem serviço; manter a configuração de identidade válida. A API de teste deve poder responder HTTP mesmo sem conexão disponível no momento da consulta.
- Ação: enviar a requisição-base com token autorizado.
- Esperado: HTTP 503 / DADOS_INDISPONIVEIS; nunca HTTP 200 com zeros.
- Evidência: registrar a tentativa de conexão e a falha na fronteira API–banco, sem expor credenciais na resposta.
- Limpeza: descartar a instância defeituosa e restaurar a configuração normal. Se a aplicação impedir inicialização sem banco, adaptar o harness para interromper a conectividade após a inicialização; não considerar falha de startup como resposta HTTP 503.

### F17 — timeout de consulta

- Preparação: banco saudável e conexão estabelecida. Introduzir atraso controlado no transporte da consulta maior que o prazo configurado para este teste.
- Ação: enviar a requisição-base autorizada.
- Esperado: HTTP 503 / DADOS_INDISPONIVEIS, sem resultado parcial ou zeros. Usar prazo finito de espera no cliente de teste, maior que o timeout da API e com margem de execução.
- Evidência: confirmar que a consulta foi tentada e o timeout foi tratado. Não substituir a consulta por um stub que lança erro.
- Limpeza: remover o atraso, cancelar consultas pendentes e restaurar conexões/configuração.

### F18 — recuperação após a falha

- Preparação: após restaurar o ambiente de F16 ou F17, aguardar o sinal de prontidão dentro de um limite definido no harness.
- Ação: enviar uma NOVA requisição-base com token autorizado.
- Esperado: HTTP 200 e `{"registros":3,"apacs":3,"valorAprovadoCentavos":80000}`.
- Evidência: nova consulta real bem-sucedida. Este caso não exige retentativa automática da requisição anterior.
- Limpeza: restaurar a fixture e encerrar recursos da execução.

## Lacunas de automação

Ainda precisam ser definidos e implementados o esquema físico, a inicialização da API, o emissor local, a instrumentação, os prazos numéricos e o mecanismo de atraso. Não há evidência de execução desses cenários nesta entrega.
