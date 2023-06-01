# LOGBBOOK 11

## LAB

### TASK 1

Antes de começarmos a tarefa propriamente dita, realizamos os seguintes comandos, como foi recomendado:
- `cp /usr/lib/ssl/openssl.cnf .` para copiar o ficheiro `openssl.conf` para o diretório do *Labsetup*;
- descomentar o parâmetro *unique_subject* no ficheiro `openssl.conf`;
- criamos o ficheiro `index.txt` com `touch index.txt`;
- criamos o ficheiro `serial` com `touch serial`, com o valor 1000.

De seguida executamos o seguinte comando no terminal para criar o nosso CA, e verificamos o conteúdo do certificado e da *key*.
![](https://i.imgur.com/pePVpWj.png)

**Que parte indica que este é um certificado de CA?**
Através da *flag* `CA:TRUE`.
![](https://i.imgur.com/OU6xCDC.png)

**Que parte do certificado indica que este é autoassinado?**
Através do facto que o emissor e o utilizador apresentam os mesmos valores de informação.
![](https://i.imgur.com/tO4nG2S.png)

**No algoritmo RSA, temos um expoente público *e*, um expoente privado *d*, um módulo *n* e dois números secretos *p* e *q*, tais que *n = pq*. Identifique os valores desses elementos no certificado e respetivos arquivos-chave.**
Os valores estão apresentados nas seguintes imagens, identificadas com a respetiva legenda.
|![](https://i.imgur.com/3c2WnTK.png)|
|:---:|
|Expoente Público (*e*)|

![](https://i.imgur.com/imH3xqh.png)|![](https://i.imgur.com/k5eEsum.png) 
:-------------------------:|:-------------------------:
Módulo (*n*) | Expoente Privato (*d*)

![](https://i.imgur.com/bv8m4H0.png)|![](https://i.imgur.com/kWbT8DV.png)
:-------------------------:|:-------------------------:
Prime1 (*p*) | Prime2 (*q*)

### TASK 2

Para criar o CRS para o nosso servidor, usamos o seguinte comando: `openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.andrade2022.com/O=Andrade2022 Inc./C=US" -passout pass:dees -addext "subjectAltName = DNS:www.andrade2022.com, \ DNS:www.andrade2022A.com, \ DNS:www.andrade2022B.com"`.

![](https://i.imgur.com/6jqVWOm.png)

Como foi sugerido, o *site* principal tem o *link* `www.bank32.com`, com dois nomes alternativos: `www.bank32A.com` e `www.bank32B.com`.

### TASK 3

Esta tarefa faz uso do CSR criado na tarefa anterior para criarm um CA para o nosso servidor. No entanto, antes de começar a criar o certificado, foi preciso realizar os seguintes passos:
- criar a pasta *demoCA*, dentro da pasta *Labsetup*;
- criar a pasta *newcerts*, dentro da pasta *demoCA*;
- colocar os ficheiros *index.txt* e *serial* dentro da pasta *demoCA*.

De seguida, realizamos o seguinte comando `openssl ca -config openssl.cnf -policy policy_anything -md sha256 -days 3650 -in server.csr -out server.crt -batch -cert ca.crt -keyfile ca.key` para criar o nosso certificado.
![](https://i.imgur.com/vjFMPx1.png)

Ao executar o seguinte comando, conseguimos confirmar que os nomes alternativos estão configurados. 
`openssl x509 -in server.crt -text -noout`
![](https://i.imgur.com/nZeo0Ty.png)

### TASK 4

Modificamos os ficheiros `Dockerfile` e `bank32_apache_ssl.conf` para usarem o nome do nosso servidor, neste caso `andrade2022`, e construimos o *container*.

![](https://i.imgur.com/oNnPLAD.png)

![](https://i.imgur.com/mw2nOMn.png)

De seguida, executamos os comandos `a2ensite andrade_apache_ssl`, para autorizar o nosso *site*, e `service apache2 start`, para iniciar o serviço. Finalmente, usamos o link `https://www.andrade2022.com` para aceder ao nosso *site*.

![](https://i.imgur.com/3eTh8N2.png)

![](https://i.imgur.com/UxXhjt9.png)

### TASK 5

De forma a redirecionar os utilizadores que tentarem aceder ao *site* `www.example.com` para os nosso site, alteramos o nome do nosso servidor para o seguinte:
![](https://i.imgur.com/m5kKNE0.png)

E adicionamos o `www.example.com` ao nosso ficheiro `/etc/hosts`.
![](https://i.imgur.com/57K4jUW.png)

Assim, sempre que um utilizador tentar aceder ao *site* `https://www.example.com`, o mesmo vai ser redirecionado para o nosso *site*.
![](https://i.imgur.com/WioIdzT.png)

### TASK 6

## CTF

### Desafio 1

Uma vez que é-nos fornecido o valor de *e*, 65537 em decimal (`0x10001 = 65537`, é necessário calcular apenas os valores de *p* e *q*. Este processo voi realizado por tentativa em erro, alcançando o resultado pretendido com os seguintes valores:

- *p* é o número primo a seguir a `2**512`:

![](https://i.imgur.com/chIwGvy.png)

- *q* é o número primo a seguir a `2**513`:

![](https://i.imgur.com/9DOouNW.png)

- obtendo os valores de *p* e *q*, conseguimos calcular o valor de *d*: `d = pow(e,-1 (p-1),(q-1))`.

![](https://i.imgur.com/2RAF0CR.png)

Finalmente, colocamos os valores encontrados no nosso ficheiro *semana12e13.py* que recebe a mensagem do servidor e descodifica a mesma, devolvendo-nos a *flag*.

![](https://i.imgur.com/555gmgK.png)
![](https://i.imgur.com/yQZIFLY.png)

### Desafio 2
