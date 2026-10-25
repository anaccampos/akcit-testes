# Contrato do cenário de integração

Este é um cenário simplificado e proposto do DataSUS Quimioterapia. Não representa endpoints já implantados. Os dados são inteiramente fictícios.

## Fronteira e ambiente de teste

Cliente HTTP de teste → API analítica → PostgreSQL de teste. O cliente substitui a interface para exercitar o protocolo HTTP, a serialização, a validação da requisição e a consulta real ao banco. Não é um teste de navegador nem apenas um teste unitário da função de filtros.

A futura automação deverá iniciar a API com o acesso real ao PostgreSQL de teste, aplicar o esquema e carregar as fixtures. Nenhuma consulta SQL será substituída por mock nos casos de sucesso. Banco e API devem ser isolados por execução; cada caso deve restaurar suas alterações. O provedor de identidade é substituído por um emissor local de tokens de teste, com chave exclusiva para testes. Login, PKCE, rotação de chaves e integração com um provedor real estão fora do escopo.

## Requisição

`POST /analytics/summary`, com `Content-Type: application/json` e `Authorization: Bearer <token de teste>`.

```json
{"competenciaInicio":"2020-01","competenciaFim":"2025-12","ufs":["SP"]}
```

- Intervalo obrigatório e inclusivo. Formato exato `AAAA-MM`, anos 0001–9999 e meses 01–12. Espaços não são removidos.
- Início deve ser menor ou igual ao fim.
- `ufs` é opcional. Ausência ou lista vazia significa todas as UFs disponíveis. Neste toy problem, apenas SP e GO são valores permitidos; duplicatas são rejeitadas.
- O catálogo fictício cobre 2020-01 a 2025-12, sem lacunas, para SP e GO. Consulta fora desse intervalo é rejeitada, não recortada.
- Permissão necessária: `indicadores:consultar`.
- Ordem de processamento: autenticação → autorização → validação → consulta. Testes de prioridade de erros usam JSON sintaticamente válido.
- Cada registro representa uma APAC distinta. O total de APACs conta `apacId` distinto e não pacientes únicos. Valores são inteiros em centavos.

## Sucesso

HTTP 200, JSON com `registros`, `apacs` e `valorAprovadoCentavos`, todos inteiros não negativos. Intervalo coberto sem ocorrências retorna os três campos com zero. Campos adicionais não fazem parte deste contrato.

## Erros

Corpo JSON: `{"erro":{"codigo":"...","mensagem":"..."}}`. A mensagem deve ser não vazia e em português; o texto literal não é fixado. Não deve expor senha, token, stack trace ou SQL.

| HTTP | Código | Condição |
|---|---|---|
| 401 | NAO_AUTENTICADO | Token ausente, inválido ou expirado |
| 403 | SEM_PERMISSAO | Token válido sem a permissão exigida |
| 400 | FILTRO_INVALIDO | Campos ausentes, tipos incorretos, formato ou ordem inválidos, UFs inválidas ou duplicadas |
| 400 | PERIODO_FORA_DO_HISTORICO | Intervalo válido fora da cobertura |
| 503 | DADOS_INDISPONIVEIS | Falha de conexão ou timeout na consulta |

Um token expirado é emitido com expiração suficientemente anterior ao relógio fixado do teste. Um token inválido tem assinatura não confiável. Não se usa token real nem se versiona chave privada de produção.

## Falhas controladas

Banco indisponível: configurar a instância isolada da API para uma porta local sem serviço. Timeout: usar atraso controlado na comunicação API–PostgreSQL superior ao prazo configurado para o teste. Interceptar a fronteira de transporte para introduzir atraso é permitido; devolver um resultado SQL inventado no lugar da consulta não é.

Os mecanismos exatos de provisionamento, emissor local, timeout e restauração do ambiente precisarão ser implementados. Este repositório registra casos de teste, não uma aplicação executável.
