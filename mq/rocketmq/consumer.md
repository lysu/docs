# rocketmq consumer

#### Relance queue and consumer

use pull internal

pullMessageService => (ProcessQueue, ConsumeMessageService) 


subscribe topic and subscribte-tags to rebalanceService.

registerMessageListener to consumer??

`RebalanceService` run with 20s interval or PushConsumer start or Consumer be removed or added in broker.

- get all message queue for topic if broadcast mode, get part of queue if using cluster mode(allocateStrategy)
- after rebalance will trigger pull（PullMessageService#executePullRequestImmediately）


##### Pull message

###### `PullMessageService`

A thread just loop a request queue do pull..

and export some method let other can add request to queue.

do pull in `DefaultMQPushConsumerImpl#pullMessage`

- check processQueue length to flow control
- find a broker from broker list(maybe suggested by pre pull request)
- call broker to fetch on network(flow control)
- handle tags filter
- put to message queue and submit to consumeService request.


##### Consume message

##### `ConsumeRequest#run`

- transfrom back retry- topic to orign
- call listener
- send back message if cluster mode
- removeMessage if sucess and updateOffset(ack)
- clean timeout message in ProcessQueue in 15min interval and send back message to broker