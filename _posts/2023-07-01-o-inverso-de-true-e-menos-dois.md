---
date: 2023-07-01
layout: post
title: O inverso de True é -2 e eu posso provar!
description: "Vamos analisar um bug curioso em que ao inverter um DataFrame de booleanos obtivemos o valor -2, sob a ótica de como booleanos e valores inteiros são armazenados e como o Python lida com números negativos."
image: https://matheusjorge.github.io/assets/img/posts/post9_quem_bate_sou_eu.jpg
category: 'Artigo'
math: true
tags:
  - binário
  - complemento de dois
  - lógica booleana
  - bitwise
author: matheusjorge
---

# O inverso de True é -2 e eu posso provar

Recentemente eu estava trabalhando com um dataframe do pandas que tinha uma coluna booleana (verdadeiro ou falso). Sempre foi tranquilo trabalhar com os tipos nativos do Python e eu nunca tive problema ... até que eu precisei inverter essa coluna. Como vocês podem ver na imagem abaixo, quando eu neguei a minha coluna com valores booleanos eu obtive uma lista cheia de -2!!!!

<details>
    <summary>Código - Inverter booleanos</summary>
    
{% highlight python%}
import pandas as pd

data = pd.Series([True for _ in range(5)], name="true_column").astype(object)
~data
{% endhighlight %}
</details>

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_pandas_object@2x.jpg" width="500px" height="500px"/> 
</center>

Se você esperava que essa operação retornasse uma lista preenchida com falsos, saiba que você não está sozinho. Imagine minha surpresa ao ver isso e tentar criar algum sentido na minha mente. Eu prometo que vou explicar exatamente o que aconteceu e o porquê o pandas está certo e o resto da humanidade errada, mas antes disso eu quero só comentar um pequeno detalhe que faz esse erro aparecer.

Quem abriu o código acima percebeu que eu criei os dados fazendo com que o tipo de dado associado à lista de booleanos é `object`. É esse detalhe que faz com que o resultado final seja -2. Se eu tivesse criado os dados com o tipo `bool` associado, o resultado seria o experado.

<details>
    <summary>Código - Inverter booleanos do jeito certo</summary>
    
{% highlight python%}
import pandas as pd

data = pd.Series([True for _ in range(5)], name="true_column").astype(bool)
~data
{% endhighlight %}
</details>

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_pandas_bool@2x.jpg" width="500px" height="500px"/> 
</center>


## Operador lógico vs bitwise

Agora que você já sabe como resolver esse problema se por acaso encontrá-lo, vamos investigar o motivo de a operação "não ter dado certo" com o primeiro código. A primeira coisa a perceber é o operador que usamos para realizar a negação: `~`. Esse operador é diferente do usual `not`. Enquanto o `not` realiza uma inversão lógica do valor, o `~` realiza o que chamamos de negação *bitwise* ou algo como bit-a-bit, no bom português. Mas o que isso significa na prática?

### Isso é verdade, e aquilo também
Vamos começar com um exemplo simples entendendo o que o Python considera como positivo e negativo. Nós sabemos que existem os valores `True` e `False` para representar valores booleanos, mas o Python também consegue realizar operações lógicas utilizando outros tipos de dados como `int`, `float` e até listas. Por exemplo, `bool(0)` retornará `False`, enquanto `bool(1)` retornará `True`. Isso significa que podemos usar 0 ou 1 no lugar de `False` ou `True`, respectivamente para fazer as operações lógicas.

<div class="code">
{% highlight python%}
if 0:
    print("Isso é falso")
    
if 1:
    print("Isso é verdade")
{% endhighlight %}
</div>

Mas o 1 não é o único valor que é considerado verdadeiro. Na verdade o único inteiro considerado falso é o 0, todos os outros são considerado verdadeiros. Por exemplo `bool(10)` retornará `True`.

Isso também significa que podemos utilizar outros operadores lógicos com os números inteiros, como por exemplo o `not`. Se 0 é falso e 1 é verdadeiro, então `not 0` é `True` e `not 1` é `False`.

