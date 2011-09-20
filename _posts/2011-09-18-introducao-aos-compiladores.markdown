---
layout: post
title: "Introdução aos compiladores"
---
No primeiro semestre de 2011 conclui minha graduação na FATEC-SP. Como TCC,
produzi uma monografia com o título ["Implementação de um Compilador que Gera
uma Representação Gráca do Programa Compilado"][compiler-pdf]. Um resumo pode
ser lido [aqui][abstract-compiler-pdf].

Como o tema de compiladores é extenso e um pouco indigesto, segue um post
para auxiliar e facilitar os estudos daqueles que desejam estudar este ramo
da Ciência da Computação.

Neste artigo escreverei uma breve apresentação e introdução ao tema de
compiladores. Então vamos ao que interessa.

## Compilador

Um compilador é um programa que produz um programa-objeto a partir de um
programa-fonte, ou seja, é um programa "tradutor". O que diferencia um
compilador de um tradutor de linguagens naturais (como o Google Translate),
é que um compilador costuma traduzir um programa-fonte (que é facilmente lido
por um programador) em um programa-objeto (que é facilmente executado pelo
computador) enquanto um tradutor efetua traduções entre linguagens como o
Português e o Inglês.

O processo de compilação é bastante complexo e costuma ser dividido em módulos
para que seja mais facilmente compreendido e implementado. Um esquema
arquitertural é demonstrado na figura abaixo:

![Esquema do Compilador][compiler-arquiteture]
<legend>Fonte: http://www.cs.uaf.edu/~cs631/notes/ </legend>

Segundo este esquema, temos três grupos de módulos: os que compõe o Front-end,
os que compõe o Back-end e outro que interage com todos os outros módulos
(Manipulador de Erros e Tabela de Símbolos).

No Front-end, estão contidos os módulos de análise e produção das estruturas
necessárias para o processo de compilação que são independentes de conhecer
a arquitetura de máquina em que o programa-fonte será executado.

O Back-end utiliza as estruturas fornecidas pelo Front-end para gerar
programas para determinada arquitetura-alvo (por exemplo, binários para x86 e
x86\_64).

O Manipulador de Erros e a Tabela de Símbolos podem ser acessadas de todos os
módulos do Front-end e do Back-end.

### Analisador Léxico

É o módulo responsável por divir o Programa-Fonte em _tokens_, ou seja,
pequena unidades que possuem significado, como números, identificadores (nomes
de variaveis e funções), sequências de caracteres e símbolos de operadores
(\+, \-, \*, /). Por ser o responsável por dividir o programa-fonte, é o único
módulo que efetua operações de leitura sobre os arquivos de entrada e controla
os buffers de forma eficiente, de forma a minimizar o custo de IO no processo
de compilação.

### Analisador Sintático

Este módulo verifica se a sequência de _tokens_ reconhecido pelo Analisador
Léxico é coerente para a Gramática (Linguagem) definida. Caso contrário, os
erros encontrados devem ser reportados. Além disso, o Analisador Sintático
produz uma Árvore Sintática que representa de forma estruturada a gramática
da linguagem e sequência de execução das instruções do programa-fonte.

### Analisador Semântico

O Analisador Semântico é o responsável, entre outras coisas, por efetuar a
inferência de tipos, garantir que uma variável tenha sido declarada antes de
sua utilização, caso a definição da gramática exija isso. Nessa fase da
compilação, são feitas anotações na Árvore Sintática para facilitar a execução
dos passos posteriores.

### Gerador de código

Este é o módulo responsável por utilizar todas as informações coletadas pelos
módulos anteriores e gerar o programa-objeto. Usualmente, o gerador de código
gera um programa em linguagem assembly para a arquitetura alvo para,
posteriormente, ser montada com um assembler da respectiva arquitetura que
gerará o programa-executável.

## Exemplo do processo completo

Para facilitar o entendimento daquilo que foi lido até agora, vou utilizar um
exemplo. Dado que tenhamos a expreesão numérica a seguir:

	resultado = 42.5 + 13;

O Analisador Léxico produziria os seguintes _tokens_ 

	<identificador, resultado> <=> <numero, 42.5>
	<+> <numero, 13> <;>

com essa sequência de *tokens*, o Analisador Sintático gerará uma Árvore
Sintática parecida com a que temos a seguir:

<pre>
          =
        /   \
      id     +
           /   \
        42.5    13
</pre>

em que **id** é o identificador (variável) "resultado".

O Analisador Semântico é o responsável por ideintificar que um dos operandos
é um número de ponto flutuante e efeturar as conversões apropriadas, pois,
usualmente, os processadores utilizam operações distintas para números
inteiros e números de ponto flutuante.

Na Árvore representada acima, o número 13 deve ser convertido (operação de
coerção) para um número de ponto flutuante (13.0) antes da operação ser
executada. Outra verificação que deve ser efetuada é se o a variável
"resultado" tem tamanho suficiente para armazenar o resultado.

Por fim, o Gerador de Código precisa gerar o código assembly conforme abaixo.
Nota, não testei o código abaixo:

	ldf  eax, 42.5
	ldf  ebx, 13.0
	fadd eax, ebx
	ld   ebx, resultado
	stf  [ebx], eax

Primeiramente, os números são carregados para os registradores do processador,
em seguida, os valores são somados e armazenados no registrador *eax*,
endereço da varável *resultado* é armazenado em *ebx* e, por fim, o resultado
da soma é armazenado no na variável *resultado*.

Por hora é isso.

PS.: Assim que possível, incluo mais referências.
 

[compiler-arquiteture]: http://www.cs.uaf.edu/~cs631/notes/phases.gif "Arquitetura de um compilador"
[compiler-pdf]: /static/coimbra-compilers-2011.pdf "Implementação de um Compilador"
[abstract-compiler-pdf]: /static/coimbra-sict-2011-compilador.pdf "Resumo Implementação de um Compilador"
