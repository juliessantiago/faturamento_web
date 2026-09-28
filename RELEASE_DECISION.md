# Release Decision — Cotação & Faturamento

## 1. Decisão

:no_entry_sign: **Status: NO-GO — Release não aprovada.**

Com base nos testes, na comparação entre a versão atual (v1) e a versão candidata (v2) e na análise dos resultados, a minha recomendação é não realizar a release neste momento.

Foram identificadas divergências nos cálculos de desconto, valor base e arredondamento na v2. Esses problemas apresentam riscos financeiros, podendo gerar cobranças incorretas e inconsistências nos registros e relatórios de faturamento.

## 2. Principais problemas identificados

### 2.1. Divergências no cálculo do desconto e do valor base

Os testes identificaram erros nos cálculos de desconto e no valor base (calculado pelo peso) da v2, resultando em valores finais superiores aos que deveriam ser cobrados dos clientes.

**Impacto:**

* Cobrança indevida 
* Prejuízo financeiro direto ao cliente.
* Possibilidade de reclamações, solicitações de estorno e possivel problema jurídico 
* Risco à confiabilidade do processo de faturamento.

**Severidade: Crítica**

Por envolver valores financeiros cobrados dos clientes, o problema deve ser corrigido e validado antes da disponibilização da nova versão.

### 2.2. Divergências no arredondamento de casas decimais

Também foram observadas diferenças de arredondamento, gerando cobranças com valor reduzido em R$ 0,01 ou R$ 0,02 por fatura.

Embora a diferença individual seja muito pequena, essas pequenas diferenças acumuladas ao longo do dia e ao longo do mês podem gerar discrepância nos relatórios financeiros.

**Exemplo de impacto acumulado:**

Considerando 100 faturas por dia, com uma diferença de R$ 0,02 em cada uma:

* Diferença diária: 100 × R$ 0,02 = **R$ 2,00**
* Diferença em 30 dias: R$ 2,00 × 30 = **R$ 60,00** 

Ou seja: seriam R$ 60,00 cobrados a menos que não irão aparecer no registro de faturas. 


**Impacto:**

* Inconsistência (embora pequena) nos relatórios financeiros.

**Severidade: Média** 


### Observação — Alteração de valores de cotações já faturadas

Durante os testes entre as versões, foi identificado que **29 das 60 cotações já faturadas (48,3%)** apresentam valores diferentes na v2. Tal comportamento está documentado no bug 004. 

Como complemento, foi realizado um teste adicional tentando faturar novamente uma cotação já faturada. A API respondeu corretamente com **HTTP 422**, informando que a cotação já havia sido faturada.

Entretanto, esse teste isolado não permite confirmar se as alterações observadas nas 29 cotações decorreram de um novo faturamento ou de alguma alteração ou recálculo dos valores anteriormente registrados.

**Impacto e risco:** a divergência compromete a confiabilidade dos valores históricos e pode afetar a integridade das informações financeiras e a conciliação dos faturamentos.

**Status:** divergência confirmada; causa ainda não determinada.

**Decisão:** Como não há confirmação se os novos valores geraram novas faturas, decidi que este bug em específico (embora importante) não deve bloquear a release no momento. Deixei registrado no arquivo Perguntas_ao_po.md. 


**Resumo**

| Problema | Severidade | Versão|Impacto medido|Bloqueia|
|---|---|---|---|--
|Erro no cálculo do desconto e do valor base|Crítica|v2|Cobrança superior ao valor devido, gerando prejuízo financeiro ao cliente| Sim
|Divergência no arredondamento de casas decimais|Média|v2|R$ 0,01 a R$ 0,02 a menos por fatura. Em 100 faturas/dia com diferença de R$ 0,02, o impacto estimado é de R$ 60,00 em 30 dias. Afeta minimamente o faturamento da empresa, mas pode quebrar os relatórios financeiros|Sim

## 3. Análise de risco

Os problemas identificados comprometem a confiabilidade dos cálculos financeiros da nova versão.

O risco não está restrito ao valor de uma única cotação ou fatura. A repetição dos erros pode ampliar o impacto financeiro e gerar inconsistências operacionais e contábeis.

A existência de cobranças superiores ao devido representa um risco alto direto aos clientes. Já as diferenças de arredondamento podem causar erros internos nos relatórios financeiros. 

## 4. Condições para aprovação da release

Considero que a release deverá permanecer bloqueada até que sejam atendidas as seguintes condições:

1. Corrigir os erros identificados no cálculo do valor base e dos descontos.
2. Revisar e corrigir a regra de arredondamento, garantindo conformidade com o comportamento especificado.
3. Executar novamente os testes funcionais dos cenários afetados, incluindo valores de fronteira.
4. Comparar os resultados da v2 com os valores esperados e, quando aplicável, com a v1, respeitando as regras específicas de cada versão.
5. Validar a consistência dos valores calculados. 

## 5. Riscos aceitos


| Risco | Por que é aceitável | Como detectaríamos em produção |
|---|---|---|
| Quebra do contrato da API em rotas não consideradas críticas. (Foi priorizada rota de criação de cotação e de faturamento) |  Indiretamente constatou-se a integridade do contrato da API através dos testes automatizados. Se houver um caso específico de algum playload ou response com erro na nova versão, podemos isolar e corrigir com maior facilidade do que os erros apresentados no cálculo da cotação|Haveria quebra na exibição das cotações e no faturamento. Pode haver um pico na taxa de erros 5xx (erros do servidor) ou 4xx (erros nas requisições).  |

## 6. Conclusão

A versão v2 não deve ser liberada no estado atual.

Os erros encontrados afetam diretamente os valores cobrados e a confiabilidade das informações financeiras. A correção desses problemas é requisito para uma nova avaliação da release.

**Decisão final: NO-GO.**

## Acompanhamento

Primeiro,sugiro uma call de alinhamento com o time de desenvolvimento para discutirmos as possíveis causas dentro do código, levantarmos cenários e possíveis soluções. Após as correções terem sido realizadas, será necessária observação atenta aos valores das novas cotações. Qualquer erro residual nos valores, decorrente do desconto ou do valor base, devem ser o gatilho para um rollback. 