<div class="code">
{% highlight python%}
if not 0:
    print("Isso é verdade")
    
if not 1:
    print("Isso é falso")
{% endhighlight %}
</div>

### Representação binária
Apesar de lógicamente os valores 10 e 1 representarem `True`, a representação binária deles é bem diferente. Só uma pequena revisão em representação binária: tudo que é armazenado e executado dentro de um computador é representado por uma sequência de 0s e 1s que chamamos de representação binária. Fica mais fácil entender quando estamos falando de inteiros. Vamos tomar o exemplo do 10, que representado em binário tem a seguinte forma:

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_bin_code_10@2x.jpg" width="600px" height="200px"/> 
</center>

Calma que se você não entendeu nada aqui vai um pequeno resumo:

- Cada 0 e 1 é chamado de bit;
- Um conjunto de 8 bits (como nesse caso) é chamado de byte;
- À direita temos o que chamamos de bit menos significativo (LSP, do inglês *Least Significant Bit*) e à esquerda temos o bit mais significativo (MSP, do inglês *Most Significant Bit*);
- Cada bit representa um valor de potência de 2: começando mo LSP, que representa $2^0$ até o MSP que, nesse caso de 8 bits, representa $2^7$;
- O valor em decimal dessa representação é feita multiplicando o valor de cada bit pela potência que ele representa e somando tudo no final:
<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_bin_10_dark@2x.jpg" width="700px" height="300px"/> 
</center>
    
O valor 1, por exemplo, seria 00000001. Perceba que apesar de represenções diferentes, o Python trata os dois números como `True`. Mas qual a representação binária do valor `True`. Assim como os inteiros, os valores booleanos também são armazenados como uma sequência de bits. Na verdade, dependendo da implementação, um valor booleano pode ser apena 1 bit. Como temos apenas 2 estados (verdadeiro e falso), podemos armazenar um deles como sendo um estado e outro como sendo outro. Em Python, o tipo de dados `bool` é um subtipo de `int`, então o esperado é que uma variável inteira e uma variável booleana tenham o mesmo tamanho. Se nesse caso assumirmos que todos os nosso *int*s tem 8 bits, as representações dos valores `True` e `False` seriam

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_bin_code_true_false@2x.jpg" width="700px" height="300px"/> 
</center>

### Diferença na representação da negação *bitwise* vs da negação lógica
Vamos comparar agora o que acontece quando utilizamos a negação lógica contra a negação *bitwise*. 

Na negação lógica só temos duas transições possíveis:

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_bool_state@2x.jpg" width="700px" height="300px"/> 
</center>

Então sempre que negamos um valor que o Python entende como `True` ele vai retornar `False`, indepedente do valor que estamos negando. O que acontece na verdade é uma conversão implícita do Python do objeto que queremos negar para um booleano antes de aplicar a operação. Algo como:

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_not_bool_int@2x.jpg" width="700px" height="300px"/> 
</center>

Já na negação *bitwise* temos dois estados possíveis para cada bit e a inversão é feita bit a bit. Ou seja, quando fazemos `~10`, o resultado que obtemos é 11110101, que é um valor diferente da representação binária de `False` (00000000), por exemplo.

### Voltando ao pandas
Certo, agora que entendemos o que acontece com cada tipo de operação, podemos tentar entender o que o pandas está fazendo. Perceba que nossos dados originais eram do tipo `object`. Quando pedimos ao pandas para negar esse vetor de dados, ele não tem conhecimento de que se trata de um tipo de dados booleano e que a operação que deve ser feita na verdade é a negação lógica. Ele simplesmente entende que estamos tentando inverter cada bit do nossos dados. Já quando dizemos explicitamente que estamos lidando com dados booleanos ele consegue entender que a operação a ser aplicada deve ser a negação lógica e não a *bitwise*.

