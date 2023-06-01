# **Semana #4**

### **Task 1**

- Usamos printenv/env para dar print às environment variables.

**Terminal**

![](https://i.imgur.com/FYCIv0k.png)

### **Task 2**

- Compilamos o programa **(gcc myprintenv.c -o childPrint.out gcc myprintenv.c -o parentPrint.out)** em dois binários, cada um diferindo em qual processo iria imprimir as environment variables (parent ou child)

- Foram executados os dois programas e direcionada a sua saída para arquivos diferentes. A saída dos programas eram as environment variables que eles 'recebiam'

- Comparou-se os arquivos de saída com a ajuda do comando diff e concluiu-se que a saída foi a mesma para ambos os programas

- **Conclusão:** As environment variables para um processo pai e processo filho, criadas através do fork com configuração padrão, são as mesmas

**Programa**

``` c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

extern char **environ;

void printenv()
{
  int i = 0;
  while (environ[i] != NULL) {
      printf("%s\n", environ[i]);
      i++;
  }
}

void main()
{
  pid_t childPid;
  switch(childPid = fork()) {
    case 0:  /* child process */
      // printenv();          
      exit(0);
    default:  /* parent process */
      printenv();       
      exit(0);
  }
}
```

**Terminal**

![](https://i.imgur.com/xEW7NNT.png)

### **Task 3**

- Compilou-se o programa fornecido **(gcc myenv.c)**. O programa essencialmente inicia outro processo através do 'execve' que imprime as environment variables

- There was no output. This can be explained by the fact that the 'execve' function starts a process and associates to it the array of environment variables given to it in the third argument. As this argument was a NULL pointer, the new process has no environment variables
- Não houve nenhum output. Pode ser explicado pelo facto de que a função 'execve' inicia um processo e associa o array de environment variables dado no terceiro argumento. Como este argumento era um ponteiro NULL, o novo processo não possui environment variables

- Quando alteramos este parâmetro para o ponteiro do array de environment variables do processo parent (environ), a saída do programa era exatamente isso, as environment variables do processo parent. Isso era esperado, pois esses seriam os do processo child, já que os passamos pelo terceiro argumento 'execve', confirmando assim o que foi mencionado no último ponto


**Programa**

``` c
#include <unistd.h>

extern char **environ;

int main()
{
  char *argv[2];

  argv[0] = "/usr/bin/env";
  argv[1] = NULL;

  execve("/usr/bin/env", argv, environ);  

  return 0 ;
}
```

**Terminal**

![](https://i.imgur.com/KkXKthw.png)


### **Task 4**

- Criamos um arquivo chamado mysystem.c com o código fornecido no lab. Este programa chama o mesmo processo que o anterior (Tarefa 3), mas através de 'sistema' em vez de execve. O programa foi compilado e executado

- O output foram as environment variables do processo parent, como esperado, já que a função 'system' usa 'execl', que por sua vez chama 'execve' com as EVs do processo parent, que finalmente executa /bin/sh com o argumento passado para o 'sistema'

**Programa**

``` c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    system("/usr/bin/env");
    return 0 ;
}
```

**Terminal**

![](https://i.imgur.com/TuxuQwT.png)

### **Task 5**

- Um programa Set-UID é um programa que assume os privilégios de seu proprietário durante a execução, em vez do usuário que o executa

- Criamos um arquivo chamado evs.c com o código fornecido, que imprime as suas EVs. Em seguida, foi compilado e tornei o root o seu proprietário e transformei-o num programa Set-UID através das instruções mostradas no terminal

- Finalmente, como um usuário normal, alterou alguns valores de variáveis ​​de ambiente e executou o programa

- Verificando o output do programa, embora os valores dos EVs tenham mudado, um deles não estava presente

**Programa**

``` c
#include <stdio.h>
#include <stdlib.h>

extern char **environ;

int main() {
    int i = 0;
    while (environ[i] != NULL) {
    printf("%s\n", environ[i]);
    i++;
    }
}
```

Terminal | Resultado
:---------:|:---------:
![](https://i.imgur.com/IWl3fdJ.png) | ![](https://i.imgur.com/x1bSqMH.png)


### **Task 6**
 - Como o programa SET-UID é executado com os privilégios de seu proprietário, um user executando o programa pode manipular os EVs do programa a seu favor. Neste caso, o programa executa o comando 'ls' chamando a função system. Podemos manipular os EVs do programa para induzi-lo a chamar outra versão de ls

- O único obstáculo é que o sistema caller acabará por levar a uma call para /bin/dash, que possui medidas preventivas para esta situação, diminuindo as permissões do processo para as permissões do caller. Para isso, e para podermos explorar a situação acima, vamos mudar o programa shell chamado para outro (/bin/zsh)

- Depois de anexar /home/seed à variável de ambiente PATH, continuamos a criar outra versão de ls nesse diretório, que tornou uma operação disponível apenas para root

- Por fim, compilamos o programa fornecido e transformamos num programa SET-UID

- Quando executamos o programa, em vez de uma lista de arquivos num diretório, vimos a mensagem "BOO!" impressa na tela e as permissões de um 'important_file' alteradas, significando que a nossa versão de ls foi a que foi executada. Isso demonstra o poder de mudar os EVs

**Programa**

``` c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    system("ls");
    return 0;
}
```

Criar ls | Execução
:---------:|:---------:
![](https://i.imgur.com/nqt1hNb.png) | ![](https://i.imgur.com/VvAFFzb.png)

## **CTF**

### **Desafio 1** ###

- Depois de uma procura por um CVE que atendesse aos requisitos descritos no enunciado, encontramos o CVE-2021-34646 (https://nvd.nist.gov/vuln/detail/CVE-2021-34646), colocando o respectivo código como flag para completar o desafio 1 da semana 4.

![](https://i.imgur.com/jeLPgR2.jpg)

### **Desafio 2** ###

- Depois, procuramos o exploit em (https://www.exploit-db.com/exploits/50299), executando código python na nossa máquina que retornou três comandos na linha de comandos.

![](https://i.imgur.com/ApBZD0e.jpg)

- Selecionando o primeiro link apresentado, fomos redirecionados para a página desejada do WordPress, com privilégios de administrador. Por fim, foi apenas necessário carregar a página disponibilizada pelo comunicado (http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php), introduzir a mensagem privada, e submeter a flag encontrada .

![](https://i.imgur.com/odk6JoH.jpg)
