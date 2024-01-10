# Relatório CTF Wakanda - Vulnhub
Link para Download: https://www.vulnhub.com/entry/wakanda-1,251/

## Objetivo
Encontrar as duas flags escondidas.

## Etapas realizadas

1. **Identificação de Portas Abertas**
   - Utilizei o nmap na rede local para descobrir o IP da máquina da Vulnhub. (192.168.15.99)
   - As portas 80, 111, 3333 estão abertas.
   - ![Prompt do Nmap](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/scannmap.png)

2. **Exploração Inicial**
   - Abrindo o site podemos encontrar o seguinte:
   - ![Imagem do site](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/site.png)
   - Abrindo o código-fonte podemos encontrar o seguinte:
   - ![Imagem do código fonte](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/codigofonte.png)

3. **Analisando o código-fonte**
   - Dentro do código-fonte da página podemos encontrar um comentário que nos sugere um parâmetro de idioma, sendo possível mudar o idioma da página utilizando um `?lang=` na url. No caso, o idioma é o francês.
   - Digitando `?lang=fr` no final da url, o idioma do site muda para francês.
   - Sabendo que o `?lang=` aceita um parâmetro, podemos tentar realizar um Path Traversal.
   - Foi testado o `../../../etc/passwd` como parâmetro, mas não funcionou.
   - O parâmetro utilizado foi o `php://filter/convert.base64-encode/resource=index` e o resultado obtido foi este:
   - ![base64criptografado](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/base64criptografado.png)
   - Utilizando um site para descriptografar o código, obtemos isto:
   - ![base64descriptografado](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/base64descriptografado.png)

4. **Utilizando SSH para ter acesso**
   - Um detalhe importante: na página inicial, na parte inferior, podemos ver o seguinte texto: Made by@mamadou.
   - Utilizando o comando `ssh mamadou@192.168.15.99 -p 3333` e utilizando a senha que foi encontrada no código descriptografado, conseguimos acesso.
   - ![Acesso ao SSH](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/acessossh.png)

5. ***Usando o Python para "spawnar" uma shell**
   - Observando atentamente o terminal, percebemos que estamos dentro da IDLE do Python, ou seja, não temos uma shell.
   - Para "spawnar" uma shell, vamos primeiro digitar `import pty`.
   - Importando o pty, digitaremos o comando `pty.spawn("/bin/bash")`
   - ![Imagem dentro da IDLE do Python; "Spawnando" uma shell](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/spawnandoshell.png)

6. **Achando a primeira flag**
   - Usando o `ls` podemos achar a primeira flag.
   - ![Primeira flag](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/primeiraflag.png)

7. **Escalação de privilégios para achar a segunda flag***
   - Induzindo que o nome da primeira flag é "flag1.txt", podemos usar o comando `locate flag2.txt` para achar o diretório que tem o arquivo "flag2.txt".
   - Indo para o diretório /home/devops/ e usando o `ls -la` achamos a segunda flag, porém não temos permissão para ver o que tem dentro do arquivo.
   - ![Sem permissão](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/sempermissaosegundaflag.png)
   - Utilizando o comando `find / -user devops` podemos ver o que o usuário tem permissão ou não para acesso.
   - ![Permissão do usuário](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/permissaousuario.png)
   - Vamos focar na permissão que temos em /tmp/test e em /srv/.antivirus.py.
   - Utilizando o `cat /tmp/test` vemos que ele retornou algo bastante interessante:
   - ![/tmp/test](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/tmptest.png)
   - Usando o `cat /srv/.antivirus.py` ele nos retorna algo que  tem relação com o /tmp/test:
   - ![Antivirus.py](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/antiviruspy.png)
   - Uma coisa que devemos notar é que o .py não foi rodado por nós, e sim, provavelmente, pelo usuário devops. Tendo isso em mente, podemos mudar o antivirus.py para que uma shell reversa seja executada.
   - Indo para o diretório do antivirus.py e usando o nano para editar o arquivo, colocaremos isso dentro do .py:
   - ![Código dentro do .py](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/codigopython.png)
   - Abrindo outro terminal, digitaremos o comando `nc -lvp 4242` para escutar a porta 4242.
   - Agora que está tudo preparado, vamos rodar o .antivirus.py. Rodando o .py, conseguimos uma conexão com o netcat:
   - ![Conexão netcat](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/netcat.png)
   - Feito tudo isso, sabemos que tudo está funcionando corretamente. Agora o que nos resta, dada a teoria de que o .py está sendo executado de tempos em tempos pelo usuário devops, é aguardar que o arquivo seja executado para que tenhamos acesso ao usuário devops. Vamos deixar o comando `nc -lvp 4242` rodando.
   - Após algum tempo a teoria se mostra verdadeira, temos acesso ao usuário devops:
   - ![Acesso ao devops](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/devopsacesso.png)
   - Agora o que nos resta é usar o comando `cat /home/devops/flag2.txt` para pegarmos a segunda flag:
   - ![Segunda flag](https://github.com/Finkeel/Relatorio-Wakanda/blob/main/imagens/segundaflagpega.png)

## Flags encontradas
- Primeira flag: `d86b9ad71ca887f4dd1dac86ba1c4dfc`
- Segunda flag: `d8ce56398c88e1b4d9e5f83e64c79098`

## Conclusão
O CTF Wakanda foi concluído com sucesso, demonstrando habilidades de enumeração, exploração e escalonamento de privilégios para obtenção das flags.

## Vídeo de Resolução
Assista à resolução completa da máquina [aqui](https://www.youtube.com/watch?v=2-VJPadF79A).

## Assinatura
[Gustavo Almeida]
[09/01/2024]
