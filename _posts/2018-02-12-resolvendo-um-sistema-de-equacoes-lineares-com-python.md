---
layout: post
comments: true
title:  "Resolvendo um Sistema de Equações Lineares com Python"
date:   2018-02-12
excerpt: "Veja duas maneiras de resolver um sistema de equações lineares utilizando Python."
image: "/images/python_math_bg.jpg"
---

## Resolvendo um Sistema de Equações Lineares com Python

Matematicamente, um sistema linear (ou sistema de equações lineares) é definido com um conjunto limitado de $$m$$ equações lineares, sendo também o número $$n$$ de variáveis finito. Sua forma segue o seguinte padrão:

$$
\begin{cases} a_{11}x_{1} + a_{12}x_2 + \dots + a_{1n}x_n = b_1 \\ a_{21}x_{1} + a_{22}x_2 + \dots + a_{2n}x_n = b_2 \\ \dots \\ \dots \\ a_{m1}x_{1} + a_{m2}x_2 + \dots + a_{mn}x_n = b_m \end{cases}
$$

Existem diversas maneiras de se resolver um sistema linear, e não é do escopo deste artigo entrar em nenhuma delas, apenas apresentar duas maneiras de resolvê-las: "manualmente" e usando a biblioteca ```numpy.linalg```. 

### Resolvendo por meio de uma equação matricial

Vamos pegar um exemplo envolvendo o tipo mais simples que um sistema linear pode ter: um sistema linear com duas equações e duas variáveis. Para isso, considere o sistema de equações lineares do exemplo abaixo:


$$
\begin{cases} x & + y & = 6 \\ -3x & + y & = 2 \end{cases}
$$

Um sistema com equações lineares pode ser representado na forma de matriz. Neste caso, nas linhas estarão os coeficientes das incógnitas, com as colunas representando o posicionamento de cada termo no sistema.

A equação acima tem a seguinte representação matricial:

$$ \mathbf{A} \mathbf{X} = \mathbf{B} $$

$$
\begin{bmatrix}
1 & 1 \\
-3 &  1  
\end{bmatrix}  
\begin{bmatrix}
x \\
y  
\end{bmatrix}  
=  \begin{bmatrix}
6 \\
2  
\end{bmatrix}  
$$

Assim, para representar matricialmente o nosso problema no Python, vamos definir 3 elementos:

$$
\mathbf{A} =  \begin{bmatrix}
1 & 1 \\
-3 &  1  
\end{bmatrix}  
$$

$$
\mathbf{X} =  \begin{bmatrix}
x \\
y  
\end{bmatrix}  
$$

$$
\mathbf{B} =  \begin{bmatrix}
6 \\
2  
\end{bmatrix}  
$$

Utilizando algumas propriedades das matrizes, podemos isolar então o vetor $$X$$ com as variáveis desconhecidas. Assim, para encontrar a solução do problema, basta resolver o produto entre a inversa da matrix $$A$$ e a matrix $$B$$.

$$ A X = B $$

$$ A^{-1} A X = A^{-1} B $$

$$ \mathbf{X} = \mathbf{A^{-1}} \mathbf{B} $$

{% highlight python %}
import numpy as np

# Declarar as matrizes
A = np.array([[1,1],[-3,1]])
B = np.array([[6],[2]])

# Encontrar a inversa de A
A_inversa = np.linalg.inv(A)

# Encontrar X = A^{-1} * B
X = np.dot(A_inversa, B)

# mostrar o resultado para cada variável
print("x = ", X[0])
print("y = ", X[1])
{% endhighlight %}

Rodando o script acima, a gente encontra a solução para esse problema. Os valores das variáveis x e y são:

{% highlight bash %}
x =  [ 1.]
y =  [ 5.]
{% endhighlight %}

### Resolvendo com *np.linalg.solve()*

O exemplo acima foi só para você entender como as coisas ocorrem nos bastidores, e relembrar os conceitos por traz dos sistemas lineares.

Entretanto, para se resolver de uma maneira bem mais fácil, tudo o que precisamos é declarar as variáveis $A$ e $B$ e usar a funcão ```np.linalg.solve(A,B)```. Veja abaixo:

{% highlight python %}
import numpy as np

# declarar as matrizes
A = np.array([[1,1],[-3,1]])
B = np.array([[6],[2]])

# encontraremos X usando a função solve()
X = np.linalg.solve(A,B)

# mostrar o resultado para cada variável
print("x = ", X[0])
print("y = ", X[1])
{% endhighlight %}

Da mesma maneira, ao rodar o script o resultado encontrado como solução matemática é o mesmo, $$x = 1$$ e $$y = 5$$.

{% highlight bash %}
x =  [ 1.]
y =  [ 5.]
{% endhighlight %}

E foi isso então, um breve artigo para entender como resolver um sistema de equações lineares e ver como podemos resolvê-lo com poucas linhas de códigos.
Caso queira ver o arquivo em Jupyter Notebook deste artigo, é só entrar no [meu GitHub](https://github.com/carlosfab/reverseshell_posts/blob/master/Sistemas_Lineares_com_Python.ipynb). Abraços!

