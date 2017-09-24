# iddd_collaboration

- 基本設計はヘキサゴナルアーキテクチャ + CQRS(Event Sourcing)
- WriteはEventStore(LevelDB)を使用したEvent Sourcing
  - Write系処理はserviceから値の返却は無い(void)。ただし、Idなどをmutableな引数を使用して返している部分もある
    ```java
    void createCalendar(/* params..., */ CalendarCommandResult aCalendarCommandResult) {
        
        // logic...
        
        aCalendarCommandResult.resultingCalendarId(calendar.calendarId().id());
    }
    ```
- ReadはMySQLを利用
  - EventStore(LevelDB)をイベントジャーナルにし、ドメインイベントをMySQLに反映させている
- WriteとReadは別データストアのため、一時的に不整合が発生する可能性がある(結果整合性)
- ユーザ情報は外部APIを使用して取得している

## 特徴
### Entityのライフサイクル
- Event Sourcingの実現方法
  - Entityクラスはスーパークラス(EventSourcedRootEntity)を継承している
  - EventSourcedRootEntityはEventStreamの管理とEntity内にあるSubscriber(#when)の呼び出しを行う
  - Entityの変更(mutation)はEntityが保持しているEventStreamにDomainEventを追加することによって行う
  - RepositoryなどからEntityを取得する場合には、EventStoreに保存されているEventStreamをコンストラクタに渡し、過去のDomainEventを再生することで復元する(所謂リプレイ)

### WriteモデルとReadモデルの連携
- FollowStoreEventDispatcher: 各MySQL ProjectionクラスをEventStoreに登録するクラス
  - Springで各DispatcherをDI後、各MySQL ProjectionにDIされる
  - EventStoreのappendWith呼び出し時に、DomainEventがProjectionにdispatchされる
- RepositoryでEventStoreにDomainEventを追加するときに、appendWithを呼び出す
- Write系の処理でトランザクションは使用していない?
  - 基本的にEventSourcedRootEntity単位で書き込みを行っているため?
