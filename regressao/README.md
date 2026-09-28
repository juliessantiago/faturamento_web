# Suíte de regressão

--> :bangbang: Atenção: para melhor organização do projeto, todos os testes estão dentro da pasta POSTMAN. 

## Como rodar

Ferramenta escolhida: Postman + Newman (executor de linha de comando do Postman).

### Por quê escolhi essas ferramentas? 

Optei por testar via API com Postman por já ter experiência da ferramenta, o que me
permitiu focar na cobertura das regras de negócio em vez de gastar tempo
aprendendo uma nova sintaxe de testes. A collection roda de forma idêntica pela
interface do Postman (pra desenvolvimento) e pelo terminal via
Newman (para execução)

### Pré-requisitos
- Node.js 18+ (mesmo do projeto)
- As duas versões do servidor rodando previamente:
  - `node server.js v1` (porta 3001)
  - `node server.js v2` (porta 3002)

### Como rodar

## Pré-condições

### O que está sendo testado

A collection está organizada em pastas: `faixa-de-peso`, `faturamento`,
`validacao-cotacao`, `desconto-volume`. Cada requisição contém as asserções
(`pm.test`) que validam o contrato descrito no README e no SPEC. A mesma
collection é rodada duas vezes — uma por environment (`v1` e `v2`) — para
comparar o comportamento entre as versões.


## Cenários cobertos

| # | Cenário | O que protege | v1 esperado | v2 esperado |
|---|---|---|---|---|
|  |  |  |  |  |

## Como a suíte compara v1 e v2

<!-- Explique o desenho: ela roda os mesmos casos nas duas portas e compara? Ela
     tem valores esperados fixos? De onde saíram esses valores? -->

## O que esta suíte NÃO cobre

<!-- E por quê. -->

## Saída esperada

-->Estrutura da resposta do Newman: 

<!-- Cole a saída da suíte rodando, para quem avaliar saber o que esperar. -->

```

```
