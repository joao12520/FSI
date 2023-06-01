# logbook5

## CTFS

## Desafio 1

No desafio 1 foi nos dado um ficheiro zip com o código que temos de "atacar", o programa compilado, dois ficheiros de texto e um script em python para fazermos o exploit.

Para começar fizemos o checksec como nos foi recomendado e verificámos que a arquitetura do ficheiro é x86 (Arch), não existe um cannary a proteger o return address (Stack), a stack tem permisssão de execução (NX), as posições do binário não estão randomizadas (PIE) e por fim existem regiões de memória com permissões de leitura, escrita e execução (RWX), neste caso referindo-se à stack. Caso alguma proteção fosse diferente como por exemplo as posições do binário serem randomizadas o nosso ataque de buffer overflow já não seria possível de se concretizar.

![](https://i.imgur.com/JpzzJXf.png)

De seguida começámos a analisar o ficheiro "main.c" e notámos logo que existem duas variáveis locais, o "buffer" que ocupa 20 bytes e o "meme_file" que ocupa 8. Como a varíavel "meme_file" contém o nome do ficheiro onde vai ser escrita a informação que nos interessa esta é a variável que pretendemos modificar.

A unica e mais óbvia maneira de atacar que conseguimos encontrar foi abusando a função strcpy que nos deixa escrever até 28 bytes mas escreve-os na variável buffer que só tem 20 bytes. Como as variaveis locais são todas posicionadas de seguida tendo em conta as proteções existentes, nós conseguimos modificar a variável "meme_file" se escrevermos 28 bytes no input pois os ultimos 8 vão passar a atualizar esta variável.

Tendo isto em conta modificámos o nosso script de python para enviar um input com 20 bytes de "lixo" e no final 8 bytes com o conteudo "flag.txt" isto permite-nos então fazer com que o programa escreva o que está em memória no ficheiro que nós queremos através do buffer overflow.

![](https://i.imgur.com/UG0jPR9.png)

Depois de executarmos este código no url que nos foi dado conseguimos obter a flag e acabar o desafio.

## Desafio 2

No desafio 2 foi outra vez nos dado um ficheiro zip com a mesma estrutura e ficheiro de código em c parecido com o anterior mas agora temos uma variável de 4 bytes entre o buffer o meme_file que queremos modificar.

No entanto, conseguimos verificar também que o tamanho de input máximo passou de 28 bytes para 32 ou seja nós conseguimos modificar os 8 bytes da variável "meme_file" na mesma.

Tendo isto em conta tentámos meter 24 bytes de lixo e nos últimos 8 bytes meter outra vez o nome do ficheiro para onde queremos que o programa escreva o que está em memória mas vimos que o programa está á espera de uns certos valores em hexadecimal para os 4 bytes no meio.

Depois de darmos mais uma vista de olhos no código conseguimos encontrar os valores esperados e adaptamos a nossa input para os incluirmos. Para valores em hexademical o que é lido no programa está em reverse então também tivemos de modificar a ordem na string enviada.

![](https://i.imgur.com/wl5ewSW.png)

![](https://i.imgur.com/AOJhBJj.png)

Depois deste processo executámos o script no url que nos foi fornecido e conseguimos encontrar a segunda flag concluindo assim os ctfs desta semana.


## LAB

Começámos este lab por desativar alguns mecanismos de segurança como nos foi pedido no guião. O primeiro mecanismo que desativámos metia os endereços de memória em locais aleatórios e faz com que ataques de buffer overflow sejam pouco efetivos. Para desativarmos o segundo mecanismo foi preciso trocar de shell pois a default "/bin/sh" prevenia-se de ser executa num processo SET-UID.

![](https://i.imgur.com/X8t3MEi.png)


Na tarefa 1 compilámos o ficheiro "call_shellcode.c" com o makefile fornecido e verificámos que os ficheiros resultantes permitiam-nos entrar na shell de 32 ou 64 bits dependendo do que escolhiamos.

![](https://i.imgur.com/iKFYqEL.png)


De seguida, na tarefa 2 analisámos o ficheiro "stack.c" que nos foi fornecido e verificámos que existia um buffer overflow no strcpy  dentro da função bof. A string pode ter 517 bytes mas o buffer só tem 100 e a função usada não verifica se existe buffer overflow. Como o programa tem como dono o root e é do tipo SET-UID(depois de compilarmos bem o ficheiro) um utilizador normal pode aceder a shell como root se conseguir manipular o ficheiro "badfile" que está dentro do nosso controlo.

![](https://i.imgur.com/V5KPetD.png)

![](https://i.imgur.com/Wm4G1yX.png)

![](https://i.imgur.com/6kxa531.png)

Chegando á tarefa 3 começámos por compilar o ficheiro "stack.c" usando as flags que nos foram dadas que desativavam a stackguard(outro mecanisma de proteção) e tornámos o programa num root-owned SET-UID process atráves dos outros dois comandos.

![](https://i.imgur.com/OwiTRxR.png)


Depois usámos o debugger para descobrir qual é que é a address do ebp register depois de passarmos pela função bof. Esta address aponta para a stack frame da função bof. Também encontrámos o endereço da variável buffer para depois conseguirmos encontrar a distancia entre esta variável e o endereço da stack frame.

![](https://i.imgur.com/tbYYtag.png)


Tendo em conta esta informação começámos a modificar o ficheiro python que nos foi fornecido para o ataque. Para começar colocámos o shellcode de 32 bits que se encontra no guião no sitio respetivo, modificámos a variável start para 517(tamanho de str) menos o tamanho da shellcode para a payload ficar colocada no final.De seguida modificámos o return address value para o endereço da stack frame que encontrámos antes e adicionamos 200 para ficar no meio da payload, também adicionámos 4 para ter em conta a diferença entre o stack frame e o endereço de retorno que num sistema de 32 bits encontra-se 4 bytes acima.

![](https://i.imgur.com/balmqkG.png)

No final modificámos o endereço de retorno para 108 porque esta é a diferença entre o endereço do buffer e o ebp e adicionámos 4 pela mesma razão que foi explicada anteriormente.

![](https://i.imgur.com/wKoV3dm.png)
