# Matriz de cobertura

## Como ler esta matriz

"Automatizado?":  Sim |  Não (manual) 

"Resultado": Passou |  Falhou |  Falhou parcialmente

"Situação" : Coberto | Coberto com bug encontrado | Não coberto | Bloqueado.

## Risco × cobertura

Antes de fazer a análise da tabela, ler a descrição dos termos usados abaixo:

-> **Tela**: Teste foi realizado diretamente na tela de operação

-> **API**: Teste foi realizado nas rotas da API utilizando ferramenta Postman (Postman foi uma determinação de escolha pessoal)

-> **Não coberto**: Planejamento foi realizado, mas priorizei outros cenários

--> **Níveis de risco**: Para cada cenário testado estabeleci um nível diferente de risco:

:exclamation: *Alto* - ao não passar, pode causar quebra de funcionamento do sistema, acarretando em perda financeira. Necessita análise criteriosa do código para correção

*Médio - ao não passar, representa um erro importante, mas que não necessariamente impacta o financeiro. Pode ser corrigido através de análise das rotas da API

*Baixo*  - ao não passar, representa um erro de baixa importância no presente momento, não impacta diretamente a área de negócios, nem o financeiro. Deve-se considerar a correção em nova versão

| # | Risco/Descrição | Área | Como foi coberto | Automat/Manual | Resultado | Problema aberto |
|---|---|---|---|---|---|---|
| 1 | Alto - valor de base incorreto no cálculo da cotação | V2 - Precificação  | Comparação campo a campo entre v1 e v2 via GET /api/cotacoes/{id}  |  manual |  OK/NOK | bug registrado em... |
|2|Alto - valor de desconto errado no cálculo da cotação  |V2 - Precificação||||
|3|Alto - cotações no sistema alteradas na V2 em relação a V1|Tela de operação||||
|4|Alto - alteração da quantidade de faturas ou valores entre V1 e V2 |Tela de Operação||||

## Cobertura por regra de negócio

| Regra | Fonte | Cenários testados | Situação |
|---|---|---|---|
| Faixa de peso | README ||  |
| Multiplicador de rota | README | | |
| Imposto | README | |  |
| Arredondamento do valor final | README |  |  |
| Fatura única por cotação | README | |  |
| Desconto por volume | SPEC | | |
| Contrato das rotas da API | README |  | |

## Lacunas conhecidas

*** PREENCHER 