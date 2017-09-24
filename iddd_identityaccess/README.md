# iddd_identityaccess

- 基本設計はヘキサゴナルアーキテクチャ + CQRS(State Sourcing)
- 設計はagilepmに近い
  - ドメインモデルのメソッド内でDomainEventをpublishする
  - Springを使用しているため、DomainEventのsubscribeはAOPでapplication層で自動的に実行される(IdentityAccessEventProcessor)
  - ORMにHibernate
  - EventStoreにMySQL(HibernateEventStore)
    - DomainEventをシリアライズして保存
- サンプルの中で唯一REST API(コントローラ)を提供