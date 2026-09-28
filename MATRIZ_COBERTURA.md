# Matriz de cobertura

## Como ler esta matriz

**Automatizado?** ✅ Sim (Postman + Newman, roda com um comando) | ❌ Não (manual) 

**Resultado:** ✅ Passou | ❌ Falhou | ⚠️ Divergência a definir com o PO

**Canal:** API ou via tela


**Risco:** nível (Alto, Médio ou Baixo) e o risco em si. Considerei como risco mudanças na cobrança ao cliente e quebra nas regras de negócio 

**Situação (tabela de regras):** Coberto | Coberto com bug | Coberto parcialmente | Não coberto

## Risco × cobertura

| # | Risco | Área | Como foi coberto | Automatizado? | Resultado | Problema aberto |
|---|---|---|---|---|---|---|
| 1 | **Alto** — valor_base calculado com a faixa errada nos limites exatos de peso (10, 50 e 100 kg) | Faixa de peso | Cotações criadas em v1 e v2 com pesos 0.5, 0.99, 10, 10.01, 50, 50.01, 100, 100.01 e 1000 kg (5 volumes, mesma UF). Compara valor_base com a tabela do README e v1 × v2. |  Sim (valor_borda_peso) | ❌ Falhou: 6 de 36 asserções, todas em 10, 50 e 100 kg| [BUG-001] (pasta bugs)
| 2 | **Alto** — desconto por volume aplicado com a faixa anterior nos limites exatos (10, 20 e 50 volumes) | Desconto por volume | Cotações na v2 com 9, 10, 19, 20, 49, 50 e 51 volumes (peso 50.5 kg, mesma UF, sem interferência do BUG-001). Compara desconto e valor_total com a tabela da SPEC | Sim (valor_borda_desconto)| ❌ Falhou: 6 de 14 asserções, em 10, 20 e 50 volumes | [BUG-002] (pasta bugs) |
| 3 | **Médio** — valor_total com desconto truncado em vez de arredondado pela regra comercial | Arredondamento | 9 casos com terceira casa decimal (7 devem subir, 2 devem cair), pesos e volumes fora das bordas dos BUG-001 e 002. Cada caso verifica antes valor_base, multiplicador e desconto (premissas) | Sim (arredondamento_desconto) | ❌ Falhou: 7 dos 9 casos (os 2 que devem cair passaram) | [BUG-003](pasta bugs)|
| 4 | **Baixo** — valor_total sem duas casas decimais na resposta da API (28 em vez de 28.00) | Arredondamento (formato) | Texto cru da resposta em 2 casos com total terminando em zero (5 kg SP→SP, 5 e 11 volumes) | Sim (valor_borda_desconto) e testes manuais|Passou. A API devolve número no JSON (28 e 26.6)mas na tela o valor é exibido corretamente|  -  |
| 5 | **Médio** — regressão no multiplicador de rota | Multiplicador de rota | Multiplicadores 1.0 (SP→SP), 1.4 (SP→MG) e 1.9 (RS→SP) conferidos na v2 como premissa dos testes de arredondamento; comparação v1 × v2 na cotação 10 (RS→SP).| Sim |  Passou  | — |
| 6 | **Médio** — regressão no imposto (1,12) | Imposto | Cobertura indireta: todo valor_total esperado incorpora o imposto (ex.: 25 × 1,0 × 1,12 = 28,00; 110 × 1,0 × 1,12 = 123,20) | sim (indireto) | Passou: valores sem arredondamento envolvido (28,00, 123,20, 84,67, 325,58) batem | — |
| 7 | **Alto** — cotação faturada mais de uma vez / erro no faturamento | Faturamento |  | Teste manual no Postman | Passou | ----|
| 8 | **Médio** — contrato da API quebrado (campos, status, validações, paginação) | Contrato da API | Foi coberta a rota mais crítica: POST /api/cotacoes  |Todos cenários criados foram cobertos  - Consultar arquivo README da pasta REGRESSÃO | Todos cenários passaram | — |

## Cobertura por regra de negócio

| Regra | Fonte | Cenários testados | Situação |
|---|---|---|---|
| Faixa de peso | README | Bordas 0.5, 0.99, 10, 10.01, 50, 50.01, 100, 100.01 e 1000 kg em v1 e v2; pesos 5, 34, 50.5 e 120 kg nos demais testes | Coberto com bug (BUG-001) |
| Multiplicador de rota | README | SP→SP (1.0), SP→MG (1.4) e RS→SP (1.9) na v2; v1 × v2 comparado em RS→SP (cotação 10) | Coberto parcialmente (sem falhas; rota para o Nordeste e comparação v1 × v2 nas rotas 1.0 e 1.4 não executadas) |
| Imposto | README | valor_total conferido em cenários com e sem desconto | Coberto (indiretamente, sem falhas) |
| Arredondamento do valor final | README | 9 casos com terceira casa decimal (2 caem, 7 sobem) e 2 casos de formato com total terminando em zero | Coberto com bug (BUG-003) |
| Fatura única por cotação | README | -----| ----- |
| Desconto por volume | SPEC | Bordas 9, 10, 19, 20, 49, 50 e 51 volumes; percentuais em 11, 12, 15, 25 e 60 volumes; 5 e 9 volumes com desconto 0 | Coberto com bug (BUG-002; BUG-003 no valor final). |
| Contrato das rotas da API | README | Request: Campos ausentes, campos inválidos. Response: estrutura e tipo dos dados | Coberto parcialmente (POST /api/cotacoes) Consultar README da pasta REGRESSÃO|


## Lacunas conhecidas

- **Interpretação de 10 volumes.** Houve uma leve ambiguidade(tabela: "10 a 19 → 5%"; texto: "acima de 10 volumes"). Foi tratado como bug (BUG-002) seguindo a tabela e registrado em PERGUNTAS_AO_PO.md.

