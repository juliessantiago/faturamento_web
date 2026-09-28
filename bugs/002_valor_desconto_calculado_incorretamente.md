
[002]_valor_desconto_calculado_incorretamente

**Severidade:** Alta

**Versão afetada:** v2

**Ambiente:** v2 em http://localhost:3002

Cotações criadas via POST /api/cotacoes

* peso_kg=50.5 (faixa 50.01–100kg, valor_base=110, sem interferência do BUG-001),
uf_origem=SP e uf_destino=SP (multiplicador 1.0), variando apenas o volume.

Cálculo: 110 × 1.0 × 1.12 = 123.20.

## Passos para reproduzir

1. Criar uma cotação na v2 com o payload abaixo, alterando `volumes` para 9, 10, 19, 20, 49, 50 e 51 (uma cotação por valor):

POST http://localhost:3002/api/cotacoes
Content-Type: application/json

{ "cliente": "Borda volume 10", "peso_kg": 50.5, "volumes": 10, "uf_origem": "SP", "uf_destino": "SP" }

2. Ler os campos `desconto` e `valor_total` da resposta de cada criação.
3. Comparar com a tabela de desconto da especificação (seção 2).

## Resultado esperado 

Conforme a tabela na SPEC-desconto-por-volume.md: 10 a 19 volumes → 5%; 20 a 49 → 10%; 50 ou mais → 15%. 

| volumes | desconto esperado | valor_total esperado |
|---|---|---|
| 10 | 0.05 | 117.04 |
| 20 | 0.10 | 110.88 |
| 50 | 0.15 | 104.72 |

Nota sobre o caso de 10 volumes: há uma contradição (de acordo com a minha análise) e a mesma está registrada no arquivo perguntas_ao_PO.md. Decidi adotar a tabela como fonte de verdade, por ser a regra explícita e por estar alinhada ao critério de que os demais limites (20 e 50) são inclusivos. 

## Resultado obtido

| volumes | desconto obtido | valor_total obtido | Faixa aplicada indevidamente |
|---|---|---|---|
| 9 | 0 | 123.20 | ✅ correto |
| **10** | **0** | **123.20** | faixa anterior (sem desconto) |
| 19 | 0.05 | 117.04 | ✅ correto |
| **20** | **0.05** | **117.04** | faixa anterior (5%) |
| 49 | 0.10 | 110.88 | ✅ correto |
| **50** | **0.10** | **110.88** | faixa anterior (10%) |
| 51 | 0.15 | 104.72 | ✅ correto |

Os valores fora dos limites inferiores de faixa (9, 19, 49 e 51) calculam corretamente. O erro ocorre somente em 10, 20 e 50 volumes, isto é, no primeiro valor de cada faixa de desconto.

## Causa provável

O padrão (sempre a faixa imediatamente abaixo, e somente no valor exato do limite inferior)pode ser explicado por um erro de operador de comparação no cálculo do desconto da v2: `volumes > 10`, `volumes > 20` e `volumes > 50` no lugar de `>=`, tornando os limites inferiores exclusivos.


## Impacto

Impacto muito grande em relação a confiabilidade da empresa. Mais uma vez houve um erro resultando em cobrança a mais para o cliente. O cliente que fecha exatamente 10, 20 ou 50 volumes recebe um desconto menor que o da tabela, ou nenhum, e paga mais do que deveria.

## Evidência

Suíte automatizada via Postman + Newman, pasta **valor_borda_desconto**:


