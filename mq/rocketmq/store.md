# rocketmq store

#### `CommitLog#putMessage` to save

- set topic to schedule topic if message is not-tx or tx commit with delay
- try to get lock or spin-lock
- get mappedFile, if not exists or full create new one
- append message to mappedFile, full to get next file and append again
- release lock
- sync or async flush/commit base on configuration
- sync slave if we are master, wait based on configuration.

*Detail*: `isWarmMapedFileEnable`, `mlock`, `munlock` operation.

#### `getLastMappedFile` get or create mappedfile

- get mappedfile from end of list
- create new one when last is full or not exists
- create mappedfile by `allocateMappedFileService` or `new`
- `allocateMappedFileService` with pre-create and async-create feature.

#### `appendMessage` and its callback handle

do write opteration in callback.

*Caution*: for `prepare` and `rollback` operation, `queueOffset` is `0` and no need to increase queueOffset, because they are unable to be consumed.

#### `commit` to filechannel

`async` with `writebuffer` situation, first write and `commit` to `writebuffer` then `flush` to file(file-channel).

In `CommitRealTimeService`, uncommited over `commitDataLeastPages` or time over `commitDataThoroughInterval` to start commit operation.

#### `flush` to file

###### `FlushRealTimeService`

`async` flusher...do flush when isFull or unflush data more than `flushLeastPages` or interval is meeted.


###### `GroupCommitService`

- use in `sync` situation...
- double buffer(r/w)
- `client` wait wakeup operation triggered after flush if `isWaitStoreMsgOK ` is setted
- loop twice for  message over two file??what about 3??

