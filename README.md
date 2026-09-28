## :robot: Teste QA 


### :eight_pointed_black_star: Antes de tudo... 

### Dependências e execução dos testes

Para executar os testes automatizados, é necessário ter o **Node.js v18 ou superior** instalado.

Após clonar o projeto, instale as dependências com:


**npm install**


Meu projeto utiliza o **Newman** para execução automatizada das collections do Postman.

Para executar uma pasta específica da collection:

***npx newman run postman/collection.json -e postman/environment.json --folder nome_da_pasta**

No lugar de "nome_da_pasta", coloque o nome da pasta da collection que gostaria de executar. 


## :eight_pointed_black_star: Objetivo

Este projeto tem como objetivo avaliar a qualidade da nova versão do sistema de **Cotação e Faturamento**, considerando os requisitos existentes, a nova funcionalidade implementada e os impactos gerados pela alteração.

A análise foi realizada por meio de:

* Análise dos requisitos e regras de negócio existentes;
* Análise da nova funcionalidade;
* Testes manuais;
* Testes automatizados;
* Comparação dos resultados entre as versões;
* Análise dos comportamentos esperados e obtidos;
* Identificação, análise e documentação de inconsistências;
* Avaliação dos riscos para a decisão de release.

##  :eight_pointed_black_star: Tecnologias e ferramentas

As principais tecnologias e ferramentas que utilizei foram:

* **Postman** — criação e execução dos testes de API;
* **Newman** — execução automatizada das coleções do Postman via linha de comando;
* **JavaScript** — implementação das validações e scripts dos testes automatizados;
* **Git/GitHub** — versionamento e organização do projeto;
* **Markdown** — documentação dos testes, resultados e decisão de release.


### :eight_pointed_black_star: Testes manuais

Foram realizados testes exploratórios e funcionais para verificar:

* Comportamento das funcionalidades;
* Regras de negócio;
* Cenários positivos e negativos;
* Valores de entrada e saída;
* Tratamento de erros;
* Comportamentos de fronteira;
* Consistência das informações apresentadas na API e na interface.

### :eight_pointed_black_star:  Testes automatizados

Foram criados testes automatizados para validar principalmente o comportamento da API, incluindo:

* Status codes;
* Estrutura e conteúdo das respostas;
* Valores retornados;
* Regras de negócio;
* Validações relacionadas aos dados de cotação e faturamento.

A execução das coleções pode ser realizada utilizando o **Newman**, permitindo a repetição dos cenários e a obtenção de resultados consistentes.

## Observação sobre testes automatizados

Collection dos testes automatizados se encontra na pasta POSTMAN e não em REGRESSAO por uma questão de organização do código.


##  :eight_pointed_black_star: Decisão de Release

A decisão de release foi realizada após a análise conjunta de:

1. Requisitos existentes;
2. Nova funcionalidade - desconto por volume;
3. Resultados dos testes manuais;
4. Resultados dos testes automatizados;
5. Comparação entre as versões;
6. Impacto dos problemas encontrados;
7. Riscos associados a cada bug encontrado

A justificativa detalhada da decisão estão disponíveis no arquivo:

**RELEASE_DECISION.md**


## Considerações finais


Os testes, evidências e conclusões que apresentei neste projeto refletem os cenários executados e os resultados observados durante a avaliação. Naturalmente não houve cobertura completa dos cenários no levantamento e na execução. Levei em conta o tempo disponibilizado, as tecnologias usadas e o que seria mais crítico a ser testado. 

Fico à disposição para esclarecer quaisquer dúvidas sobre os testes realizados, critérios utilizados, automações, resultados ou decisões apresentadas neste projeto.
