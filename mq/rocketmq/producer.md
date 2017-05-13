# rocketmq producer

#### Start `MQProducer`

- check config just like topic name and so on.. - -
- set instance name to pid if not inner group
- get cached `MQClient` or create new one
- `MQClient` creative with `pull`, `rebalance` and so on service
- register producer in specify group in local memory
- start `MQClient` if needed, start underhook netty, pull, reblance, ns fetch threads in here.

#### Send with `MQProducer`

many `send` methods with different arguments and them divided by..

- async with `SendCallback` or sync
- specify target queue by `MessageQueue` or `MessageQueueSelector`
- automic select target queue by producer internal

so longest logic path is auto select queue one..just `DefaultMQProducerImpl#sendDefaultImpl`

###### 1) `tryToFindTopicPublishInfo` for topic

- first, get topic publish info from cache
- fetch topic publish info from ns
- return topic pub info if av
- try to use `TBW102` default topic to send and wait broker auto topic creation logic.

###### 2) `MQFaultStrategy#selectOneMessageQueue` for `topicPublishInfo`

- select queue from last used broker, and check broker's av
- select new broker from fault list(has problem but maybe ok - - now)
- select a queue without check broker av if all failed in above
- fault data is collect after every send, it's mainly about cal not av interval by latency.

###### 3) `sendKernelImpl` for man send logic.

- find broker address by brokerName of selected message queue.(first local cache, then remote if failure)
- set message id, deflate compress if message big then 4k, and mark compress sysflag
- set tx sysflag if it's transaction message.
- just send with `mqclient`(...later)

###### 4) with retry above(sync or async.)
