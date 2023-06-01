# LOGBOOK 10

## LAB

### Tarefa 1

Ao colocarmos o código `<script>alert('XSS');</script>` no campo de *Brief Description*, o mesmo é executado aparecendo uma mensagem de alerta sempre que a página de utilizador é carregada. Assim, comprovamos que é possível executar código de ***JavaScript*** através de *Cross-Site Scripting (XSS)*.

![](https://i.imgur.com/Azvaqmt.png)

![](https://i.imgur.com/0REb4MC.png)

### Tarefa 2

À semelhança da tarefa anterior, introduzimos o código `<script>alert(document.cookie);</script>` na *Brief Description* do utilizador. Desta vez, no entanto, a mensagem de alerta mostra informação referente às cookies do utilizador, comprovando que é possível aceder a informação sobre o mesmo.

![](https://i.imgur.com/qou2IYn.png)

![](https://i.imgur.com/qnRZCSE.png)

### Tarefa 3

Uma vez que a informação obtida na tarefa anterior só é visivel pelo utilizador, e não pelo atacante, introduzimos o código seguinte de forma a enviar a informação pretendida para o atacante.

![](https://i.imgur.com/A3bUThc.png)

Este comando carrega o *URL* intrduzido no campo *src* da imagem que, consequentemente, envia a infromação das *cookies* para porta 5555 do IP 10.9.0.1, pertencente ao atacante.

A seguinte imagem, mostra que a informação pretendida foi recebida com sucesso.

![](https://i.imgur.com/oBkHPK5.png)

### Tarefa 4

Nesta tarefa, pretendemos criar um `script` que cria e aceita um pedido de amizade entre a Alice e o Samy. De froma a tentar perceber como os pedidos de amizade funcionam, enviamos um pedido para qualquer utilizador e recorremos ao *HTTP Header Live* para obter o seguinte *link*:

![](https://i.imgur.com/Ig0B422.png)

Após realizarmos o mesmo processo para o Samy, descobrimos que o seu *ID* é 59. De seguida, modificamos o código fornecido no enunciado e construímos o seguinte pedido: 
`var sendurl='http://ww.seed-server.com/action/friends/add?friends=59' + ts + token.` e colocamo-lo no campo de *About me* no perfil do Samy.

![](https://i.imgur.com/rL5xvLp.png)

Primeiro confirmamos que a Alice não é amiga do Samy.

![](https://i.imgur.com/ntMNQXx.png)

Depois, fomos ao perfil do Samy com a Alice.

![](https://i.imgur.com/DQPCt1J.png)

O *script* é executado automaticamente, adicionando o Samy aos amigos de Alice.

![](https://i.imgur.com/nfoyR6p.png)

## CTF

### Desafio 1

A estrategia usada para resolver este CTF foi usar uma ataque XSS e esperar que o site desse refresh para depois obtermos a flag.

#### Com vulnerabilidade XSS demos um alerta ####
|Script| Alert Trigger |
|:---------:|:---------:|
|![](https://i.imgur.com/0VgxCbx.png) | ![](https://i.imgur.com/MTinkZ2.png)|

#### Usamos XSS para clicar no botao que estava desativado ####
|Script| Flag |
|:---------:|:---------:|
|![](https://i.imgur.com/neX2dZ0.png) | ![](https://i.imgur.com/K96kkCT.png)|

### Desafio 2

Neste desafio tentamos dar login sem credenciais e recebemos um output de erro. Tendo a possibilidade de pingar o host assim o fizemos e usando um ping aleatório e o comando cat obtemos por fim a flag pretendida.

|Login page| No credentials |
|:---------:|:---------:|
|![](https://i.imgur.com/pkHRgQi.png) | ![](https://i.imgur.com/PWAGLNd.png)|

|Ping page| Ping command |
|:---------:|:---------:|
|![](https://i.imgur.com/Q006LAT.png) | ![](https://i.imgur.com/6nOWwiH.png) | 

|Flag|
|:---------:|
|![](https://i.imgur.com/yOQwFxb.png)|
