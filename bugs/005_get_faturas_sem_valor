# [005]  Rota  GET /api/faturas não retorna valor da cotação (fatura) 

--> **Erro ocorre para cotações da carga inicial, em ambas as versões**

**Severidade:** Média

**Versão afetada:** ambas (v1 e v2)

**Ambiente:** v1 em http://localhost:3001 e v2 em http://localhost:3002, ambas com a carga inicial (60 faturas, correspondentes às cotações de 1 a 60). 

Para comparação, uma cotação criada e faturada durante o teste (id 201, 34 kg, RS→SP, 11 volumes).

## Passos para reproduzir o erro

1. Resetar o estado inicial nas duas versões: POST /_reset em v1 e em v2.
2. Consultar a fatura de uma cotação da carga inicial, em cada versão:

   GET /api/faturas?id_cotacao=10

3. Observar os campos do objeto retornado.
4. Para comparar, criar uma cotação nova (POST /api/cotacoes), fatura-la (POST /api/cotacoes/{id}/faturar) e consultar GET /api/faturas?id_cotacao={id}.
5. Consultar GET /api/faturas sem filtro e verificar quais faturas trazem o campo "valor".

## Resultado esperado 

Na descrição de "Contrato da API": a fatura é descrita com os campos `id`, `id_cotacao`, `cliente`, `valor` e `emitida_em`, e `GET /api/faturas` responde com um array de faturas. O README também diz que a fatura é emitida pelo valor final vigente da cotação. Entende-se que o valor final fica registrado.

Observação de leitura: o README mostra o formato completo da fatura junto da rota de emissão (`POST .../faturar`), e a listagem é descrita apenas como "array de faturas". Considerei que a listagem deve trazer o mesmo formato, porém, deixei registrado no arquivo Perguntas_ao_PO.md 

## Resultado obtido

Nas duas versões, as faturas da carga inicial voltam **sem o campo `valor`**:

```
GET /api/faturas?id_cotacao=10 (v1 e v2, idêntico)
[ { "id": 10, "id_cotacao": 10, "cliente": "Indústria Horizonte", "emitida_em": "2026-06-11" } ]
```

Já a fatura emitida durante o teste traz o campo, tanto na resposta do `POST .../faturar` quanto na listagem:

```
GET /api/faturas?id_cotacao=201 (v1)
[ { "id": 61, "id_cotacao": 201, "cliente": "Teste Faturamento", "valor": 127.68, "emitida_em": "2026-09-28" } ]

GET /api/faturas?id_cotacao=201 (v2)
[ { "id": 61, "id_cotacao": 201, "cliente": "Teste Faturamento", "valor": 121.29, "emitida_em": "2026-09-28" } ]
```

Na listagem sem filtro (`GET /api/faturas`), somente a fatura nova (id 61) retornou o campo `valor`.

## Causa provável

Como a mesma rota devolve o valor para a cotação (fatura) nova, o problema não está na listagem em si e sim nos dados da carga inicial. Minha sugestão é a verificar a geração da carga inicial de faturas. 

## Impacto

**60 de 60** faturas da carga inicial não trazem o valor faturado, nas duas versões.


## Evidência

```
GET /api/faturas?id_cotacao=10
v1: [ { "id": 10, "id_cotacao": 10, "cliente": "Indústria Horizonte", "emitida_em": "2026-06-11" } ]
v2: [ { "id": 10, "id_cotacao": 10, "cliente": "Indústria Horizonte", "emitida_em": "2026-06-11" } ]

POST /api/cotacoes/201/faturar (fatura nova)
v1: { "id": 61, "id_cotacao": 201, "cliente": "Teste Faturamento", "valor": 127.68, "emitida_em": "2026-09-28" }
v2: { "id": 61, "id_cotacao": 201, "cliente": "Teste Faturamento", "valor": 121.29, "emitida_em": "2026-09-28" }
```