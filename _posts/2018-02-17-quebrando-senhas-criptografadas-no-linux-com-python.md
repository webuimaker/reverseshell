---
layout: post
comments: true
title:  "Como quebrar senhas criptografadas do Linux com Python"
date:   2018-02-17
excerpt: "Veja como fazer um script em Python para quebrar senhas de usuários do Linux"
image: "/images/python_bg01.jpg"
---

## Como quebrar senhas do Linux usando Python?

Ento, aqui começo o texto...

### Como o Linux armazena as senhas

Os conceitos de algoritmos de hash e algortimos de criptografia confundem muitas pessoas. Muitos acreditam que as senhas são criptografadas e armazenadas no computador em algum tipo de arquivo protegido.

Entretando, na realidade não faz sentido nenhum armazenar as senhas do Sistema Operacional (SO) por vários motivos práticos e de segurança.

Assim, o Linux - e outros SO - simplesmente não armazenam a senha propriamente dita, mas sim o seu **hash**. Por exemplo, o hash MD5 da string *"reverseshell.io"* pode ser obtido com o seguinte comando `md5sum` no Terminal:

<pre><font color="#CC0000"><b>carlos@localhost</b></font>:<font color="#3465A4"><b>~</b></font># echo -n "reverseshell" | md5sum
cab3297daeee59980632ffe85f6dc733  
</pre>

Em um sistema Linux atual, os hash das senhas de cada usuário estão localizados no arquivo `/etc/shadow`. Para ver como o SO armazena as informações, vamos buscar criar um usuário **melo**, com a senha **teste123** para ver como ela é escrita no sistema.

<pre><font color="#CC0000"><b>root@localhost</b></font>:<font color="#3465A4"><b>~</b></font># adduser melo
Adding user `melo&apos; ...
Adding new group `melo&apos; (1005) ...
Adding new user `melo&apos; (1005) with group `melo&apos; ...
Creating home directory `/home/melo&apos; ...
Copying files from `/etc/skel&apos; ...
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
</pre>

Com o usuário criado, vamos ver o formato que as informações foram passadas para o arquivo *shadow*.
<pre>
<font color="#CC0000"><b>root@localhost</b></font>:<font color="#3465A4"><b>~</b></font># cat /etc/shadow | grep melo
<b>melo:$6$f/KQjYVZ$s4nSu.O1UTFznXyS1gH0il6ysCzCDIC0g6e.41EixJ3gHvK6mlERBihy9W5T/6btWeyrTRDZWq2YhD6P1Qi5W/:17580:0:99999:7:::</b>
</pre>
