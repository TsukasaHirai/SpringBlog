# Springノート

## Spring共通

- @Autowired(オートワイド)

  DIのために使う。必須。

  ただし最近はコンストラクタインジェクションが推奨されている。

  - 機能

    new してくれる

  - 例

    - 通常

      ```java
      private SampleRepository repository = new SampleRepository();
      ```

    - 使用時

      ```java
      @Autowired
      private SampleRepository repository;
      ```

  - まみむめメモ

    インターフェースをnewする場合は便利　例：匿名クラスなど

  - コンストラクタインジェクション・フィールドインジェクション

    [2つの違い](https://qiita.com/kmuro/items/aead50c699fefe56c120)

    [テストが楽になる](https://qiita.com/yoshiyuki-iwata/items/aba17db40dc800869188)
    
    単体テストが楽になるためコンストラクタにつけてやるべき

- @Bean(ビーン)

  JavaBeansの略

  @Configurationがついているクラスでしか使えない

  - 機能

    Beanと書いたメソッドでインスタンス化されたクラスがシングルトンクラスとしてDIコンテナに登録される。任意のクラスでで注入してアクセスできる。
    
    @Component、@Service、@Repository、@Controllerをつけるクラスについては、このアノテーションをつけた時点でBeanとして登録されるため、@Beanを付与する必要はない。　例：JpaRepositoryとかはいらない
    
  - サンプル

    1. BaseBall.java(modelクラス)

       ```Java
       @Data
       @Entity
       public class BaseBall {
           @Id
           @GeneratedValue
           private String name;
       }
       ```

    2. BaseBallConfig.java(Configurationクラス)

       ```Java
       @Configuration
       public class BaseBallConfig {
           @Bean //ここでDIコンテナに登録
           public BaseBall createBaseBall {
               return new BaseBall();
           }
       }
       ```

    3. Main.java(実装クラス)

       ```Java
       public class Main {
           @Autowired　//ここで注入
           private BaseBall baseball;
       }
       ```

- @Service(サービス)

  Sping MVCでサービス層のクラス（ビジネスロジック等）に付与する。
  Service は業務処理を提供する。

  例：UserDetailsとかがこれ

- Repository()

  Spring MVCでデータ層のクラス（DAO等のDBアクセスを行うクラス）に付与する。

  例：Spring Data JPAがこれだがJpaRepository継承してるので暗黙的についてる

- @Component

  Spring MVCに限らず、SpringのDIコンテナにbeanとして登録したいクラスへ付与する。

  正直使い分けが難しい その他的なカテゴリー、CommandLinnerとかで使ってるけど



## Spring Batch

[Spring Batchのアーキテクチャ](https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/ja/Ch02_SpringBatchArchitecture.html#Ch02_SpringBatchArch_Overview_SpringBatch)

<img src="https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/ja/images/ch02/SpringBatchArchitecture/Ch02_SpringBatchArchitecture_Architecture_ProcessFlow.png" alt="Spring Batch Process Flow" style="zoom:50%;" />

1. ジョブスケジューラからJobLauncherが起動される （たぶんこれがJP1）
2. JobLauncherからJobを実行する。（シェルスクリプトからJava起動）
3. JobからStepを実行する。
4. StepはItemReaderによって入力データを取得する
5. StepはItemProcessorによって入力データを加工する。
6. StepはItemWriterによって加工されたデータを出力する



基本はこんな感じ

- ItemReaderインターフェース

  - FlatFileItemReaderクラス

    CSVファイルなどの、フラットファイル(非構造的なファイル)の読み込みを行う。Resourceオブジェクトをインプットとし、区切り文字やオブジェクトへのマッピングルールをカスタマイズすることができる。

  - StaxEventItemReaderクラス

    XMLファイルの読み込みを行う。名前のとおり、StAXをベースとしたXMLファイルの読み込みを行う実装となっている。

  - JdbcPagingItemReaderクラス JdbcCursorItemReaderクラス

    JDBCを使用してSQLを実行し、データベース上のレコードを読み込む。データベース上にある大量のデータを処理する場合は、全件をメモリ上に読み込むことを避け、一度の処理に必要なデータのみの読み込み、破棄を繰り返す必要がある。
    JdbcPagingItemReaderはJdbcTemplateを用いてSELECT SQLをページごとに分けて発行することで実現する。一方、JdbcCursorItemReaderはJDBCのカーソルを使用することで、1回のSELECT SQLの発行で実現する。
     TERASOLUNA Batch 5.xではMyBatisを利用することを基本とする。

  - MyBatisCursorItemReaderクラス MyBatisPagingItemReaderクラス

    MyBatisと連携してデータベース上のレコードを読み込む。MyBatisが提供しているSpring連携ライブラリMyBatis-Springから提供されている。PagingとCursorの違いについては、MyBatisを利用して実現していること以外はJdbcXXXItemReaderと同様。
    その他に、JPA実装やHibernateなどと連携してデータベース上のレコードを読み込むJpaPagingItemReader、HibernatePagingItemReader、 HibernateCursorItemReaderが提供されている。 TERASOLUNA Batch 5.xでは`MyBatisCursorItemReader`を利用することを基本とする。

  - JmsItemReaderクラス　AmqpItemReaderクラス

    JMSやAMQPからメッセージを受信し、その中に含まれるデータの読み込みを行う。
