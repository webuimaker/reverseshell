---
layout: post
comments: true
title:  "Como quebrar senhas criptografadas do Linux com Python"
date:   2018-02-17
excerpt: "Veja como fazer um script em Python para um ataque de dicionário contra os hashes de senhas do Linux. Primeiramente, vamos entender como os hashes SHA-512 são gerados e em qual arquivo são armazenados. Após, com poucas linhas de código, vamos escrever o programa."
image: "/images/python_bg01.jpg"
---

## Como quebrar senhas do Linux usando Python?

Neste primeiro artigo, vamos desenvolver um script simples em Python para efetuar um **ataque de dicionário** em hashes de senhas de Linux.

Para isso, vamos entender primeiramente como as senhas são armazenadas, e entender o conceito de hash.

### Hash, senha e criptografia

Os conceitos de algoritmos de hash e algortimos de criptografia confundem muitas pessoas. Muitos acreditam que as senhas são criptografadas e armazenadas no computador em algum tipo de arquivo protegido.

Entretando, na realidade não faz sentido nenhum armazenar as senhas do Sistema Operacional (SO) por vários motivos práticos e de segurança.

Assim, o Linux - e outros SO - simplesmente não armazenam a senha propriamente dita, mas sim o seu **hash**. Por exemplo, o hash MD5 da string *"reverseshell.io"* pode ser obtido com o seguinte comando `md5sum` no Terminal:

{% highlight bash %}
carlos@localhost:~# echo -n "reverseshell" | md5sum  
cab3297daeee59980632ffe85f6dc733
{% endhighlight %}

### Como o Linux armazena as senhas

Em um sistema Linux atual, os hash das senhas de cada usuário estão localizados no arquivo `/etc/shadow`. Para ver como o SO armazena as informações, vamos buscar criar um usuário **melo**, com a senha **teste123** para ver como ela é escrita no sistema.

{% highlight bash %}
root@localhost:~# adduser melo
Adding user `melo`; ...
Adding new group `melo`; (1005) ...
Adding new user `melo`; (1005) with group `melo`; ...
Creating home directory `/home/melo`; ...
Copying files from `/etc/skel`; ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for melo
Enter the new value, or press ENTER for the default
    Full Name []: 
    Room Number []: 
    Work Phone []: 
    Home Phone []: 
    Other []: 
Is the information correct? [Y/n] y
{% endhighlight %}

Com o usuário criado, vamos ver o formato que as informações foram passadas para o arquivo *shadow*.


{% highlight bash %}
root@localhost:~# cat /etc/shadow | grep melo
melo:$6$f/KQjYVZ$s4nSu.O1UTFznXyS1gH0il6ysCzCDIC0g6e.41EixJ3gHvK6mlERBihy9W5T/6btWeyrTRDZWq2YhD6P1Qi5W/:17580:0:99999:7:::
{% endhighlight %}

Para criar o script em Python para quebrar esse hash, vamos pegar apenas a parte que está destacada em negrito na linha acima. Para entender essa *string*, é preciso entender que ela é dividida em 3 partes, cada uma com um significado:

1. **Algoritmo de hash:** $6
2. **Salt:** f/KQjYVZ
3. **Valor do hash:** s4nSu.O1UTFznXyS1gH0il6ysCzCDIC0g6e.41EixJ3gHvK6mlERBihy9W5T/6btWeyrTRDZWq2YhD6P1Qi5W/

Neste caso, o algoritmo de hash utilizado foi o **SHA-512** (representado por **$6**). Este é o padrão do Linux, e essa informação pode ser achada em `/etc/login.defs`.

{% highlight bash %}
root@localhost:/# cat /etc/login.defs | grep ENCRYPT_METHOD
ENCRYPT_METHOD SHA512
{% endhighlight %}

Com isso já temos uma informação de entrada para desenvolvermos um script básico de ataque ao hash, baseado em dicionário.

