---
date: 2023-07-20
layout: post
title: Duck Typing
description: "Neste artigo vamos conhecer o conceito de tipagem de pato utilizada no Python e ver algumas alternativas quando precisamos de um pouco mais de controle na utilização dos tipos de dados"
image: https://matheusjorge.github.io/assets/img/posts/post11_duck_typing.jpg
category: 'Artigo'
math: true
tags:
  - python
  - duck typing
  - tipagem
  - protocolo
  - tipagem nominal
  - tipagem estrutural
author: matheusjorge
---

## Duck Typing

Se você programa em Python, com certeza já ouviu falar que ela é uma linguagem não tipada. Mas você entendeu o que isso significa e quais são as implicações?
No artigo de hoje a ideia é apresentar a filosofia por trás do modo de tipagem do Python (e de outras linguagens que seguem o mesmo padrão), chamada de *duck typing* ou no belo português **tipagem de pato**, indicar quais as implicações que isso traz na hora de programar além de introduzir um conceito chamado **tipagem nominal** e como podemos usá-lo para deixar nosso código mais legível e organizado.

Parece bastante coisa, né? E é mesmo ... então chega de enrolação e vamos para o que interessa!

### Será que é um pato?

Provalmente você deve ter achado no mínimo curioso o nome *duck typing*. Mas como sempre, tudo tem uma explicação e nesse caso ela de uma frase que é tão curiosa quanto esse nome:

> Se anda como um pato e grasna como pato, então deve ser um pato

