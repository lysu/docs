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


