# LOGBOOK 7

## LAB

Para começar desativámos as proteções tal como nos foi recomendado no anexo.

![](https://i.imgur.com/avCdbWO.png)

Analizamos o ficheiro e verificamos a string format vulnerability

De seguida usamos os comandos de *docker* para construir e começar os *containers* que vão servir como servidores.

![](https://i.imgur.com/E8CfHWD.png)

![](https://i.imgur.com/kCwOFxD.png)

Mandámos uma *string* para o servidor e verificamos na consola que a resposta foi a esperada

![](https://i.imgur.com/69eYWXn.png)

![](https://i.imgur.com/yvjRncZ.png)

De seguida modificámos o ficheiro build-string.py para que a *string* s ficasse com %x em vez de de %.8x para ficar parecido com o que nos foi sugerido no site recomendado no moodle, incluimos isto no exploit.py e enviámos o ficheiro para o servidor verificando de seguida que crashou porque não enviou a mensagem de "Returned properly"

![](https://i.imgur.com/G5zRA2X.png)

![](https://i.imgur.com/UTHlID2.png)

A tarefa seguinte implicava encontrar em que posição da *stack* os nossos valores estavam a ser guardados. Decidimos usar a *string* "aaaa", com valor hexadecimal igual a 0x61616161, como o nosso valor a ser encontrado. Através de tentativa e erro, descobrimos que a *stack* guarda estes valores 63 posições acima, sendo o 64.º carácter %x usado para escrever o endereço pretendido.

![](https://i.imgur.com/9JWe4I6.png)

![](https://i.imgur.com/DWX902o.png)

---

De forma a tentar descobrir a mensagem secreta escondida *heap*, decidimos alterar o valor do nosso *number* para o endereço da mensagem secreta - 0x080b4008 - e, usando o mesmo *offset* de 63 bytes, mais o carácter especial %s, que escreve a *string* guardada no endereço para o qual está a apontar, conseguimos escrever a mensagem pretendida.

![](https://i.imgur.com/WXYuEIa.png)

![](https://i.imgur.com/W3N3ojL.png)

---

Finalmente, era necessário tentar alterar o valor da variável *target*, armazenada no endereço 0x080e5068, para o valor 0x5000, por isso decidimos usar o carácter especial %n. Devido a esta abordagem, era necessário preencher o nosso *buffer* com 20 480 bytes de forma a que o %n armazenasse o valor correto em *target*. Este processo implicava reduzir o nosso *offset* para 62 bytes de forma a conseguir suportar o carácter %n sem comprometer a posição que queríamos apontar. De seguida, bastava preencher a nossa *string* com carácteres suficientes, alcançando o valor pretendido com a *string* "%20244x".

![](https://i.imgur.com/8D4HlwH.png)

![](https://i.imgur.com/TYksMGs.png)

## CTF

### Desafio 1

A vulneravilidade do código encontra-se na linha 27 do ficheiro **main.c** : `print(buffer);`. Esta vulnerabilidade é do tipo `Format String` e permite-nos crashar o programa, ler e alterar valores em memória e injetar código malicioso.

De seguida, tentamos encontrar o endereço da nossa *flag* recorrendo ao gdb, verificando que a mesma encontra-se no endereço `0x60c00408`, uma vez que a *stack* guarda os bytes menos significativos em posições mais baixas da *stack*, e os mais significativos em posições mais elevadas.

![](https://i.imgur.com/omyiKBz.png)

Finalmente, fomos ao ficheiro **exploit_example.py** e modificamo-lo de forma  a enviar o endereço da variável *flag*, seguido de um `%s`:

![](https://i.imgur.com/4O8MMLQ.png)

Desta forma, só bastava correr o programa de forma remota, devonvendo-nos a *flag* desejada: **flag{c977ba9a8c3c2345c93a6068e6e1e67b}**.

![](https://i.imgur.com/Z2XZFFK.png)

### Desafio 2

À semelhança do desafio 1, a vulnerabilidade é na mesma de tipo `Format String` e encontra-se na linha 14 do ficheiro **main.c** : `printf(buffer);`. No entanto, a *flag* não é carregada para a memória, mas podes ter acesso à mesma através da varável local `ḱey`. Assim, o nosso objetivo é alterar o valor de 'key' para '0xbeef' de forma a ter acesso à *flag*.

Para realizar este *exploit*, decidimos reutilizar o programa **exploit_example.py** fornecido no desafio 1.

Uma vez que o número hexadecimal `0xbeef` corresponde ao número decimal **48879**, vamos instroduzir 48879 bytes na nossa *string* antes de um caractér `%n`, de forma a alterar a *flag* para o valor pretendido.

Recorrendo ao gdb, conseguimos verificar o endereço da variável *key*.
![](https://i.imgur.com/uPi2dxw.png)

Assim, enviamos a *string* **"????\x34\xc0\x04\x08%48871%n"**. Esta *string* é constituida por: um valor aleatório de 4 *bytes*; o nosso endereço alvo em formato *little endian*; uma sequência de *bytes* para chegar ao valor pretendido (neste caso só colocamos 48871 porque já são colocados 4 *bytes* com o valor aleatório, mais 4 *bytes* com o endereço); e o caractér `%s` para encrever o valor para a variável *key*.

![](https://i.imgur.com/4RmTm61.png)

Executando o *exploit* e o comando `cat flag.txt`, encontramos a *flag*.

![](https://i.imgur.com/CGKden1.png)