Caso tenha ficado confuso e queira ver um tutorial básico muito interessante, recomendo [este artigo aqui](https://www.vivaolinux.com.br/artigo/Armazenamento-de-senhas-no-Linux?pagina=1)

### Gerando hash com o módulo *passlib.hash*

O primeiro passo é aprender como gerar um hash SHA-512 em Python. Para isso, vamos utilizar a biblioteca `passlib.hash`. Nele podemos encontrar diversos algoritmos de hash, incluindo o `sha512_crypt`, que é o que queremos.

{% highlight python %}
from passlib.hash import sha512_crypt
{% endhighlight %}

Com o algoritmo importado, vamos definir as variáveis contendo o *salt* e o valor do *hash* que queremos obter.

{% highlight python %}
from passlib.hash import sha512_crypt

salt = "f/KQjYVZ"
hash_esperado = "s4nSu.O1UTFznXyS1gH0il6ysCzCDIC0g6e.41EixJ3gHvK6mlERBihy9W5T/6btWeyrTRDZWq2YhD6P1Qi5W/"
{% endhighlight %}

Vamos criar duas funções para o nosso script de ataque de dicionário. Uma função vai importar um arquivo *txt*, que representa o nosso dicionário, e outra função que vai retornar o hash SHA-512. A função que gera o hash recebe dois argumentos: (1) uma senha em *textplain*  e (2) o salt que identificamos no arquivo ```/var/shadow/```.

{% highlight python %}
def gerar_hash(password, salt):
    ans = sha512_crypt.hash(password, salt=salt, rounds=5000)
    return ans

def importar_dicionario(nome_do_arquivo):
    dicionario = []
    with open(nome_do_arquivo) as senhas:
        for line in senhas:
            dicionario.append(line.strip("\n"))
    return dicionario
{% endhighlight %}

Apenas um detalhe, por padrão o Linux utiliza ```rounds=5000```. Por isso ```sha512_crypt.hash(password, salt=salt, rounds=5000)```. 

Pronto! Apenas com essas duas funções e um dicionário de senhas, podemos definir a função principal do script. A ideia é usar o ```enumerate()``` para iterar sobre as senhas do dicionário e calcular seus hashes.

Dai, basta fazer a compração entre o hash que havíamos conseguido extrair lá no começo e o hash gerado em cada iteração. Apenas um lembrete, a string gerada pela função ```gerar_hash()``` segue o formato do arquivo *shadow*. Ou seja, o hash propriamente dito inicia apenas no $13 ^ o $ dígito.   

{% highlight python %}
def main():
    dicionario = importar_dicionario("listadesenhas.txt")
    
    for i,j in enumerate(dicionario):
        hash_gerado = gerar_hash(j, salt)
        if hash_gerado[12:] == hash_esperado:
            print("[+] Senha encontrada: {}".format(j))
            return None
    print("[-] Senha não encontrada.")
{% endhighlight %}

Assim, o arquivo final fica:

{% highlight python %}
from passlib.hash import sha512_crypt

salt = "f/KQjYVZ"
hash_esperado = "s4nSu.O1UTFznXyS1gH0il6ysCzCDIC0g6e.41EixJ3gHvK6mlERBihy9W5T/6btWeyrTRDZWq2YhD6P1Qi5W/"

def gerar_hash(password, salt):
    ans = sha512_crypt.hash(password, salt=salt, rounds=5000)
    return ans

def importar_dicionario(nome_do_arquivo):
    dicionario = []
    with open(nome_do_arquivo) as senhas:
        for line in senhas:
            dicionario.append(line.strip("\n"))
    return dicionario

def main():
    dicionario = importar_dicionario("listadesenhas.txt")
    
    for i,j in enumerate(dicionario):
        hash_gerado = gerar_hash(j, salt)
        if hash_gerado[12:] == hash_esperado:
            print("[+] Senha encontrada: {}".format(j))
            return None
    print("[-] Senha não encontrada.")
        
        
if __name__ == "__main__":
    main()
{% endhighlight %}

Para o teste, estou usando um arquivo contendo 1000 senhas mais utilizadas. Vamos executar esse script no Terminal e ver o resultado:

<pre><font color="#CC0000"><b>carlos@localhost</b></font>:<font color="#3465A4"><b>/</b></font>$ python crack.py
[+] Senha encontrada: teste123
</pre>

### Conclusão

Aí está! Neste primeiro artigo vimos como fazer um script rápido para ataque de dicionário contra hashes do Linux. Obviamente, o sucesso de um ataque desse tipo depende muito da qualidade das senhas a serem testadas. Por isso vale a pena baixar dicionários já prontos ou construir o seu próprio, o que representaria uma eficiencia bem maior também.
