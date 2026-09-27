# Estratégia de teste

<!-- Este arquivo é entregável. Preencha as seções abaixo. -->

## Contexto e objetivo da validação

--> A atual validação tem como objetivo trazer embasamento para liberação da nova versão, não trazendo riscos financeiros e de negócio. A principal pergunta é: "Podemos subir a nova versão de forma segura e sem perdas de funcionalidade?"


<!-- Em duas ou três frases: o que está em jogo nesta validação e qual pergunta
     você precisa responder até sexta. -->

## Análise de risco


<!-- Quais áreas do sistema concentram o maior prejuízo se falharem, e por quê.
     Ordene por risco, não por facilidade de teste. -->

| Área | O que pode dar errado | Impacto se acontecer | Probabilidade | Prioridade |
|---|---|---|---|---|
| Engine de cálculo - cotação | Uso de valor incorreto para desconto, base de cálculo por peso ou de multiplicador  | Impacto crítico - pode causar prejuízo financeiro à empresa em caso de cobrança abaixo do correto ou prejuízo ao cliente, gerando possíveis questões jurídicas  |Alta  |1  |
|Contrato da API (entre v1 e v2)|Mudança dos campos em requests ou responses - alteração de campos, de tipos dos campos |Impacto importante - pode causar quebra da integração com a API, impossibilitando o funcionamento correto do sistema|Média|1
|Engine de Cálculo - desconto|Aplicação de desconto a cargas com volumes não elegíveis de acordo com as regras estabelecidas| Impacto crítico - poderá haver cobrança ao cliente abaixo do valor correto, o que acarretaria em perda financeira à empresa |Alta|1
|Engine de cálculo - arredondamento |Ausência de precisão no arredondamento do valor final da cotação |Impacto médio - a curto e médio prazo não haveria risco alto relacionado a perdas à empresa ou aos clientes, tratando-se diferenças dízimas decimais. Porém, seria necessário ajuste para que o acúmulo das diferenças não cause impacto financeiro. |Baixa|2

## Fontes de verdade usadas

--> Realizei a análise detalhada do READ.ME e da Spec da nova funcionalidade, além de uma leitura do Changelog. Embora o changelog seja importante para estabelecer uma base das áreas afetadas, não considerei como pontos imutáveis. É totalmente possível que áreas fora do escopo da nova funcionalidade sejam afetadas e para isso são necessários os testes de regressão. 

<!-- Contra o que você validou cada comportamento: README, especificação,
     changelog, comparação direta v1 × v2. Diga quando as fontes divergiram
     entre si e o que você fez nesse caso. -->

## Abordagem por área

--> Smoke Test - verificação manual das funcionalidades mais críticas da tela de cotação

--> Teste de API - validação de todas as rotas, utilizando Postman, manualmente (sem script)

--> Automação via Postman - utilização de script em javascript (puro) para validação de cpodigo
de response, estrutura da response, tipos de dados 

<!-- Como testou cada área priorizada: exploratório, comparação entre versões,
     leitura de código, automação. E por que essa escolha para essa área. -->

## O que decidi NÃO testar

--> Devido ao relativo curto tempo para análise, criação e execução dos testes, decidi não testar a usabilidade e a experiência do usuário. 
--> Também tomei a decisão de não realizar, no primeiro momento, teste de carga nas rotas de criação de cotação e faturamento. 



| Ficou de fora | Por quê | Risco que estou aceitando |
|---|---|---|
|  |  |  |

## Ambiente e dados

-> Utilizei o VSCode para manejo do código e subir o servidor das duas versões da API, separadamente. Escolhi manejar a tela de cotações no navegador Firefox e não dentro do VsCode, para melhor visualização. 

## Limitações da minha análise

<!-- O que você não conseguiu concluir, e o que precisaria para concluir. -->
