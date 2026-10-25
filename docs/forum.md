# Texto para publicação

Repositório: https://github.com/anaccampos/akcit-testes

Escolhi uma integração simplificada do DataSUS: um cliente HTTP envia filtros para uma API em TypeScript, que valida o acesso e consulta indicadores históricos em PostgreSQL. Utilizei dados fictícios e apliquei as técnicas zero-shot e few-shot na geração de casos de teste.

No prompt zero-shot, forneci o contrato da API, as regras e os dados de teste, mas não apresentei exemplos resolvidos de casos. Isso ajudou a gerar cenários de sucesso e de limite com resultados verificáveis, como filtros por UF, mês único e período coberto sem ocorrências. O principal refinamento foi substituir instruções genéricas por status HTTP e valores esperados definidos a partir da fixture.

No prompt few-shot, acrescentei dois exemplos completos: uma requisição sem token e uma falha de comunicação com o banco. Cada exemplo mostrava preparação, requisição, resposta esperada, evidência da interação e limpeza. Solicitei novos casos seguindo esse padrão, sem repetir os exemplos. A saída manteve essa estrutura e acrescentou situações como tipos incorretos, UF nula e falta de autenticação combinada com banco indisponível.

Foi necessário explicitar quais componentes seriam reais, quais seriam controlados e quais comportamentos não deveriam ser inventados, como retentativas automáticas. Também forneci o catálogo anterior para reduzir duplicações. O repositório reúne os prompts, os exemplos e 30 casos documentados, dos quais quatro foram acrescentados na etapa few-shot. Os casos ainda não foram executados contra uma API.

A comparação foi qualitativa: além dos exemplos, o foco e o contexto dos pedidos variaram, portanto não atribuo as diferenças apenas à técnica. O aprendizado foi que zero-shot com um contrato claro já pode produzir casos úteis, enquanto few-shot ajuda a demonstrar o nível de detalhe esperado. Em ambos, a revisão é necessária para conferir aderência às regras e evitar lacunas ou comportamentos inventados.
