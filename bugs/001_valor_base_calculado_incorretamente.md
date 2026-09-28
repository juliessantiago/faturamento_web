
## [001]_valor_base_calculado_incorretamente ##

**Severidade**: Alta

**Versão afetada**: v2

**Ambiente**: v1 em http://localhost:3001 (referência), v2 em http://localhost:3002 (com bug). Cotações de teste criadas via POST /api/cotacoes com peso_kg exatamente nos limites de faixa (10, 50, 100), volumes=5, uf_origem=SP, uf_destino=SP

Por que escolhi esses valores, especificamente? -> mesma UF com o objetivo de excluir o multiplicador e 5 volumes para excluir a faixa de desconto, que só se aplica a partir de 10 volumes. Assim pude isolar o cálculo de valor base.

**Passos para criar o teste**

Resetar o estado inicial em ambas as versões: POST /_reset em v1 e em v2.
Criar a mesma cotação em ambas as versões: POST /api/cotacoes Content-Type: application/json

{ "cliente": "Borda 10kg", "peso_kg": 10, "volumes": 5, "uf_origem": "SP", "uf_destino": "SP" }

Consultar GET /api/cotacoes/{id} em ambas as versões, usando o id retornado no passo 2.
Comparar o campo valor_base entre as duas respostas.
Repetir os passos 2–4 para peso_kg = 50 e peso_kg = 100.

**Resultado esperado**

Conforme a tabela de faixas de peso do README, os limites são inclusivos ("até 10kg", "de 10.01kg a 50kg", etc.). Logo:

    peso_kg = 10 → valor_base = 25 (última posição da faixa "até 10kg")
    peso_kg = 50 → valor_base = 60 (última posição da faixa "10.01–50kg")
    peso_kg = 100 → valor_base = 110 (última posição da faixa "50.01–100kg")

**Resultado obtido**

peso_kg 	valor_base esperado 	valor_base obtido na v2 	Faixa aplicada indevidamente
10 	25 	60 	10.01–50kg
50 	60 	110 	50.01–100kg
100 	110 	180 	acima de 100kg

Em todos os três casos, a v1 retornou o valor correto (25, 60 e 110, respectivamente). Valores fora dos limites exatos (0.5, 0.99, 10.01, 50.01, 100.01, 1000kg) calcularam corretamente em ambas as versões — o problema ocorre apenas nos três limites exatos testados.

**Causa provável**

O padrão — sempre a faixa imediatamente seguinte, nunca a anterior, e apenas nos valores exatos de limite — é consistente com um erro de operador de comparação no motor de precificação da v2 (possivelmente src/pricing/v2.js, a confirmar): um > usado onde deveria ser >= (ou < onde deveria ser <=) na definição dos limites de faixa, tornando-os exclusivos em vez de inclusivos.

**Evidência**

Suíte de teste via Postman e Newman, pasta valor_borda_peso

**Impacto**

Como valores de base estão sendo usados "fora da borda",  acima da faixa que deveriam pertencer, o valor das cotações está sendo calculado a mais do que deveria. Tal erro causa uma cobrança indevida aos clientes, tratando-se de um bug CRÍTICO. 

--> Para rodar teste: 

:arrow_forward: **npx newman run postman/collection.json -e postman/environment.json --folder valor_borda_peso**

--> RESULTADO: 

 36 asserções executadas, 6 falhas — concentradas exatamente nos 3 limites de faixa testados (10, 50, 100kg)




