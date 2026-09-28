# Perguntas ao Product Owner

## Perguntas em aberto

<!-- Para cada pergunta, use o bloco abaixo. Copie quantas vezes precisar. -->

__________________________________________________________



### 1. Quantidade de volumes

**Onde apareceu:** 
Tela de cotações 

**O que está ambíguo:**
->  A documentação explica que devemos considerar 
o valor base (de acordo com o peso), o multiplicador 
pela rota e o imposto sobre o valor, não incluindo a 
quantidade de volumes. Porém, a informação volumes 
aparece na tela de cotações. Essa informação é relevante? 
Deve-se compreender que o peso informado já é 
o peso com todos os volumes? 

**O que a v1 faz hoje:**
-

**O que a v2 faz:**
-

**Por que isso importa:**
Há diferença considerável no preço se incluirmos a quantidade
de volumes no cálculo da cotação. 

**Interpretação que adotei enquanto não há resposta:**
Interpretação tomada é que o sistema considera que o peso
pertence ao conjunto de volumes (ex: 20kg, 2 volumes -> cada volume teria 10kg), dada a informação de como 
deve-se realizar o cálculo. 
Como a versão 1 já está em produção e o cálculo está sendo realizado dessa forma, sem informação no read.me, adotou-se
a decisão de desconsiderar a informação dada na tela de 
cotações. 

**Bloqueia o go/no-go?** 
<!-- Sim/Não e Por quê   -->
Não. No momento da escrita dessa dúvida, não havia motivo 
para considerar o comportamento como bug, visto que o 
read.me não leva em consideração a quantidade de volumes. 
---

### 2. Desconto por volume - valor de borda 10 

**Onde apareceu:** 
Versão 2 da API, com a nova funcionalidade

**O que está ambíguo:**
->  No arquivo SPEC-desconto-por-volume há de certa forma uma inconsistência. A tabela para determinar quanto de desconto deve ser dado para cada quantidade de volumes é a seguinte: 

| volumes | desconto que deve ser dado 
|---|---|
| 10 a 19 volumes | 5% (0.05) |  
| 20 a 49 volumes | 10% (0.10) |
| 50 ou mais volumes | 15% (0.15) |  

Porém, logo abaixo da tabela existe a informação: 
*A política vale para pedidos **acima de 10 volumes** e não altera em nada a tabela de faixa de peso nem os multiplicadores de rota, que permanecem exatamente como estão hoje em produção.*

Tendo essas duas informações, ocorreu a dúvida: o desconto de 5% é dado realmente de 10 (incluindo o 10) volumes a 19 ou somente acima de 10? 

**O que a v1 faz hoje:**
No caso da v1 não há influencia desse comportamento.

**O que a v2 faz:**
Na v2, uma carga de exatamente 10 volumes não está recebendo desconto de 5%. 

**Por que isso importa:**
O valor final da cotação de uma carga de 10 volumes é calculado acima do que deveria pois não está sendo cedido o desconto. 

**Interpretação que adotei enquanto não há resposta:**
Para prosseguimento dos testes, adotei a interpretação de que o sistema deve considerar o comportamento exibido na tabela. Ou seja: para 10 volumes, deve-se dar o desconto de 5%. Então, como a v2 erra nesse ponto, considerei que este comportamento é um bug. 

**Bloqueia o go/no-go?** 
<!-- Sim/Não e Por quê   -->
O atual comportamento da v2 nesse quesito influenciou diretamente na decisão do go/no-go, mas não impediu que eu tomasse uma deliberação.  

## Decisões que tomei sem perguntar

<!-- Ambiguidades menores que você resolveu sozinho por não valerem uma ida ao
     PO. Diga qual interpretação adotou e por quê. -->
