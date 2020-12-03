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

_(que correções ou melhorias foram feitas ao código da primeira parte -- incluir link para commits no GitHub onde a alteração foi feita)_

- [descrição da alteração](https://github.com/tecnico-distsys/GXX-StaySafe/commit/156e1ac25798e2360b362b3a8fc474f7cfe64d01)


## Modelo de faltas
Na medida em que implementamos um sistema assíncrono, não existem limites temporais para entrega de mensagens. Apenas confirmamos que de 30 em 30 segundos.
Na parte do cliente -> ver o isTerminated na cena do stub!!!!!!!!!!!!!!!!!!!!!

FALTAS SILENCIOSAS
Relativamente ao modelo de faltas, decidimos implementar 

Notas:
(falta é a causa de um erro)


## Solução

_(Figura da solução de tolerância a faltas)_

_(Breve explicação da solução, suportada pela figura anterior)_


## Protocolo de replicação
Visto que nos foi pedido que implementassemos um protocolo de replicação que garantisse coerência fraca, replicação periódica por parte dos servidores e coerência na leitura por parte dos clientes, decidimos utilizar o protocolo *Gossip*.
O *Gossip* garante coerência fraca e oferece acesso rápido entre os clientes e o servidor, pelo que cada cliente apenas tem de contactar uma réplica para obter resposta aos seus pedidos. 
Para isto ser possível e as várias réplicas oferecerem valores atualizados, estas trocam entre si os pedidos recebidos e respetivas *vectorial timestamps* (vetor que contém o número de atualizações correspondente a cada réplica), ao fim de cada período de 30 segundos.
De modo a que um cliente não receba respostas incoerentes por parte do servidor (no caso em que faz um pedido de leitura a uma réplica que se encontra mais desatualizada que a última contactada), este guarda a resposta mais recente recebida pela réplica e a respetiva *vectorial timestamp*. No caso em que a *vectorial timestamp* recebida pela réplica está desatualizada em relação à *timestamp* do cliente, o resultado obtido por este é o que ficou guardado da última leitura.

Relativamente à distribuição de réplicas por cliente, é dada ao mesmo a opção de indicar a réplica pretendida para a execução dos pedidos. Caso contrário, é-lhe atribuída uma das réplicas aleatoriamente.

Aquando de um pedido do cliente à réplica, este é enviado para um Frontend onde existe um canal de comunicação GRPC entre o cliente e a réplica, sendo através deste enviado o pedido e recebida a respetiva resposta. Quando a resposta for recebida, esta é enviada pelo frontend para o cliente. Para isto, são utilizados canais bloqueantes (GRPCBlockingStub) que permitem fazer a ligação entre o cliente e o servidor, de forma a ser possibilitar a troca de pedidos e respostas.  

_(descrição das trocas de mensagens)_


## Opções de implementação

Decidimos implementar 3 réplicas de servidores de forma a suportar 1 eventual falta. Assim, resultam de 2*f+1 = 3 réplicas, onde f é o número de faltas suportadas.

_(Descrição de opções de implementação, incluindo otimizações e melhorias introduzidas)_


## Notas finais

_(Algo mais a dizer?)_

