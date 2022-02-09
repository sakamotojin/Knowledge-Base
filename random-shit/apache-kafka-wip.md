# Apache Kafka\[WIP]

### Terminologies&#x20;

**Message     :**       An Piece of Information / Data . Eg. Logs , Events&#x20;

**Producer**    :     An Application that sends message to Kafka Server.

**Consumer**  :      An Application that requests data from Kafka Server.

**Broker          :**        Broker is Just A Meaningful Name given to Kafka Server.(As The Producer And         Consumer  Don't Exchange Message Directly rather use Kafka Server As A Broker )

**Cluster**   :  A Cluster Is A Group Of Computers^ Sharing Workload For A Common Purpose And Can Be Viewed As A Single System. In case Of Kafka They Are Group Of Kafka Brokers .

**Topic**   :  A Unique Name For A Data Stream. Producers Send Data To A Topic Which Can Be Subscribed By Consumers .

**Partition** : A Partition Is A Part Of A Topic . Since A Topic can grow significantly large such that A Single Broker Cannot&#x20;
