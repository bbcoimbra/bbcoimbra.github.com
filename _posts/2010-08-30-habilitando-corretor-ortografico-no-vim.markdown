---
layout: post
title: "Habilitando o corretor ortográfico em pt-BR no vim"
---
Neste semestre vou terminar meu curso de Graduação na [FATEC-SP][fatec-link].
Como todo bom nerd, vou escrever minha monografia no [Vim][vim-link],
utilizando o [LaTeX][latex-link] para formatação e geração dos pdfs.

Para facilitar a correção de erros ortográficos, o [Vim][vim-link] incluiu,
a partir da versão 7.x, um corretor ortográfico interno. Para habilitá-lo,
basta incluir os comandos abaixo no arquivo ~/.vimrc:

    :set spell
    :set spelllang=pt-BR

Mas neste momento, provavelmente não temos o arquivo de dicionário que o vim
necessita para fazer as verificações, portanto precisamos obter uma lista de
palavras e em seguida gerar o arquivo do dicionário para o Vim.

Primeiramente, vamos obter a lista de palavras.

A lista de palavras que eu uso, foi obtida do projeto [Aspell][aspell-link].
O Aspell é o corretor ortográfico que substituiu o Ispell. Mais informações
sobre ele podem ser obtidas [aqui][wiki-aspell].

Para obter os dicionários em pt-BR basta instalar o pacote aspell-pt-br, no
Debian (no Ubuntu o pacote deve ter o mesmo nome). Depois de ter instalado o
pacote com

    sudo apt-get install aspell-pt-br

obtemos a lista de palavras executando o comando abaixo:

    aspell -l pt-BR dump master > lista_de_palavras.txt

com a lista de palavras disponível, precisamos gerar o arquivo de dicionário
do Vim. Para isso, basta abrir o Vim e digitar:

    :mkspell pt-BR lista_de_palavras.txt

O comando acima gera um arquivo chamado pt-BR.latin1.spl no meu caso, pois
meu sistema utiliza o encoding Latin1. Agora basta mover o arquivo .spl para
o diretório ~/.vim/spell/. E teremos o corretor ortográfico assim que
o próximo arquivo for aberto.

Para aprender os atalhos do corretor ortográfico e obter mais informações,
digite o comando abaixo no modo de comando no [Vim][vim-link]:

    :help spell

Por hora é só.

[vim-link]: http://vim.org/ "Vim"
[wiki-aspell]: http://en.wikipedia.org/wiki/GNU_Aspell "Wikipedia GNU/Aspell"
[aspell-link]: http://aspell.net/ "GNU/Aspell"
[latex-link]: http://www.latex-project.org/ "LaTeX"
[fatec-link]: http://fatecsp.br "FATEC-SP"
