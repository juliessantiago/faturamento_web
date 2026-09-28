# Regressão 

> **Atenção:** todos os testes ficam na pasta `postman/` e não aqui dentro da pasta "regressão" por uma questão de organização do código. 

## Como rodar

Ferramenta: **Postman + Newman** (o executor de linha de comando do Postman). Os testes foram criados em uma única collection no Postman, separados em pastas. Cada pasta tem um teste diferente, com suas requisições e seus scripts. Depois de criados, a collection e o environment foram exportados para `postman/` e são executados pelo terminal, dentro do VSCode.

### Pré-requisitos

- Node.js 18 ou superior (o mesmo do projeto).
- Dependências de teste instaladas uma vez: `npm install` (instala o Newman como dependência de desenvolvimento; o servidor continua sem dependências).

### Pré-condições

Subir as duas versões da API, cada uma em um terminal:

```
node server.js v1   # http://localhost:3001
node server.js v2   # http://localhost:3002
```

Recomendado: restaurar a carga inicial nas duas antes de rodar (`POST /_reset` em cada porta, ou o botão "Resetar dados" na tela). Os testes criam cotações novas (ids a partir de 201), e o teste `impacto_carga_inicial` assume os dados originais.

### Para rodar todos os testes

Em um terceiro terminal, na raiz do projeto:

```
npx newman run postman/collection.json -e postman/environment.json
```

### Rodar só um teste


```
npx newman run postman/collection.json -e postman/environment.json --folder "nome_da_pasta"
```


### Por que escolhi essas ferramentas

Optei por testar via API com Postman por já ter experiência com a ferramenta, o que me permitiu focar na cobertura das regras de negócio em vez de aprender uma nova tecnologia para os testes. A mesma collection roda pela interface do Postman e pelo terminal via Newman (para execução). Usei a versão gratuita do Postman, que não permite arquivo de dados, então os casos de cada teste ficam em uma lista dentro do próprio script.

## O que está sendo testado

| Pasta | O que verifica |
|---|---|
| `valor_borda_peso` | Faixa de peso nos limites e ao redor deles, comparando v1 e v2 |
| `valor_borda_desconto` | Percentual de desconto nos limites das faixas de volume (v2) |
| `arredondamento_desconto` | Arredondamento do valor final com desconto e formato de duas casas decimais (v2) |
| `post_cotacao_response` | Código da resposta, estrutura da resposta, estrutura do objeto retornado|
| `post_cotacao_request_campos_ausentes` | Código da resposta (422) e status quando os campos obrigatórios não são informados |
| `post_cotacao_request_peso` | Código da resposta (422) e status quando peso é informado sendo 0, negativo e menor valor positivo (0.01)|
| `post_cotacao_request_volume` | Código da resposta (422) e status quando volume é negativo, igual a zero, menor valor aceitável (1) ou valor muito elevado (2000)|
| `impacto_carga_inicial` | Quantas das 60 cotações faturadas da carga inicial mudaram de valor na v2 (conferido no console) |

## Cenários cobertos

