---
layout: post
title: "Dígrafos no vim"
---
Recentemente descobri uma funcionalidade bastante interessante no
[*vim*][vim-link], os dígrafos.

Dígrafos, de origem grega, /di/ "dois" e /grafo/ "escrever", ocorrem,
segundo a [Wikipédia] [digraph-link], quando duas letras são
utilizadas para representar um [fonema] [fonem-link] - alguem lembra das aulas
da quarta série?

No [*vim*][vim-link], os dígrafos são utilizados para que
possamos escrever caracteres não-ASCII, imprimíveis, que
normalmente não aparecem no teclado. Ex: &AElig; &aelig;.

Para fazer uso dos dígrafos, basta pressionar Ctrl-k, no modo de
inserção. No lugar onde o cursor está posicionado
aparecerá um ponto de interrogação, indicando que o
[*vim*] [vim-link] está esperando pela digitação dos dois
caracteres que formam o dígrafo.

Para uma lista completa dos dígrafos disponíveis, em modo de
comando, digite:

    :digraphs

Para mais informações é só dar uma olhada na ajuda
do *vim* com ":help digraphs" ou [aqui] [vim-help-digraphs].

[vim-link]: http://vim.org/ "Vim"
[digraph-link]: http://pt.wikipedia.org/wiki/Dígrafo "Dígrafo"
[fonem-link]: http://pt.wikipedia.org/wiki/Fonema "Fonema"
[vim-help-digraphs]: http://vimdoc.sourceforge.net/htmldoc/digraph.html "Help de dígrafos do Vim"