# Relatório do projeto StaySafe

Sistemas Distribuídos 2020-2021, primeiro semestre


## Autores
  
**Grupo G06**  

| Número | Nome              | Utilizador                       | Correio eletrónico                               |
| -------|-------------------|----------------------------------| -------------------------------------------------|
| 93584  | João Lopes        | <https://github.com/varcheiro99> | <mailto:joao.costa.lopes@tecnico.ulisboa.pt>     |     
| 93617  | Teresa Matos      | <https://github.com/teresacm17>  | <mailto:teresa.colaco.matos@tecnico.ulisboa.pt>  |

POR AQUI FOTOS
![Alice](alice.png) ![Bob](bob.png)


## Melhorias da primeira parte

- [descrição da alteração](https://github.com/tecnico-distsys/GXX-StaySafe/commit/156e1ac25798e2360b362b3a8fc474f7cfe64d01)


Correção de erro: no ficheiro DGSServImpl, o método AVGCalculator(cálculo da probabilidade média) era chamado como medCalculator.
https://github.com/tecnico-distsys/G06-StaySafe/commit/c523e95a6c4bcd92a171613751295cd257ebbc53


## Modelo de faltas
Na medida em que implementamos um sistema assíncrono, não existem limites temporais para entrega de mensagens. Apenas confirmamos que de 30 em 30 segundos.

FALTAS SILENCIOSAS
Relativamente ao modelo de faltas, decidimos verificar os casos em que as réplicas falham: se estiverem ligadas a algum cliente, este será redirecionado para uma réplica disponível, podendo assim continuar a enviar os seus pedidos. Caso não esteja nenhuma réplica disponível, o cliente é notificado.
Certificamo-nos que, na ocorrência de uma falha, o conteúdo da réplica afetada é replicado para as restantes réplicas ativas, impedindo a perda de informação. 

## Solução

_(Figura da solução de tolerância a faltas)_
![Alice](alice.png)
figura 2a

Na figura 2a é representado o caso "normal" em que tudo corre como é previsto, ou seja, a réplica à qual o cliente se conectou e, posteriormente, enviou um pedido responde ao mesmo com o resultado suposto. 

![Alice](alice.png)    ![Bob](bob.png)
figura 2b               figura 2c

Na figura 2b, apresentamos um caso no qual ocorre uma falta. Neste caso, o cliente conecta-se a uma réplica que, mais tarde, perde a ligação ao servidor de nomes. Após este acontecimento, quando o cliente enviar um pedido é levantada uma exceção que vai redirecionar o cliente a uma nova réplica disponível, de forma aleatória, o que podemos verificar na figura 2c. Desta forma, será possível ao cliente continuar a enviar pedidos e receber as respetivas respostas.


## Protocolo de replicação
Visto que nos foi pedido que implementassemos um protocolo de replicação que garantisse coerência fraca, replicação periódica por parte dos servidores e coerência na leitura por parte dos clientes, decidimos utilizar o protocolo *Gossip*.
O *Gossip* garante coerência fraca e oferece acesso rápido entre os clientes e o servidor, pelo que cada cliente apenas tem de contactar uma réplica para obter resposta aos seus pedidos. 
Para isto ser possível e para que as diferentes réplicas ofereçam valores atualizados, estas trocam entre si os pedidos recebidos e respetivas *vectorial timestamps* (vetor que contém o número de atualizações correspondente a cada réplica), ao fim de cada período de 30 segundos.
De modo a que um cliente não receba respostas incoerentes por parte do servidor (no caso em que faz um pedido de leitura a uma réplica que se encontra mais desatualizada que a última contactada), este guarda a resposta mais recente recebida pela réplica e a respetiva *vectorial timestamp*. No caso em que a *vectorial timestamp* recebida pela réplica está desatualizada em relação à *timestamp* do cliente, o resultado obtido por este é o que ficou guardado da última leitura, evitando a receção de resultados equivocados.

Relativamente à distribuição de réplicas por cliente, é dada ao mesmo a opção de indicar a réplica pretendida para a execução dos pedidos. Caso contrário, é-lhe atribuída uma das réplicas aleatoriamente.

Aquando de um pedido do cliente à réplica, este é enviado para um Frontend onde existe um canal de comunicação GRPC entre o cliente e a réplica, sendo através deste enviado o pedido e recebida a respetiva resposta. Quando a resposta for recebida, esta é enviada pelo frontend para o cliente. Para isto, são utilizados canais bloqueantes (GRPCBlockingStub) que permitem fazer a ligação entre o cliente e o servidor, de forma a ser possibilitar a troca de pedidos e respostas.  


## Opções de implementação

Decidimos implementar 3 réplicas de servidores de forma a suportar 1 eventual falta. Assim, resultam de 2*f+1 = 3 réplicas, onde f é o número de faltas suportadas.
Relativamente aos argumentos dados pelo cliente, é sempre necessário dar o Host e o Port do servidor de nomes (ZKNaming). Caso o cliente queira especificar a réplica à qual quer ser conectado poderá fazê-lo após a indicação do host e port do servidor de nomes (referidos anteriormente), dando um número de 1 a 3, sendo estes os indicadores possíveis para uma réplica. Se o cliente indicar um número inválido (diferente de 1,2 ou 3), é-lhe enviada uma mensagem de erro. Poderá também acontecer, que a réplica dada pelo cliente esteja indisponível. Perante esta situação, o servidor de nomes atribui ao mesmo uma réplica disponível de forma aleatória. Relativamente ao caso em que não é fornecido um identificador de réplica, o ZKNaming atribui uma réplica de forma aleatória, tal como na última situação. 

Para além isso, estabelecemos que quando um cliente tenta enviar um pedido ao servidor que lhe está atribuído e o mesmo já não se encontra disponível, o ZKNaming atribui uma nova réplica ao cliente. Este, por sua vez, recebe a resposta ao pedido que efetuou anteriormente, que entretanto foi reenviada para a nova réplica.


## Notas finais