| # | Cenário | O que protege | v1 esperado | v2 esperado |
|---|---|---|---|---|
| 1 | Pesos 0.5, 0.99, 10, 10.01, 50, 50.01, 100, 100.01 e 1000 kg, 5 volumes, mesma UF | Faixa de peso com limites inclusivos (README) | "valor_base" 25, 25, 25, 60, 60, 110, 110, 180, 180 | Igual à v1. **Hoje falha em 10, 50 e 100 kg** (BUG-001) |
| 2 | 9, 10, 19, 20, 49, 50 e 51 volumes, 50.5 kg, mesma UF | Tabela de desconto por volume (SPEC) | Não se aplica (a v1 não tem desconto) | "desconto" 0, 0.05, 0.05, 0.10, 0.10, 0.15, 0.15; "valor_total" = 123,20 × (1 − desconto). **Hoje falha em 10, 20 e 50 volumes** (BUG-002) |
| 3 | 9 cotações com terceira casa decimal (7 devem subir, 2 devem cair), com valor_base, multiplicador e desconto conferidos antes (premissas) | Arredondamento comercial do valor final (README) | Não se aplica (sem desconto não há terceira casa) | Ex.: 34 kg RS→SP 11 volumes = 121,30. **7 casos que devem subir no arredondamento falham** (BUG-003) |
| 5 | Criação de cotação válida | Contrato da API para criação da cotação | 201, retorno de objeto criado|   201, retorno de objeto criado|
| 6 | Cada um dos cinco campos ausente | Contrato da API para criação da cotação | 422 | 422|
| 7 | "peso_kg" zero ou negativo e menor valor positivo |Contrato da API para criação da cotação  | 422, 422, 201| 422, 422, 201 |
| 8 | "volumes" valor igual a zero, valor negativo, menor valor aceitável (1) e grande quantidade de volumes| Contrato da API para criação da cotação| 422, 422, 201, 201 | 422, 422, 201, 201 
| 9 | Cotações 1 a 60 (já faturadas), detalhe nas duas versões | Restrição de não retroatividade (Spec da nova funcionalidade)| Valor original | Igual ao da v1. **29 de 60 mudaram de valor** (BUG-004) |

## Como a suíte compara v1 e v2


- **Regras que não deveriam mudar (faixa de peso, multiplicador):** o script  de teste (feito no Postman e exportados na collection) cria a mesma cotação na v1 e na v2 e  compara. Cada caso tem duas verificações: o valor obtido contra o valor esperado fixo ("valor_base" da tabela do README) e a comparação v1 × v2. 
- **Regra nova (desconto por volume):** a v1 não tem a funcionalidade, então a comparação com ela não faz sentido. O teste roda só na v2, contra valores esperados fixos calculados a partir da SPEC (tabela de percentuais) e do README (faixa de peso, multiplicador, imposto de 12%). 
- **Arredondamento:** valores esperados fixos. Cada caso confere antes "valor_base", "multiplicador" e "desconto", para que uma falha de arredondamento não seja confundida com os outros defeitos. Escolhi casos em que arredondar e truncar (cortar o valor) dão resultados diferentes.
- **Impacto na carga inicial:** compara o detalhe "GET /api/cotacoes/{id}" de cada uma das 60 cotações faturadas na v1 e na v2. Verifica quantas foram afetadas. 


## O que esta suíte NÃO cobre

- **Faturamento:** foi testado manualmente, no Postman (nova cotação, tentativa de repetição com resposta 409, cotação inexistente que deve retornar 404). Não foram criados testes automatizados. 
- **Rotas de listagem, filtros, paginação e `GET /api/faturas`:** ficaram fora, por priorização de tempo.
- **Tela:** conferida manualmente por amostra, não automatizada. Em outro momento poderíamos automatizar através da ferramenta Cypress. 
- **Limite exato do arredondamento (terceira casa igual a 5):** nenhuma combinação de peso, rota e desconto produz esse valor via API. Só um teste unitário direto na função de arredondamento cobriria.


## Saída esperada

Estrutura da resposta do Newman: para cada requisição, o método e a URL com o status, seguidos das asserções (`√` passou, número = falhou), o quadro de resumo e a lista de falhas. As asserções do "impacto_carga_inicial" aparecem como informativas, e a contagem sai no console.

Resultado esperado por pasta, com a v2 no estado atual:

| Pasta | Asserções | Falhas esperadas | Bug |
|---|---|---|---|
| `valor_borda_peso` | 36 | 6 (10, 50 e 100 kg) | BUG-001 |
| `valor_borda_desconto` | 14 | 6 (10, 20 e 50 volumes) | BUG-002 |
| `arredondamento_desconto` | 55 | 16 (14 nos 7 casos que devem subir e 2 de formato) | BUG-003 |
| `impacto_carga_inicial` | 1 (informativa) | 0 (o número aparece no console: 29 de 60) | BUG-004 |

Obs.: BUG 005 refere-se a ausencia do valor da fatura na rota GET /api/faturas?id_cotacao={id}, de forma manual no Postman

**Exemplo de resposta do Newman**

![screenshot da resposta no terminal](../bugs/assets/detalhe_cotacao_arredondamento_2.png)

