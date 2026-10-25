# Casos gerados pelo prompt orientado por contrato

**Estado: especificados, não executados contra uma API.**

Precondições comuns: API e PostgreSQL isolados em execução, fixture carregada, catálogo conforme contrato e token local válido com `indicadores:consultar`. Em cada caso, enviar POST `/analytics/summary` com JSON e Bearer token. Verificar HTTP 200, Content-Type JSON e igualdade exata do corpo abaixo. A interação deve percorrer cliente HTTP → API → banco real. Limpeza: encerrar conexões do cliente e restaurar o estado inicial; nenhuma rota de consulta deve alterar a fixture.

## C01 — Todo o histórico, sem filtro de UF

Requisição:

```json
{"competenciaInicio": "2020-01", "competenciaFim": "2025-12"}
```

Resposta esperada:

```json
{"registros": 5, "apacs": 5, "valorAprovadoCentavos": 150000}
```

## C02 — Filtrar apenas SP

Requisição:

```json
{"competenciaInicio": "2020-01", "competenciaFim": "2025-12", "ufs": ["SP"]}
```

Resposta esperada:

```json
{"registros": 3, "apacs": 3, "valorAprovadoCentavos": 80000}
```

## C03 — Filtrar apenas GO

Requisição:

```json
{"competenciaInicio": "2020-01", "competenciaFim": "2025-12", "ufs": ["GO"]}
```

Resposta esperada:

```json
{"registros": 2, "apacs": 2, "valorAprovadoCentavos": 70000}
```

## C04 — Um mês intermediário

Requisição:

```json
{"competenciaInicio": "2022-06", "competenciaFim": "2022-06"}
```

Resposta esperada:

```json
{"registros": 1, "apacs": 1, "valorAprovadoCentavos": 30000}
```

## C05 — Extremo inferior inclusivo

Requisição:

```json
{"competenciaInicio": "2020-01", "competenciaFim": "2020-01"}
```

Resposta esperada:

```json
{"registros": 2, "apacs": 2, "valorAprovadoCentavos": 30000}
```

## C06 — Extremo superior inclusivo

Requisição:

```json
{"competenciaInicio": "2025-12", "competenciaFim": "2025-12"}
```

Resposta esperada:

```json
{"registros": 2, "apacs": 2, "valorAprovadoCentavos": 90000}
```

## C07 — Período coberto sem ocorrências

Requisição:

```json
{"competenciaInicio": "2021-01", "competenciaFim": "2021-12"}
```

Resposta esperada:

```json
{"registros": 0, "apacs": 0, "valorAprovadoCentavos": 0}
```

## C08 — UFs vazias significam todas

Requisição:

```json
{"competenciaInicio": "2020-01", "competenciaFim": "2025-12", "ufs": []}
```

Resposta esperada:

```json
{"registros": 5, "apacs": 5, "valorAprovadoCentavos": 150000}
```
