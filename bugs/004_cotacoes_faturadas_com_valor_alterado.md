# [004] Cotação já faturada exibe valor recalculado 

**Severidade:** Alta

**Versão afetada:** v2

**Ambiente:** v1 em http://localhost:3001  e v2 em http://localhost:3002, ambas com a carga inicial (após resetar). Cotação usada: id 10, da carga inicial, que já nasce faturada (as cotações de 1 a 60 são faturadas na carga inicial, conforme explica o README): Indústria Horizonte, 34 kg, RS→SP, 11 volumes.

## Passos para reproduzir

1. Resetar o estado inicial nas duas versões: `POST /_reset` em v1 e em v2.
2. Consultar a cotação 10 em cada versão:
```
   GET http://localhost:3001/api/cotacoes/10
   GET http://localhost:3002/api/cotacoes/10
```
3. Comparar os campos faturada, desconto e valor_total das duas respostas.
4. Consultar a fatura da cotação: **GET /api/faturas?id_cotacao=10** (nas duas versões).

## Resultado esperado (e a fonte: README, spec ou changelog)

SPEC-desconto-por-volume.md, seção 5: "A política não é retroativa. Cotações já faturadas mantêm o valor pelo qual foram faturadas; não há recálculo nem nota de ajuste." A cotação 10 já estava faturada quando a política de desconto foi criada, então deveria manter o valor da v1: desconto: 0 e valor_total: 127.68 (60 × 1,9 × 1,12).

Limitação: o valor efetuvo da fatura não pôde ser lido na API, porque as faturas da carga inicial estão vindo sem o campo valor (ver BUG-005).

## Resultado obtido

| Campo | v1 | v2 |
|---|---|---|
| faturada | true | true |
| desconto | 0 | **0.05** |
| valor_total | 127.68 | **121.29** |

A v2 aplicou 5% de desconto (11 volumes) a **uma cotação já faturada**, alterando o valor exibido.

Já a fatura da cotação 10 é idêntica nas duas versões **sem o campo valor**, então não é possível afirmar pela API se o valor faturado também mudou. O que está confirmado é que a cotação, já faturada, passou a exibir outro valor.

## Causa provável

A v2 parece recalcular o `valor_total` a partir dos dados da cotação (peso, volumes e rota) sem considerar se a cotação já foi faturada, aplicando o desconto por volume a toda a base. A SPEC pede que o desconto valha apenas para cotações novas. Não li o código-fonte, então isto é hipótese; a verificação sugerida é o cálculo do valor na leitura da cotação (provavelmente `src/cotacoes.js`) e o motor `src/pricing/v2.js`, para ver se existe alguma checagem de `faturada` antes de aplicar o desconto.

## Impacto

--> 29 das 60 cotações faturadas da carga inicial tiveram o valor alterado na v2 em relação à v1. Porém, devo deixar esclarecido que essa alteração de valor pode ter vindo do desconto em si, do erro de faixa de peso ou do arredondamento. A soma (líquida) das diferenças foi de +R$ 333,02, com aumentos e reduções. 

## Evidência

```
GET /api/cotacoes/10 (v1)
{ "id": 10, "cliente": "Indústria Horizonte", "peso_kg": 34, "volumes": 11,
  "uf_origem": "RS", "uf_destino": "SP", "faturada": true, "criada_em": "2026-06-11",
  "valor_base": 60, "multiplicador": 1.9, "desconto": 0, "valor_total": 127.68 }

GET /api/cotacoes/10 (v2)
{ "id": 10, "cliente": "Indústria Horizonte", "peso_kg": 34, "volumes": 11,
  "uf_origem": "RS", "uf_destino": "SP", "faturada": true, "criada_em": "2026-06-11",
  "valor_base": 60, "multiplicador": 1.9, "desconto": 0.05, "valor_total": 121.29 }

GET /api/faturas?id_cotacao=10 (v1 e v2, idêntico)
[ { "id": 10, "id_cotacao": 10, "cliente": "Indústria Horizonte", "emitida_em": "2026-06-11" } ]
```