# 02 Work Queue

https://www.rabbitmq.com/tutorials/tutorial-two-python.html

## Round-robin dispatching

RabbitMQ のデフォルトでは，
複数の subscriber が同じ queue に subscribe している際，
publish されたメッセージは全ての subscriber に均等に配信される．

### 所感

クライアントごとに queue を用意する場合は，この辺りの設定は必要なさそう．
何か明示的に管理する必要がある？

## Message acknowledgment

subscriber がメッセージを受け取ったことを publisher に伝えるために，
subscriber はメッセージを受け取った後に ack を送信する必要がある．
ack が送信されずにタイムアウトした場合，RabbitMQ はメッセージを再度配信する．
この時，他の subscriber がオンラインの場合は，その subscriber にメッセージが配信される．

メッセージ受け取り直後に ack を送信する場合は `channel.basic_consume` のパラメータに `auto_ack=True` とすることで自動的に ack を送信することができる．

メッセージ受け取り後，何らかの処理を行った後に ack を送信する場合は，`auto_ack=False` とする．
callback 関数内で `channel.basic_ack(delivery_tag=method.delivery_tag)` を実行することで，ack を送信することができる．

### 所感

ここはエラーハンドリングの際に考えることになりそう．

## Message durability

RabbitMQ はメッセージをメモリ上に保持している．
そのため，RabbitMQ がクラッシュした場合にはメッセージが失われてしまう．

queue の durability を保証するためには，`queue_declare` に `durable=True` を指定する必要がある．

メッセージの durability を保証するためには，`basic_publish` に `delivery_mode=pika.spec.PERSISTENT_DELIVERY_MODE` を指定する必要がある．

### 所感

サーバサイドに DB を用意する際は，メッセージの durability は保証しなくても良いのではないかと思った．

### Fair dispatch

queue に大量のメッセージが溜まっている場合，RabbitMQ はデフォルトでは，メッセージを一気に subscriber に配信する．
このような挙動を避けるためには，`basic_qos` に `prefetch_count=1` を指定する必要がある．

### 所感

qos 　周りを真面目に考える必要がありそう．
