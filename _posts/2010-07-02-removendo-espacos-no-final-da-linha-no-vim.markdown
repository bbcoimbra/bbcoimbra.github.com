---
layout: post
title: Removendo espaços no final da linha no vim
---
Sempre fico incomodado quando abro um arquivo .rb e verifico que deixei
espaços no final da linha.

Para solucionar isso, escrevi uma função que faz a limpeza e
depois configurei para que ela sempre seja executada quando um arquivo do Ruby
for salvo.

Para isso basta incluir o trecho de código abaixo no arquivo .vimrc:

{% highlight vim %}
function! RemoveTraillingSpaces()
	let cursor_pos = getpos(".")
	%s/[ \t]*$//g
	call setpos(".", cursor_pos)
endfunction

au BufWrite *.rb :call RemoveTraillingSpaces()
{% endhighlight %}
