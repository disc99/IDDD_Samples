# iddd_agilepm

- SpringなどのDIコンテナは未使用
- LevelDBをデータソースに使用
  - 自作のORM(Unit of Work)
  - ApplicationServiceLifeCycle.classでトランザクションコントロール
- 自作のSlothMQを使用(RabbitMQは使ってない?)
- 基本Write系処理
- 参照はIdなどを使用した取得処理(検索はない)
- Serviceの引数は基本クラスを保持するCommandオブジェクトが引数(ドメインオブジェクトは未使用)、返り値も同様
- アプリケーションコントロールにstaticメソッドやドメインモデル内でシングルトンなpublisherを使ったりとSpring環境で相性の悪そうな処理がある
- クラスの内部処理でもsetter,getterを使用かつ、バリデーションもsetter内にある

## 特徴
### イベントストア
- LevelDBEventStore(EventStoreの実装)
  - ドメインモデルなどからpublishされるイベント(DomainEvent)を保存
  - LevelDBのKVS機能を利用

### Notification
- LevelDBPublishedNotificationTrackerStore(PublishedNotificationTrackerStoreの実装)
  - Notification履歴を保存
- SlothMQNotificationPublisher(NotificationPublisherの実装)
  - 自作のMQ
- Storeに保存されているunpublishedなnotificationをSloth MQを通じてpublish

### ApplicationServiceLifeCycle
- AOPなどを使用していないため、トランザクションコントロールをこのクラスで管理
- トランザクション内のEventStoreの管理
  - EventStoreはLevelDBを使用
  - トランザクション中のDomainEventのサブスクライブ
  - EventStoreへの保存
  - NotificationApplicationService#publishNotificationsの定期実行
  

## アプリケーションライフサイクル
1. application.Service内でトランザクションを開始(同時にドメインイベントのsubscribeも開始)
1. 引数となるCommandからドメインモデルなどを作成しビジネスロジックを実行
1. 処理内容次第でドメインモデルからドメインイベントをPublish
1. ドメインイベントをSubscriberが受け取り、EventStoreに保存
1. トランザクション終了時に、commit or rollbackとトランザクション間で発生したドメインイベントを取得し、MQでNotificationとしてpublish
