# DataSUS · Casos de teste de integração com GenAI

Experimento de geração de casos de integração para uma API de indicadores históricos de quimioterapia. O cenário utiliza dados fictícios e uma arquitetura proposta: **cliente HTTP → API TypeScript/Fastify → PostgreSQL de teste**.

Foram utilizadas duas técnicas: **zero-shot**, com contrato e contexto, mas sem exemplos resolvidos de testes; e **few-shot**, com duas demonstrações completas para orientar novos casos de falha. As versões e os resultados estão registrados neste repositório.

**Entrega:** 30 casos documentados — 8 positivos/de limite, 18 negativos/de recuperação do catálogo inicial e 4 adicionais produzidos após a introdução de few-shot. São especificações para uma futura suíte automatizada; não foram executados contra uma API. A conferência dos totais fictícios não equivale a execução de integração.

## Cenário e contrato

A rota `POST /analytics/summary` recebe intervalo de competências e UFs, valida identidade e permissão e retorna quantidade de registros, APACs distintas e valor aprovado em centavos. O PostgreSQL contém histórico fictício de vários anos. A alimentação da base, a interface visual e o login em um provedor real estão fora do recorte.

O [contrato completo](docs/contrato.md) define as entradas, a fixture, os códigos HTTP, os limites e a ordem das validações. Essas regras pertencem ao toy problem e não representam um contrato de produção.

O teste de integração deve exercitar HTTP e persistência real. A identidade é controlada com tokens locais. Não basta chamar uma função de validação nem substituir todas as dependências por mocks.

## Prompts e resultados

| Tipo | Efeito na geração | Resultado |
|---|---|---|
| [Zero-shot orientado por contrato](prompts/01-contrato.md) | Produz resultados precisos e comparáveis a partir de entradas, regras e fixtures | [C01–C08](casos/contrato.md): filtros, limites inclusivos, mês único, ausência de ocorrências e semântica de UFs vazias |
| [Few-shot orientado por falhas](prompts/02-falhas.md) | Demonstra como relacionar preparação, erro, evidência de interação e limpeza | [FS01–FS04](casos/few-shot.md): tipos incorretos, null, elementos inválidos e falhas simultâneas |

O catálogo [F01–F18](casos/falhas.md) foi gerado anteriormente com instruções orientadas por falhas, ainda sem demonstrações. Ele foi mantido como referência, sem atribuir sua geração ao few-shot.

## Ajustes de prompt e análise crítica

Os [resultados iniciais](casos/iniciais.md) eram úteis como lista de ideias, mas genéricos: “verificar totais”, “retornar erro” e “simular lentidão” não definem critérios executáveis.

No primeiro prompt, foram acrescentados contrato, fixture e resultados exatos, além da distinção entre período coberto sem ocorrências e período fora do histórico. Isso tornou os casos mais precisos e evitou respostas alternativas como “vazio ou zero”.

No segundo, foram especificados os códigos HTTP, a ordem de processamento, as formas de provocar falhas e a restauração do ambiente. Também foi proibido inventar fallback e retentativas. Assim, a geração passou a considerar erros simultâneos e a interação API–banco, em vez de apenas variações de campos.

Na revisão dos casos, foi explicitado que uma API que não inicia sem banco exige outro mecanismo de injeção de falha; startup malsucedido não comprova uma resposta 503. Também foi distinguida a recuperação por nova requisição de uma retentativa automática, que não faz parte do contrato.

A quantidade de casos não é suficiente para demonstrar qualidade. Os critérios adotados foram rastreabilidade ao contrato, resultado verificável, fronteiras exercitadas, isolamento e ausência de decisões inventadas.

## Aplicação de zero-shot e few-shot

O prompt 1 permanece zero-shot mesmo após receber contrato e fixtures: não contém demonstrações de casos resolvidos. Seu resultado oferece casos com valores esperados exatos.

Na revisão do prompt 2, foram acrescentadas duas demonstrações: requisição sem token e banco indisponível. Elas mostram o nível de detalhe desejado, incluindo a observação da comunicação e a limpeza. O modelo produziu quatro casos adicionais com essa estrutura, em vez de apenas títulos de cenários.

Foi necessário pedir explicitamente casos novos e fornecer o catálogo anterior para evitar repetir os exemplos. Os novos casos ampliam a cobertura de tipos de entrada e da prioridade entre falhas. Ainda assim, as instruções refinadas anteriores já exigiam esse formato: não é possível atribuir a diferença exclusivamente aos exemplos.

Esta é uma aplicação qualitativa das técnicas, não um experimento controlado: os escopos e o contexto também variaram, e a mesma conversa contém os resultados anteriores. Para comparar efeitos isolados, seria necessário usar a mesma tarefa em sessões independentes, alterando apenas a presença das demonstrações.

## Registro do experimento

A geração foi realizada nesta conversa com a LLM do Codex: os dois prompts iniciais produziram esboços, e os refinamentos produziram os casos detalhados. A revisão foi assistida, mas sem avaliação independente por outro modelo. Não são transcrições de chamadas de API separadas. O few-shot foi aplicado posteriormente aos quatro casos FS01–FS04, após a inclusão dos exemplos no prompt.

Foram conferidas as somas das fixtures e as referências locais dos documentos. Os casos não comprovam funcionamento da autenticação, da API ou do PostgreSQL: essa evidência depende da futura automação e execução.

## Próximo passo

Implementar um harness com API e PostgreSQL isolados, carregar as fixtures, emitir tokens locais e automatizar as requisições/assertivas. Os cenários de falha exigem controle do transporte e limpeza confiável. Os detalhes pendentes estão descritos nos casos e no contrato.

Dados fictícios: [registros.json](dados/registros.json).
