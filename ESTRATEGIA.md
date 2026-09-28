# Estratégia de teste

## Contexto e objetivo da validação

A v2 traz o desconto por volume. Esta validação existe para embasar a decisão de liberar (ou não) a v2 sem gerar risco financeiro nem de negócio. A pergunta que precisei responder: **"Podemos subir a v2 com segurança e sem causar impacto financeiro?"**

## Análise de risco


| Área | O que pode dar errado | Impacto se acontecer | Probabilidade | Prioridade |
|---|---|---|---|---|
| Cálculo da cotação (faixa de peso, multiplicador, imposto) | Valor base calculado com a faixa errada, principalmente nos limites (o README define limites inclusivos); multiplicador ou imposto alterados sem aviso | Crítico: afeta toda cotação criada. Pode gerar cobrança a mais (dano financeiro ao cliente, perda de clientes ) ou a menos (perda financeira para a empresa) | Alta (confirmado: BUG-001, limites de 10, 50 e 100 kg) | 1 |
| Desconto por volume (feature nova) | Percentual errado nas faixas ou nos limites (10, 20 e 50 volumes); desconto aplicado ou ausente indevidamente | Crítico: mexe direto no valor cobrado| Alta (confirmado: BUG-002) | 1 |
| Faturamento e não retroatividade | valor de cotação já faturada recalculado com o desconto (cotações faturadas na carga inicial de dados não podem ter valor recalculado)| Crítico: inconsistência financeira | Indefinido - foi confirmado BUG de alteração do valor das cotações, mas não houve confirmação de refaturamento. Comportamento está documentado no BUG-004 | 1 |
| Contrato da API entre v1 e v2 | Campos renomeados ou com valor diferente entre rotas; mudança de tipo | Importante: quebra de integrações e de telas que consomem a API | Média. Obs.: rota verificada: POST/api/cotacoes | 2 |
| Arredondamento do valor final | Truncar em vez de arredondar pela regra comercial; total sem duas casas decimais | Médio em cada cotação (centavos), mas se acumula em volume e chega às faturas emitidas | Alta (confirmado: BUG-003) | 2 |

## Fontes de verdade usadas

Usei o README como referência do que é correto hoje (regras vigentes na v1 e contrato da API), a SPEC como referência da regra nova, e a v1 como comportamento de comparação para tudo que não deveria mudar. O CHANGELOG serviu para saber onde o time de desenvolvimento afirmou ter mexido, mas não o tratei como limite do que pode ter sido afetado.


## Abordagem por área

| Área | Como testei |
|---|---|
| Faixa de peso, multiplicador e imposto | Postman + Newman, valores com pesos nos limites e ao redor (0.5, 0.99, 10, 10.01, 50, 50.01, 100, 100.01 e 1000 kg), mesma UF e 5 volumes para isolar o efeito do peso | |
| Desconto por volume | Postman + Newman. Casos nos limites das faixas (9, 10, 19, 20, 49, 50 e 51 volumes) contra a tabela da SPEC, usando 50.5 kg e mesma UF para isolar dos limites de peso | |
| Arredondamento | Postman + Newman - 9 casos com terceira casa decimal (7 devem subir, 2 devem cair) e 2 casos de formato (total terminando em zero). | |
| Faturamento | Manual, via Postman: fatura de cotação nova, repetição (409), cotação inexistente (404), listagem por id_cotacao e conferência da carga inicial  |  |
| Impacto na carga inicial | Script que compara, para as 60 cotações faturadas, o detalhe na v1 e na v2, mais contagens por regra sobre as 200 cotações |  |
| Tela | Smoke test manual no navegador: detalhe da cotação, percentual de desconto e formato do valor | |

**Ferramentas**: Postman (versão gratuita) e Newman, com scripts em JavaScript. Como a versão gratuita não permite arquivo de dados, escrevi os casos manualmente, dentro do próprio script. 

## O que decidi NÃO testar

| Ficou de fora | Por quê | Risco que estou aceitando |
|---|---|---|
| Limite exato do arredondamento (terceira casa igual a 5) | Inalcançável via API: nenhuma combinação de faixa de peso, multiplicador e desconto produz ",xx5". Testei os vizinhos (terceira casa 4 e 6) | Baixo. Só um teste unitário na função de arredondamento cobriria |
| Validações 422 (a não ser post de cotação), filtro por cliente e paginação em detalhe | Tempo disponível; regras simples e menos ligadas ao valor cobrado | Médio |
| Validação dos campos nas requests e responses (verifiquei rota POST api/cotacoes por ser a rota mais crítica)| Limitação do tempo disponível e priorização da rota mais crítica  | Baixo a médio |
| Teste de carga | Tempo disponível e fora do escopo da mudança | Baixo |
| Usabilidade e experiência do usuário | Tempo disponível; o foco foi a regra de negócio | Baixo |
| Leitura do código-fonte | limitação do tempo disponível | Baixo |

## Ambiente e dados

- Servidores locais, uma instância por versão, subidos pelo terminal do VSCode.  Node v24.21.0.
- Postman 12.29.5 e Newman  6.2.2
- Dados: carga inicial (200 cotações e 60 faturas), restaurada com `POST /_reset` antes de cada execução. Os testes criam cotações novas (ids a partir de 201), que não afetam a contagem sobre a carga inicial.

## Limitações da minha análise

- **Não analisei o código-fonte.** As causas prováveis dos bugs são apenas sugestões. Para confirmar, deveria acontecer uma leitura detalhada do código. 

- **Faturamento foi testado manualmente e por amostra**, sem automação. Concorrência, ids inválidos e outros cenários serão feitos futuramento.
- **A versão gratuita do Postman impede arquivo de dados**, então os casos ficam embutidos nos scripts. Mudar ou adicionar casos exige editar o script.
