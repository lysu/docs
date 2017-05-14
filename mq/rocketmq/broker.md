# rocketmq broker

#### `SendMessageProcessor#processRequest`

recieve message from client.

##### recieve message

- parse message at first and call hook
- check start-ts inf config, check perm, check inner topic name(defaultTopic),
- find topic config in localcache or create new topic inherited from default config if `autoCreateTopicEnable` in broker and also persist topic conf and register itself.(also direct create and register retry topic)
- check request queue id not overflow
- random a queue if queue if queueid < 0
- check retry- message, put into dead-letter topic if retry more than subscription conf.
- reject tx message if not support tx.
- save message to store.

##### comsumer back(CONSUMER_SEND_MSG_BACK)

consumer consume a message failured will send back a message(offset,group, topic) to broker.

broker will find oldMessage and reput into commit with RETRY- topic

#### Consumer Queue

###### `ReputMessageService` 

To dispatch commit message to consumer queue

- `doreput` as 1ms interval
- read commitlog from `reputFromOffset`
- `doDispatch` to consumerQueue for non-tx or tx-commit
- dispatch to indexservice too.
- notify `messageArrivingListener` if not slave and long-poll enable
- normal shutdown will retry to reput to newest commitlog offset

###### `ConsumerQueue#putMessage`

- retry until put sucess
- `flushConsumeService` interval run, flush LeastPages or interval
- shutdown will force flush
- flush savepoint too

#### Pull Message

PullMessage header: consumeGroup topic, queueId, queueOffset, subscription-exp, maxMsgNums

##### `PullMessageProcessor#processRequest`

###### 1) check request

- check has permission to read this broker
- find subscription info for consumeGroup and check whether it be consumable
- find request topic's config and check queueId
- check subscription-exp, check consumeGroupInfo, check message model for consumeGroup, 

###### 2) get message and response

- get messages by `MessageStore#getMessag`
- mapping message statue to response code
- send message by heap or sendfile
- hold on if no message and need wait.
- upload consume offset to broker

response: will give a suggestion for client to using what broker in next turn.


##### `MessageStore#getMessage` details

- using group, topic, queueId, queueOffset, msgNums, subscriptionData as param.
- find consume queue by topic and queueId
- check offset in queue's offset range
- loop to fetch consumer queue and query commitlog to return batch

#### OffsetManager

save in `config/consumerOffset.json` and persist per 5s

offset is store in master broker just as a map with persist file.