Essa ideia vem do chamado [teste do pato](https://pt.wikipedia.org/wiki/Teste_do_pato), que em sua essência é uma forma de raciocínio abdutivo que sugere que pode-se compreender a natureza de um sujeito desconhecido analisando as características prontamente indicaficáveis dele. Até eu achei essa definição um pouco misteriosa, então vamos tentar colocar isso em termos mais simples: pelo modo de agir do sujeito você consegue compreender e inferir o que ele é. 

Parece simples e intuitivo. Provavelmente utilizamos esse conceito diariamente para avaliar pessoas, objetos e situações no nosso cotidiano. Mas se pararmos para pensar bem, essa ideia pode ter uma implicação ainda mais forte: eu não me importo com o que esse sujeito é, contanto que ele aja de um determinado modo. Por exemplo, podemos ter o seguinte programa em Python:

<div class="code">
{% highlight python%}
def print_len(obj):
    print(f"Esse objeto tem {len(obj)} elementos")
{% endhighlight %}
</div>

Perceba que não dissemos em nenhum momento no código qual o tipo de dado esperado do parâmetro `obj`, mas dentro do código deixamos subentendido que, independente do tipo desse objeto, esperamos que ele possa retorna algo quando utilizamos a função `len`. Por exemplo, todos os seguintes tipos de dados funcionariam:

<div class="code">
{% highlight python%}
l = list(1, 2, 4)
print_len(l)

d = dict(key_1=1, key_2=2)
print_len(l)

s = "string"
print_len(l)
{% endhighlight %}
</div>

A ideia por trás do *duck typing* é que nós realmente não precisamos saber qual tipo de dado estamos lidando, apenas que esse objeto terá o comportamento esperado quando chamarmos seus métodos, atributos ou os utilizarmos como argumentos em alguma outra função. Não precisamos saber se estávamos recebendo uma lista, um dicionário ou uma *string*:  o método funciona igual para todos eles.

A vantagem de não passarmos qual o tipo de dados é que minimizamos o número de funções que escrevemos: enquanto em linguagens tipadas teríamos que declarar 3 funções diferentes e lidar com conceitos complicados como [sobrecarga de funções](https://pt.wikipedia.org/wiki/Sobrecarga_de_função), aqui podemos declarar um única função e não nos preocupamos mais com o tipo do parâmetro.

Claro que essa facilidade tem um custo: do mesmo jeito que não precisamos nos preocupar com tipo de dados passado, não fazemos nenhuma validação para garantir que o objeto passado tem as características que esperamos. Por exemplo, nada impede que passemos um `int` para a função. Nesse caso acabaríamos com um erro na mão.

<div class="code">
{% highlight python%}
print_len(1)
# Erro
{% endhighlight %}
</div>

### Definindo o que é um pato

Parece que podemos ter um problema então: se por um lado temos uma liberdade muito maior com essa flexibilização do tipo de dado, ela pode ser um pouco dificíl de lidar se não tomarmos cuidado. Nosso exemplo aqui é bem simples, mas quando estamos lidando com projetos e sistemas mais complexos pode ser muito fácil se perder no que cada objeto é capaz de fazer.

Claro que existem maneiras de evitar esses problemas fazendo a checagem manual de qual tipo de dado está sendo passado e tratá-lo com a exceção ou retorno adequado. Uma maneira simples é utilizando a função `isinstance` para checar se o objeto pertence a algum tipo aceito pela função. Por exemplo, no caso da nossa função de imprimir o tamanho de um objeto poderíamos ter:

<div class="code">
{% highlight python%}
def print_len(obj):
    if (
        isinstance(obj, list)
        or isinstance(obj, dict)
        or isinstance(obj, str)
    ):
        print(f"Esse objeto tem {len(obj)} elementos")
    else:
        print(f"Tipo {type(obj)} não suportado")
{% endhighlight %}
</div>

Agora, quando passassemos um `int` para a função não teríamos mais um erro e sim uma mensagem diferente na tela. Problema resolvido e podemos parar por aqui com o artigo! Hmm ... ainda não. O que aconteceria se você quisesse imprimir o tamanho de uma tupla (`tuple`) ou de um conjunto (`set`)? Teríamos que entrar na função e adicionar manualmente esses tipos dentro da função. Parece meio trabalhoso, não? Mas calma que ainda temos algumas alternativas para explorar. 

Vamos aproveitar e mudar um pouco nosso exemplo. Vamos criar uma função `fazer_grasnar` que vai receber um objeto e chamar o método `grasnar` desse objeto. Também vamos definir duas classes `Pato` e `Ornitorrinco` que implementam esse método.

<div class="code">
{% highlight python%}
class Pato:
    def grasnar(self,):
        print("Quack! Eu sou um pato")
        
class Ornitorrinco:
    def grasnar(self,):
        print("Quack! Não sei porque um ornitorrinco grasna")
        
def fazer_grasnar(animal: Pato | Ornitorrinco):
    if (
        isinstance(animal, Pato)
        or isinstance(animal, Ornitorrinco)
    ):
        animal.grasnar()
    else:
        print(f"{type(animal)} não grasna!")
{% endhighlight %}
</div>

Só um adendo antes de prosseguirmos: perceba que, logo depois do argumento `animal`, escrevemos `: Pato | Ornitorrinco`. Isso é chamado de **dica de tipo** (*type hint*, em inglês). Ela indica quais os tipos de dados são aceitos para aquele objeto. Mas tem um porém: em termos de execução ela não significa nada pois, como o Python tem tipagem dinâmica, essa indicação de tipo é só um indicação da pessoa programadora para ela mesma, outras pessoas que entrem em contato com o código ou algum programa de checagem estática (como o [mypy](https://mypy-lang.org)). Por isso recebe o nome de dica.

### Vida de herdeiro

Qual o problema desse código? Novamente, sempre que quisermos trabalhar um novo tipo de dado teremos que adicionar na função uma nova verificação, o que não é muito legal. Se você já estudou ou trabalhou com Programação Orientada a Objetos (POO), certamente tem um mente um solução baseada em herança: por que não criar uma classe abstrata (ou interface) comum à todas as outras classes que grasnam? Vamos ver como ficaria essa implementação:

<div class="code">
{% highlight python%}
from abc import ABC, abstractmethod

class Grasnador(ABC):
    @abstractmethod
    def grasnar(self, ):
        ...

class Pato(Grasnador):
    def grasnar(self,):
        print("Quack! Eu sou um pato")
        
class Ornitorrinco(Grasnador):
    def grasnar(self,):
        print("Quack! Não sei porque um ornitorrinco grasna")
        
def fazer_grasnar(animal: Grasnador):
    if (
        isinstance(animal, Grasnador)
    ):
        animal.grasnar()
    else:
        print(f"{type(animal)} não grasna!")
{% endhighlight %}
</div>

O que aconteceu aqui? Breve revisão de alguns conceitos em POO: 

- Uma classe pode herdar o comportamento e atributos de outra. Isso é feito nas linhas `class Pato(Grasnador)` e `class Ornitorrinco(Grasnador)`, é chamado de **herança**;
- A classe **Grasnador** é chamada de classe pai e as classes **Pato** e **Ornitorrinco** são chamadas de filhas;
- Uma classe abstrata (ou interface) é um tipo especial de classe que não pode gerar objetos, ou seja, não é possível fazer algo do tipo `grasnador = Grasnador()`;
- Uma classe abstrata pode definir **métodos abstratos**, que são métodos que não contém um corpo no momento em que são declarados. A utilidade desses métodos é que uma classe filha tem que necessariamente implementar esse método quando a definimos. Se não tiver a implementação, o Python dará um erro.

Isso resolve nosso problema, pois agora tudo que temos que fazer é herdar essa classe base nas novas classes que queremos criar e tudo funcionará bem! Fazendo dessa forma, temos o que chamamos de **tipagem nominal**: nós criamos um novo tipo de dados (`Grasnador`) e criamos subtipos dele. A checagem que é feita é para sabermos se um objeto pertence a algum subtipo de `Grasnador`. Dessa forma, nós perdemos o poder da *tipagem de pato* do Python e estamos trabalhando com o tipo do dado e não mais com a sua forma. O que eu quero dizer com isso? Vamos analisar o seguinte exemplo:

<div class="code">
{% highlight python%}
class Granso:
    def grasnar(self,):
        print("Quack! Eu sou um ganso")
        
fazer_grasnar(Ganso())
# Erro: Ganso não é do tipo grasnador
{% endhighlight %}
</div>

O código acima ilustra um exemplo em que mesmo que a classe Ganso implemente o método `grasnar`, a função não irá funcionar porque ele não é do tipo `Grasnador`. Com essa abordagem, não importa mais se o objeto tem as características necessárias para aquela função, importa somente se ele é do tipo esperado.

Isso não é de todo ruim e se assemelha muito a conceitos de linguagens tipadas como C++ e Java. Mas acaba inflexibilizando algumas características que fazem o Python muito popular. Esse exemplo pode não ser tão complexo, mas pense que você criou um novo tipo de dados que tem um método `sum`, assim como os *arrays* do *numpy* ou os *DataFrames* do *pandas*. Para fazer a sua classe herdar dessas libs, você teria que herdar todas os outro métodos e atributos associados a esses tipos (o que pode ser muito mais do que o necessário). Para você fazer essas libs herdarem o seu tipo de dado é quase um trabalho hercúleo, principalmente se você não estiver disposto a sujar um pouco as mãos.

### Protocolos ao resgate

Parece que estamos num tipo de encruzilhada. Se de um lado não ter nenhuma checagem pode fazer que nosso programa fique muito suscetível a erros, utilizar esse tipagem nominal nos deixa um tanto quando engessados. Para nossa sorte, existe uma terceira alternativa: a **tipagem estrutural**. Na tipagem estrutural, não temos mais uma classe que serve como base para todas as outras através de herança. Temos agora uma classe chamada **protocolo**, quer serve como um documento de requisitos para um objeto: se uma classe implementar todos os métodos que foram definidos no protocolo, ela automaticamente será associada a esse tipo de dados, sem precisar herdar nada de nenhum outro lugar. Vamos ver como isso fica na prática:

<div class="code">
{% highlight python%}
from abc import abstractmethod
from typing import Protocol

class Grasnador(Protocol):
    @abstractmethod
    def grasnar(self, ):
        ...

class Pato:
    def grasnar(self,):
        print("Quack! Eu sou um pato")
        
class Ornitorrinco:
    def grasnar(self,):
        print("Quack! Não sei porque um ornitorrinco grasna")
        
def fazer_grasnar(animal: Grasnador):
    if (
        isinstance(animal, Grasnador)
    ):
        animal.grasnar()
    else:
        print(f"{type(animal)} não grasna!")
{% endhighlight %}
</div>

Perceba que nem a classe `Pato` nem a classe `Ornitorrinco` estão diretamente associadas ao protocolo `Grasnador`, mas como ambos implementam o método `grasnar` isso já é suficiente para que o Python entenda que eles podem ser representantes dessa classe. Na verdade ... estamos quase lá. Da maneira exposta acima, a checagem só ocorreria por interpretadores estáticos como o *mypy*. Para fazer com que a checagem aconteça forma adequada preciamos de mais duas linhas:

<div class="code">
{% highlight python%}
from abc import abstractmethod
from typing import runtime_checkable, Protocol

@runtime_checkable
class Grasnador(Protocol):
    @abstractmethod
    def grasnar(self, ):
        ...

class Pato:
    def grasnar(self,):
        print("Quack! Eu sou um pato")
        
class Ornitorrinco:
    def grasnar(self,):
        print("Quack! Não sei porque um ornitorrinco grasna")
        
def fazer_grasnar(animal: Grasnador):
    if (
        isinstance(animal, Grasnador)
    ):
        animal.grasnar()
    else:
        print(f"{type(animal)} não grasna!")
{% endhighlight %}
</div>

Pronto! Isso é o suficiente que a checagem aconteça em tempo de execução também! A vantagem que temos é que os programadores não tem mais que saber qual é a classe que eles devem herdar e nem se ela tem algum efeito colateral indesejado no seu código. Sabendo somente o que o objeto tem que fazer é suficiente para implementar uma nova classe totalmente compatível com o código atual. 

> Obs: A checagem dinâmica tem algumas limitações. Quem quiser dar uma olhada sugiro ler a [PEP 544](https://peps.python.org/pep-0544/#runtime-checkable-decorator-and-narrowing-types-by-isinstance)

### Conclusão

A *tipagem de pato* (*duck typing*) é uma característica muito poderosa de linguagens como Python e Javascript. Ela permite uma curva de aprendizado muito mais rápida se comparada com linguagens mais tradicionais com utilizam a *tipagem nominal*. Porém ela também pode ser perigosa se não estivermos atento aos requisitos dos métodos e funções que implementamos. Quando somente uma pessoa está trabalhando no código isso pode não ser um problema, já que essa pessoal tem todo o contexto, mas em projetos maiores essa pode ser uma dor considerável.

As duas alternativas que vimos hoje são ótimas opções para termos um pouco mais de controle sobre os tipos de dados de entrada dos métodos e funções e ambos são igualmente válidos. A *tipagem nominal*, com a utilização de classes abstratas, tende a ser mais comum por já ser utilizada em outras linguagens como C++ e Java, mas nem por isso significa que é melhor: a *tipagem estrutural* pode ser uma ótima saída em algumas situações.

------------------
Por hoje é isso, pessoal! Espero que tenham gostado da explicação e eu vou procurar trazer mais conceitos ligados ao mundo da programação em si como algoritmos e estruturas de dados, arquitetura e desenvolvimento de software e muito mais, além dos tradicionais conteúdos sobre inteligência artifical. Fiquem ligados!
