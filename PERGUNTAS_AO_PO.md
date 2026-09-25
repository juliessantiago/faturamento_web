# Perguntas ao Product Owner

## Perguntas em aberto

<!-- Para cada pergunta, use o bloco abaixo. Copie quantas vezes precisar. -->

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

## Decisões que tomei sem perguntar

<!-- Ambiguidades menores que você resolveu sozinho por não valerem uma ida ao
     PO. Diga qual interpretação adotou e por quê. -->