Voilá! Já entendemos tudo que aconteceu e podemos dormir tranquilo, certo? Humm ... quase, mais ainda não! O leitor mais atento deve ter tentado imaginar o que seria a inversão *bitwise* de `True` e pode ter chegado em uma conclusão um pouco estranha. Dissemos que a representação binária de `True` é 00000001, então é lógico inferir que a negação *bitwise* the `True` (`~True`) tem representação 11111110. Se aplicarmos a mesma lógica de representação de binários que foi apresentada antes esse valor deveria ser equivalente a 254!!! Mas porque então, quando invertemos inicialmente, obtivemos o valor -2??

<center> 
    <img src="{{ site.baseurl }}/assets/img/uploads/neg_bin_254_dark@2x.jpg" width="700px" height="300px"/> 
</center>


## 254 é igual a -2 ... pelo menos em binário ... e para o Python
Antes de prosseguir eu queria fazer uma pergunta: da forma como foi apresentada a representação binária, temos sempre uma soma de números positivos, então sempre teremos no final um número positivo sendo representado. Como então fazemos para representar números negativos? 

A resposta para essa pergunta é complexa e pode ter muitas soluções (que não são escopos desse artigo), mas vamos focar em como o Python lida com isso. O modo como a linguagem lida com números negativos é através de uma representação conhecida como **complemento de dois** (*two's complement*, em inglês). Não vou discutir o porquê do nome, mas a ideia dele é relativamente simples:

$$ -x = \  \sim x + 1 $$

A equação acima nos diz que para gerar o negativo de um número devemos primeiro fazer uma inversão *bitwise* na representação binária desse número e em seguida somar 1 (ou 00000001) ao resultado final. Então se quisermos o valor -10, o passo a passo seria

$$-10 = ~10 + 1 = ~(00001010) + 1 = 11110101 + 00000001 = 11110110$$


Indo um pouco mais além, podemos manipular a equação de representação para chegarmos a 

$$ -(x+1) = \ \sim x$$

E estamos chegando cada vez mais perto de entender o motivo de `~True = -2`! Se lembrarmos, `bin(True) = bin(1)`, então quando invertemos fazemos `~True` é a mesma coisa que fazer `~1` que, pela equação acima é justamente -2!!! Wow, finalmente chegamos ao real motivo de toda a nossa confusão!

<center>
    <iframe src="https://giphy.com/embed/iVtxoVJyHC5FEfzWzp" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
    <p><a href="https://giphy.com/gifs/judgejudy-iVtxoVJyHC5FEfzWzp">via GIPHY</a></p>
</center>

Uma última dúvida que você deve estar na sua cabeça é como eu diferencio se um número é positivo ou negativo, se eles podem ter a mesma representação. A resposta pra essa pergunta vem do tipo de dados declarado. Por padrão, os inteiros em Python são o que chamamos de **número inteiro com sinal**, e seguem a lógica de representação que mostramos acima. Mas algumas bibliotecas como o numpy, permitem que você defina estruturas com **números inteiros sem sinal** e, nesse caso, a representação segue aquela primeira ideia apresentada, na qual só conseguimos obter números positivos.

# Conclusão
Por hoje é isso, pessoal! Eu trouxe esse post como uma motivação para entendermos mais como realmente os comandos que executamos funcionam por baixo dos panos.
Imagino que quando vocês leram o título e a primeira seção ficaram tão confusos quanto eu, mas a verdade é que sempre existe uma explicação para aquelas pessoas curiosas o suficiente para procurá-la. 
Hoje pudemos ver o quanto isso é importante: apesar de serem muito similares, a operação *bitwise* não é a mesma coisa que a operação lógica e pode gerar alguns comportamentos inesperados no seu programa, então fique atento a esses detalhes!
Espero que tenham gostado da explicação e eu vou procurar trazer mais conceitos ligados ao mundo da programação em si como algoritmos e estruturas de dados, arquitetura e desenvolvimento de software e muito mais, além dos tradicionais conteúdos sobre inteligência artifical. Fique ligado!

