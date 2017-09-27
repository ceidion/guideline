★：要検討

.. _ImplementOfUnitTest:

単体テストの実装
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

本節の実装例として記載しているサンプル及び、OSSライブラリ構成は一例である。
実際に採用される際には、業務要件に従って実装いただきたい。

|

単体テスト対象のクラス、試験方法及びその試験方法の詳細の一覧を以下に示す。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 25 35

    * - レイヤー
      - 試験対象
      - 試験方法
      - 詳細
    * - インフラストラクチャ層
      - Repository
      - spring-test
      - データアクセスにSpring JDBCを使用する場合
    * - 
      - 
      - spring-test + DBUnit + spring-test-dbunit
      - データアクセスにDBUnitを使用する場合
    * - ドメイン層
      - Repository + Service
      - spring-test
      - テスト済みのRepositoryを使用する場合。トランザクション境界の確認する
    * - 
      - Service
      - Junit + Mockito
      - モックを使用する場合
    * - アプリケーション層
      - Repository + Service + Controller
      - spring-test + MockMVC
      - モックを使用する場合
    * - 
      - Controller
      - JUnit + MockMVC + Mockito
      - モックを使用する場合
    * - 
      - Repository + Service + Helper
      - spring-test
      - データアクセスにSpring JDBCを使用する場合
    * - 
      - Repository + Service + Helper
      - spring-test + DBUnit
      - データアクセスにDBUnitを使用する場合
    * - 
      - Validation
      - JUnit
      - Bean Validationを使用する場合
    * - 
      - 
      - JUnit
      - Spring Validationを使用する場合

|

利用するOSSライブラリ
--------------------------------------------------------------------------------

OSSのバージョン★
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体テストで利用するOSS一覧を以下に示す。
なお、本章では全ての機能を網羅しているわけではないため、使用するOSSはあくまで1例である。
また、サンプルを動作させるために利用するOSS一覧については、\ :ref:`frameworkstack_using_oss_version`\ を参照されたい。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.27\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.13\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 27 25 15 13

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Spring IO platform
    * - Spring
      - org.springframework
      - spring-test
      - 4.3.5.RELEASE
      - \*
    * - Mockito
      - org.mockito
      - mockito-core
      - 1.10.19
      - \*
    * - JUnit
      - junit
      - junit
      - 4.12
      - \*
    * - SpringDBUnit
      - com.github.springtestdbunit
      - spring-test-dbunit
      - 1.3.0
      - \
    * - DBUnit
      - org.dbunit
      - dbunit
      - 2.5.4
      - \

|

ライブラリの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体テスト用のライブラリを\ ``pom.xml``\ の\ ``dependency``\ に追加を行う。以下に追加するライブラリを示す。

* ``pom.xml``

.. code-block:: xml

    <!-- == Begin Unit Test == -->
    <!-- (X) -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <!-- (X) -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
    </dependency>
    <!-- (X) -->
    <dependency>
      <groupId>org.DBUnit</groupId>
      <artifactId>DBUnit</artifactId>
      <version>2.X.X</version>
      <scope>test</scope>
    </dependency>
    <!-- (X) -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>X.X.X</version>
      ★現行ATRS（terasoluna-gfw-parent 5.2.0.RELEASE）の場合、1.10.19
      <scope>test</scope>
    </dependency>
    <!-- (X) -->
    <dependency>
      <groupId>com.github.springtestDbUnit</groupId>
      <artifactId>spring-test-dbunit★</artifactId>
      <version>1.3.0</version>
      <scope>test</scope>
    </dependency>
    <!-- == End Unit Test == -->

|

.. _SetUpOfTestingData:

テストデータのセットアップ
--------------------------------------------------------------------------------

ここではテストを実装する前段階として、テストデータについて説明する。
本章では、テストクラス実行時にテストデータをデータベース上に用意することを前提として、テスト用テーブルの作成方法および
テストデータの初期化方法について説明する。

.. _CreateTableforTest:

テスト用テーブルの作成方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストを実施するにあたり、データストアにデータベースを使用する場合、テスト用のデータベースのセットアップが必要になる。

テスト用のテーブルは、テスト用のコンテキストファイルに\ ``<jdbc:initialize-database>``\ を定義することで
テストクラス実行時にテスト用コンテキストファイルを読み込むことでテスト用のRDBMSのテーブル定義(DDL文)や
データ操作(DML文)を発行してデータベースを初期化することができる。
なお、\ ``<jdbc:initialize-database>``\ を使用して作成したテーブルと初期化データは実行後にコミットされるため、
テスト終了後もデータベースの状態は戻らないことに注意されたい。

設定例を以下に示す。

* ``test-context.xml``

.. code-block:: xml

  <!-- (1) -->
  <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${database.driverClassName}" />
    <property name="url" value="${database.url}" />
    <property name="username" value="${database.username}" />
    <property name="password" value="${database.password}" />
    <property name="defaultAutoCommit" value="false" />
    <property name="maxTotal" value="${cp.maxActive}" />
    <property name="maxIdle" value="${cp.maxIdle}" />
    <property name="minIdle" value="${cp.minIdle}" />
    <property name="maxWaitMillis" value="${cp.maxWait}" />
  </bean>

  <!-- (2) -->
  <jdbc:initialize-database data-source="dataSource">
    <jdbc:script location="classpath*:/META-INF/sql/test-schema.sql" />
  </jdbc:initialize-database>

  <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | データソースの実装クラスを指定する。
          例では、Apache Commons DBCPから提供されているデータソースクラス
          (\ ``org.apache.commons.dbcp2.BasicDataSource``\ )を指定する。
    * - | (2)
      - | 実行するSQLスクリプトの場所をscriptタグの\ ``location``\ 、SQLスクリプトファイルの文字コードを\ ``encoding``\ 
          に指定する。試験共通データがある場合、試験共通データ挿入用のDML文を指定することも可能である。


* ``RouteRepositoryTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class) // (1)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/test-context.xml" }) // (2)
    @Transactional
    public class RouteRepositoryTest {
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RunWith``\ に\ ``SpringJUnit4ClassRunner``\ を指定することによって、Spring固有のアノテーションを
          テストクラスで利用できる。
    * - | (2)
      - | \ ``@ContextConfiguration``\ アノテーションにテスト用の設定ファイルを指定することによって、テストを行う際は
          テスト用の設定ファイルを読み込むようにできる。classpathを指定することによって、resource直下を参照できる。

.. warning::

   \ ``<jdbc:initialize-database>``\ タグに設定するSQLスクリプトには、明示的に「COMMIT;」を記述すること。

テスト用データの追加方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テスト実行時にテストデータが必要な場合、クラスレベルまたはメソッドレベルで、\ ``@Sql``\ アノテーションを使用することで
テスト実行前にテストデータを追加・更新することができる。
なお、\ ``@Before``\ アノテーションを使用して、テスト実行前にテストデータを追加・更新する方法もあるが、ここでは
\ ``@Sql``\ アノテーションを使用した方法を説明する。

設定例を以下に示す。

* ``RouteRepositoryTest.java``

.. code-block:: java

    @Test
    @Sql("classpath:META-INF/sql/route-dataset.sql") // (1)
    public void testFindAll() {
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Sql``\ アノテーションをメソッドレベルで指定することによって、対象のテストメソッド実行前に
          \ ``@Sql``\ の引数に指定したSQLファイルが実行され、テストデータの追加・更新ができる。
        | なお、 \ ``@Sql``\ アノテーションをクラスレベルで指定した場合は、\ ``@Sql``\ アノテーションの指定のない
          テストメソッドすべてに対して適用される。

.. note:: **シーケンスの初期化方法**

    シーケンスは、トランザクションをロールバックしても進んだ値は戻らないという特徴を持つ。
    そのため、DBUnitでシーケンスから採番したカラムを持つレコードを検証する場合、シーケンスから採番したカラムは
    検証対象外とするか、以下のように明示的にシーケンスの初期化を行うSQLを実行し、テストの実施前に初期化する必要がある。

    * initialSequence.sql（PostgreSQLの例）
    
     .. code-block:: sql
     
        ALTER SEQUENCE SQ_MEMBER_1 RESTART WITH 1;

    * シーケンスの初期化

     .. code-block:: java

        @Inject
        private JdbcTemplate jdbcTemplate;

        @Test
        @Sql("classpath:META-INF/sql/initialSequence.sql")
        public void testUpdate() throws Exception {

            // シーケンスに依存した処理の呼び出し
        }

    * テストクラス内の全テストメソッドでシーケンスの初期化が必要な場合の共通化（PostgreSQLの例）

    テストクラス内の全テストメソッドでシーケンスの初期化が必要な場合、 クラスレベルに@Sqlを付与すると、
    @Sqlを付与していない各メソッドに対してシーケンスの初期化処理を呼び出すことができ、共通化できる。

     .. code-block:: java

        @Sql("classpath:META-INF/sql/initialSequence.sql")
        public class TicketReserveServiceImplTestInject {

            @Test
            public void testUpdate1() throws Exception {

                // シーケンスに依存した処理の呼び出し
            }
        }

|

インフラストラクチャ層の単体テスト
--------------------------------------------------------------------------------

インフラストラクチャ層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、インフラストラクチャ層の単体テストについて説明する。
インフラストラクチャ層の詳細については、開発ガイドラインの\ :ref:`LayerOfInfrastructure`\を参照されたい。

DBとのアクセス部分がインフラストラクチャ層のテストスコープとなる。
本節は、インフラストラクチャ層の\ ``Repository``\ クラスに対するテストの作成例を示す。

なお、MyBatis3を使用して\ ``Repository``\ を実装している場合、\ ``RepositoryImpl``\ はMapperインタフェース
（\ ``Repository``\）とマッピングファイルから自動生成される。
本節のテスト対象は正確には\ ``Repository``\ インタフェースではなく、自動生成された\ ``RepositoryImpl``\ となることに
注意すること。詳細は、\ :ref:`repository-mybatis3-label`\ を参照されたい。

インフラストラクチャ層のテスト対象のコンポーネントを以下に示す。

.. figure:: ./images/UnitTestLayerOfTestTargetRepository.png
   :width: 95%


Repositoryの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 30 40

    * - 使用するテストライブラリ(JUnit以外)
      - 説明
      - 使い分けの方針
    * - spring-test
      - Spring JDBCを使用してデータ操作を行う。
      - テストデータをsqlファイルで管理する場合
    * - spring-test + DBUnit + spring-test-dbunit
      - DBUnit、spring-test-dbunitの機能を使用してデータ操作を行う。
      - テストデータをExcelまたはCSVファイルで管理する場合

DBUnitは主にデータベースをセットアップする機能と、検証する機能を提供している。DBUnitを使用することで、
データ比較による結果の検証が効率的に実施できる。

spring-testを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Repositoryの単体テストは、JUnitを使用して実施する。
プロジェクト要件などでDBUnitが使用できない場合、\ ``org.springframework.jdbc.core.JdbcTemplate``\ を用いて
データアクセスを行う。
また、Repositoryの単体テストを行う際は単体テスト用の設定ファイルを用意すること。

データのセットアップを行う方法については、\ :ref:`SetUpOfTestingData`\ を参照されたい。

作成するファイル例を以下に示す。

.. figure:: ./images/UnitTestRepositorySpringTestItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - ReservationRepositoryTest.java
      - ReservationRepository.javaのテストクラス。
    * - test-context.xml
      - spring-testを使用したRepositoryの単体テストを行う際に使用する設定ファイル。
        本節で説明する内容はあくまで参考例のため、業務要件に合わせて設定ファイルを用意すること。

.. _TestGuideSettingOfSpringTest:

spring-testを使用するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Repositoryの単体テストのための設定ファイルとして  \ ``test-context.xml``\ を作成する。

* ``test-context.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:tx="http://www.springframework.org/schema/tx"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation=
           "http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-3.0.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">


      <!-- (1) -->
      <context:property-placeholder location="classpath*:/META-INF/spring/*.properties" />

      <bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${database.driverClassName}" />
        <property name="url" value="${database.url}" />
        <property name="username" value="${database.username}" />
        <property name="password" value="${database.password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="maxTotal" value="${cp.maxActive}" />
        <property name="maxIdle" value="${cp.maxIdle}" />
        <property name="minIdle" value="${cp.minIdle}" />
        <property name="maxWaitMillis" value="${cp.maxWait}" />
      </bean>

      <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
        <constructor-arg index="0" ref="realDataSource" />
      </bean>

      <!-- (2) -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource" />
          <property name="configLocation" value="classpath:/META-INF/mybatis/mybatis-config.xml" />
      </bean>

      <!-- (3) -->
      <mybatis:scan base-package="jp.co.ntt.atrs.domain.repository" />

      <!-- (4) -->
      <bean class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg ref="dataSource" />
      </bean>
      <bean class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg ref="dataSource" />
      </bean>

      <!-- (5) -->
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
      </bean>

      <!-- (6) -->
      <tx:annotation-driven />

      <!-- (7) -->
      <context:component-scan base-package="jp.co.ntt.atrs.domain.repository" />

    </beans>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | プロパティファイルを読み込む。
        | Bean定義ファイルに ``<context:property-placeholder/>`` タグを定義することで、
          JavaクラスやBean定義ファイル内でプロパティファイル内の値にアクセスできるようになる。
    * - | (2)
      - | \ ``SqlSessionFactory`` \を生成するためのコンポーネントとして\ ``org.mybatis.spring.SqlSessionFactoryBean`` \
          をBean定義する。
    * - | (3)
      - | MyBatisがマッパーを自動スキャンするパッケージを設定。
        | Repositoryのメソッドが呼び出されるとマッパーのSQLが実行される。
    * - | (4)
      - | \ ``org.springframework.jdbc.core.JdbcTemplate``\ クラスをBean定義する。
    * - | (5)
      - | \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager`` \クラスをBean定義する。
          \ ``dataSource`` \プロパティには、設定済みのデータソースのbeanを指定する。
    * - | (6)
      - | \ ``<tx:annotation-driven>``\ を追加することで、\ ``@Transactional``\ アノテーションを使った
          トランザクション境界の指定が有効となる。
    * - | (7)
      - | \ ``jp.co.ntt.atrs.domain.repository``\ パッケージ配下をcomponent-scan対象とする。

.. _ImplementOfRepositoryTest:

Repositoryテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Repositoryの単体テストクラスの作成方法を説明する。
テストテーブルのセットアップ、データの初期化は設定ファイル読み込み時に

* ``ReservationRepositoryTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/test-context-ReservationRepositoryTest.xml" })
    @Transactional // (1)
    public class ReservationRepositoryTest {

        @Inject
        ReservationRepository target; // (2)

        @Inject
        JdbcTemplate jdbctemplate; // (3)

        // ommited
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@Transactional``\ アノテーションを付与する。
        | クラスレベルに\ ``@Transactional``\ アノテーションを付与することで、トランザクション境界が各テストメソッドの前
          に移動するため、テスト終了時にロールバックされるようになる。
          これによって、テストの実行によるDBの内容の変更を防ぐことができる。
    * - | (2)
      - | 試験対象のクラスをインジェクションする。
        | 試験対象である\ ``ReservationRepository``\ クラスをインジェクションする。
    * - | (3)
      - | \ ``JdbcTemplate``\ クラスをインジェクションする。
        | \ ``JdbcTemplate``\ とはSpring JDBCサポートのコアクラスである。JDBC APIではデータソースからコネクションの取得、
          PreparedStatementの作成、ResultSetの解析、コネクションの解放などを行う必要があるが、\ ``JdbcTemplate``\ 
          を使うことでこれらの処理の多くが隠蔽され、より簡単にデータアクセスを行うことができる。
          DBUnitを使用しない場合は、\ ``JdbcTemplate``\ を使用してテストデータの投入を行うことを推奨する。


.. note:: **テスト用のトランザクション制御**

    \ ``@Sql``\ を使用してテストデータをセットアップする場合、デフォルトではテストデータをセットアップする際の
    トランザクションと、テストメソッド実行でデータアクセスする際のトランザクションは別々となる。
    テストデータをセットアップした後に一度コミットが行われ、テストメソッド実行後にデータアクセスがある場合は
    もう一度コミットが行われる。
    そのため、タイミングによってはテストメソッド実行前とデータベースの状態が変わっている可能性があることに注意されたい。
    
    なお、\ ``@Transactional``\ を付与することで、同一トランザクション内でテストデータのセットアップと
    テストメソッド実行を行うことができる。
    \ ``@Transactional``\ はデフォルトでテストメソッド実行後にロールバックされる。
    \ ``@Transactional``\ をクラスレベルで指定すると、指定したテストクラス全てのテストメソッドに対して
    トランザクション境界をテストメソッド単位に移動することができる。

.. note:: **ロールバックを実施しない場合について**

    ロールバックをしないようにするには、\ ``@TransactionConfiguration``\ アノテーションのオプションで
    \ ``defaultRollback=false``\ を与えるか、テストメソッドへ明示的に\ ``@Rollback(false)``\ のように
    アノテーションでロールバックを行わないことを記す必要がある。
    
    注意点としては、テストメソッドがロールバックを行わない設定になっているとテストが失敗した場合でも
    トランザクションがコミットされてしまう。中途半端なデータをDBに残してしまうことがあるので、
    どうしてもGUIツールなどでテーブルの中身を確認する必要がある場合のみ使用すること。


.. warning:: **@Rollbackと@TransactionConfigurationについて**

    Macchinettaオンライン 1.2版よりクラス単位で\ ``@Rollback``\ の設定が可能となった。
    これに伴い\ ``@TransactionConfiguration``\ が非推奨となった。但し、Macchinettaオンライン 1.1版以前では
    \ ``@Rollback``\ はメソッド単位にのみ設定が可能であり、クラス単位でロールバックの設定をする場合は
    \ ``@TransactionConfiguration(defaultRollback = true)``\ を設定する必要がある。

|

.. note:: **JdbcTemplateの使い方(INSERT/UPDATE/DELETE文)**

    JdbcTemplateにて、INSERT/UPDATE/DELETE文を発行する際はupdateメソッドを使用する。
    INSERT/UPDATE/DELETE文はいずれも更新系のSQLなので、1つのメソッドに集約されている。
    メソッド名の「update」は、UPDATE文を意味するわけではないので、注意すること。
    使用法としては、第1引数にSQL文を指定し、第2引数以降にパラメータの値を指定すること。

|

テストメソッドの作成例を以下に示す。

* ``ReservationRepositoryTest.java``

.. code-block:: java

    package jp.co.ntt.atrs.domain.repository.reservation;

    @Test
    public void insertTest() {

        // (1)
        Reservation reservation = new Reservation();
        reservation.setReserveNo("0000000001");

        // omitted

        Member member = new Member();
        member.setMembershipNumber("0000000001");
        reservation.setRepMember(member);


        // (2)
        int actInsert = target.insert(reservation);

        // (3)
        assertEquals(actInsert, 1);

        assertDB(reservation.getReserveNo(), reservation);
    }
    
    private void assertDB(String reserveNo, Reservation exReservation) {

        Reservation actReservation = getReservation(reserveNo);

        assertEquals(actReservation.getReserveNo(), exReservation
                .getReserveNo());

        // omitted
    }

    private Reservation getReservation(String reserveNo) {

        // (4)
        String sql = "SELECT * FROM reservation WHERE reserve_no=?";
        Reservation reservation = (Reservation) jdbctemplate.queryForObject(sql,
                new Object[] { reserveNo }, new RowMapper<Reservation>() {

                    // (5)
                    public Reservation mapRow(ResultSet rs,
                            int rowNum) throws SQLException {

                        Reservation dbReservation = new Reservation();

                        dbReservation.setReserveNo(rs.getString("reserve_no"));

                        // omitted

                        Member member = new Member();
                        member.setMembershipNumber(rs.getString(
                                "rep_customer_no"));
                        dbReservation.setRepMember(member);

                        return dbReservation;
                    }
                });

        return reservation;
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テスト対象メソッドを実行するためのテストデータを作成する。
    * - | (2)
      - | テスト対象メソッドを実行する。
    * - | (3)
      - | 更新件数を確認する。
    * - | (4)
      - | テスト対象メソッド実行後のテストデータを取得し、データが挿入されていることを確認する。
    * - | (5)
      - | RowMapperを使用することで、DBから取得した\ ``ResultSet``\ を特定のPOJOクラス（\ ``Member``\クラスと
          \ ``Reservation``\ クラス）にマッピングすることができる。


spring-testとDBUnitを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

データアクセスにDBUnitを使用する場合のRepositoryの単体テスト実装方法について説明する。

DBUnitとは、データベースに依存するクラスのテストを行うためのJUnit拡張フレームワークである。
テスト実行後のデータベースの状態の検証機能を使用することで試験工数を削減できるため、基本的にはDBUnitを用いて
実装することを推奨する。

データのセットアップはDBUnitが提供している機能を用いて行う。

DBUnitを利用したRepositoryの単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/UnitTestRepositoryDbunitItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - MemberRepositoryTest.java
      - MemberRepository.javaのテストクラス(DBUnitと連携する場合)
    * - test-context-MemberRepositoryTest.xml
      - Repositoryの単体テストを行う際に使用する設定ファイル(DBUnitと連携する場合)
    * - test_data_member.xml
      - テストデータセットアップ用ファイル
    * - afterupdate_data_member.xml
      - テストの期待結果検証用ファイル

.. _TestGuideSettingOfDbUnit:

DBUnitを使用するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

RepositoryのDBUnitを利用した単体テストのための設定ファイルとして \ ``test-context-dbunit.xml``\ を作成する。
\ :ref:`TestGuideSettingOfSpringTest`\ で作成したファイルに
\ ``org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy``\ のBean定義を追加する

* ``test-context-dbunit.xml``

.. code-block:: xml

  <!-- (1) -->
  <bean id="realDataSource" class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">
    <constructor-arg index="0" ref="log4jdbc" />
  </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | データソースのクラスを\ ``TransactionAwareDataSourceProxy``\ のbeanにすることで、
           DBUnitをSpringのトランザクション管理下にすることができる。

Repositoryテストの実装(DBUnitと連携する場合)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

* ``RouteRepositoryDbUnitTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/test-context-MemberRepositoryTest.xml" }) // (1)
    @TestExecutionListeners({                                                    // (2)
            DependencyInjectionTestExecutionListener.class,                      // (3)
            DirtiesContextTestExecutionListener.class,                           // (4)
            TransactionDbUnitTestExecutionListener.class,                        // (5)
            SqlScriptsTestExecutionListener.class })                             // (6)
    @Transactional
    public class MemberRepositoryTest {

        @Inject
        MemberRepository target;

        @Inject
        JdbcTemplate jdbctemplate;

         // omitted
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ :ref:`TestGuideSettingOfDbUnit`\ で作成した設定ファイルを読み込む。
          
    * - | (2)
      - | テストクラスに\ ``@TestExecutionListeners``\ を付与し、テスト実行関連のイベントに対するリスナを
          追加することで、テスト実行関連のイベントを捕捉出来る。
    * - | (3)
      - |  \ ``DependencyInjectionTestExecutionListener``\ は、テストインスタンスのDI機能を提供する。
    * - | (4)
      - | \ ``DirtiesContextTestExecutionListener``\ は、\ ``@DirtiesContext``\ アノテーションを処理する機能を
          提供する。\ ``@DirtiesContext``\ は、コンテキストのキャッシュを破棄、リロードする機能を提供する。
          詳細は、\ `@DirtiesContext <https://docs.spring.io/spring/docs/current/spring-framework-reference/html/integration-testing.html#__dirtiescontext>`_\
          を参照されたい。
    * - | (5)
      - | \ ``TransactionDbUnitTestExecutionListener``\ は、同一トランザクション内でBUnitによるデータセットアップや
          期待する結果の検証を行う機能を提供する。
    * - | (6)
      - | \ ``SqlScriptsTestExecutionListener``\ は、\ ``@Sql``\ アノテーションで設定されたSQLスクリプトを実行する
          機能を提供する。

テストメソッドの作成例を以下に示す。


* ``RouteRepositoryDbUnitTest.java``

.. code-block:: java

    @Test
    @DatabaseSetup("classpath:META-INF/dbunit/test_data_member.xml") // (1)
    @ExpectedDatabase( // (2)
            value = "classpath:META-INF/dbunit/afterupdate_data_member.xml", 
            assertionMode = DatabaseAssertionMode.NON_STRICT)
    public void updateTest() {

        String customerNo = "0000000001";
        Member member = createMember(customerNo);
        member.setKanjiFamilyName("電信柱");

        int actUpdate = target.update(member);

        assertEquals(actUpdate, 1);
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | spring-test-dbunitが提供する\ ``@DatabaseSetup``\ アノテーションにテストセットアップ用データファイルを
          指定することで、テストメソッド実行前にDBUnitによって自動でデータベースのセットアップが行われる。
        | 例のようにメソッドレベルにアノテーションを付与した場合、対象のテストメソッドに対してのみ有効になる。
          クラスレベルに付与すると、対象のテストクラスに含まれる全てのテストメソッドで設定が有効になる。
    * - | (2)
      - | \ ``@ExpectedDatabase``\ アノテーションにテストの期待結果検証用ファイルを指定することでテストメソッド
          実行後にDBUnitによってテーブルと期待結果データファイルが自動で比較検証される。
        | \ ``@DatabaseSetup``\ アノテーション同様に、クラスレベルとメソッドレベルで付与できる。
        | ファイルフォーマットはテストセットアップ用データファイルと同じである。\ ``assertionMode``\ 属性には、
          以下の値が設定可能である。

        * DEFAULT：全てのテーブルとカラムの一致を比較する
        * NON_STRICT：期待結果データファイルに存在しないテーブル、カラムが実際のデータベースに存在しても無視する
        * NON_STRICT_UNORDERED：NON_STRICTモードに加え、行の順序についても無視する

* テストセットアップ用データファイルの作成

試験前提条件データファイルは、FlatXMLと呼ばれる以下のフォーマットで作成する。

.. code-block:: xml

    <?xml version='1.0' encoding='UTF-8'?>
    <dataset>
        <!-- (1) -->
        <MEMBER CUSTOMER_NO="0000000001" KANJI_FAMILY_NAME="電電" KANJI_GIVEN_NAME="花子" KANA_FAMILY_NAME="デンデン" KANA_GIVEN_NAME="ハナコ" BIRTHDAY="1979-01-25" GENDER="F" TEL="111-1111-1111" ZIP_CODE="1111111" ADDRESS="東京都港区港南Ｘ－Ｘ－Ｘ" MAIL="xxxxxx@ntt.co.jp" CREDIT_NO="1111111111111111" CREDIT_TYPE_CD="VIS" CREDIT_TERM="01/20" />
        <MEMBER_LOGIN CUSTOMER_NO="0000000001" PASSWORD="$2a$10$AUvby7NA4i5MpFbks.lYd.pgUCv7Ze32FdnQFE03N4EeEePqVAH0C" LAST_PASSWORD="$2a$10$bJ8HB/5LaMN/ntOQHpgiAu8tfG1Y/rP21MaoK4RBenghxcbhrLW5C" LOGIN_DATE_TIME="2017-09-13 16:47:04.283" LOGIN_FLG="FALSE" />
    </dataset>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``dataset``\ 要素配下の各XML要素は、テーブルのレコードに対応しており、各XMLの要素名はテーブル名、
          属性名はカラム名、属性値は投入するデータを定義する。

.. warning:: **外部キー制約のあるテーブル**

    外部キー制約のあるテーブルに対し、DBUnitを用いてDBの初期化をすると、参照条件によってはエラーが発生するため、
    参照整合性を保つようにデータセットの順序を指定する必要があることに注意されたい。

|

.. note:: **DBUnitのExcelバージョンについて★★**

    DBUnitでは、FlatXML以外にExcel形式（.xlsx）またはCSV形式のデータ定義ファイルをテストデータや期待結果データとして
    用いることが出来る。

    spring-test-dbunitでは、データ定義ファイルの読込機能を\ ``com.github.springtestdbunit.dataset.DataSetLoader``\
    というインタフェースを実装したクラスに委譲しており、Excel形式やCSV形式のデータ定義ファイル読込ロジックを定義した
    \ ``DataSetLoader``\ を実装し、spring-test-dbunitが利用するように設定すれば実現できる。
    詳細は、\ `Spring Test DBUnit <http://springtestdbunit.github.io/spring-test-dbunit/>`_\ を参照されたい。

    以下、Excel形式の場合の実装例を示す。★記載する？

    * XlsDataLoaderの実装

    spring-test-dbunitが提供する抽象基底クラスである\ ``com.github.springtestdbunit.dataset.AbstractDataSetLoader``\ を
    利用して、以下のようにExcel形式のデータ定義ファイルの\ ``XlsDataSetLoader``\ を定義する。

     .. code-block:: java

        public class XlsDataSetLoader extends AbstractDataSetLoader {

            @Override
            protected IDataSet createDataSet(Resource resource) throws Exception {
                try(InputStream inputStream = resource.getInputStream()){
                    return new XlsDataSet(inputStream);
                }
            }
        }


    * 単体テスト用設定ファイルへのBean定義の追加

    以下のBean定義を、単体テスト用設定ファイルに追記する。 
    spring-test-dbunitは\ ``dbUnitDataSetLoader``\ というbean名のBean定義をルックアップしてデータ定義ファイルの読込に使用する。

     .. code-block:: xml

        <bean id="dbUnitDataSetLoader" class="<パッケージ名>.XlsDataSetLoader" />

    * Excel形式のデータ定義ファイルの作成

     .. figure:: ./images/UnitTestExcelFile.png
        :width: 70%

    Excel形式のデータ定義ファイルでは、各シートが各テーブルに対応する。
    シート名にはテーブル名、シートの一行目にはカラム名を設定する。 二行目以降にテーブルに挿入されるデータを記述する。

|

ドメイン層の単体テスト
--------------------------------------------------------------------------------

ドメイン層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、ドメイン層の単体テストについて説明する。
ドメイン層の詳細については、開発ガイドラインの\ :ref:`LayerOfDomain`\ を参照されたい。

業務ロジックや、CRUD操作についての部分がドメイン層のテストスコープとなる。
本節は、ドメイン層の\ ``ServiceImpl``\ クラスに対するテストクラスの作成例を示す。

ドメイン層のテスト対象のコンポーネントを以下に示す。

.. figure:: ./images/UnitTestLayerOfTestTargetDomain.png
   :width: 95%


.. _UnitTestOfService:

Serviceの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 30 40

    * - 使用するテストライブラリ(JUnit以外)
      - 説明
      - 使い分けの方針
    * - spring-test
      - テスト済みのRepositoryを使用してServiceをテストする。
      - 依存クラスがテスト済みでモック化する必要がない場合
    * - Mockito
      - Repositoryをモック化してServiceをテストする。
      - 依存クラスのモック化が必要な場合

Serviceの単体テストについては、JUnitを使用して\ ``Service``\ クラスの実装クラス（\ ``ServiceImpl``\）に対して
試験を実施する。テスト対象の\ ``ServiceImpl``\ クラスがテストを実施していないクラスを
インジェクションしている場合はモックを作成すること。
モックの作成方法については、\ :ref:`TestingServiceWithSpringTest`\ を参照されたい。

なお、インジェクションするクラスにモッククラスを別途用意してもよい。
モッククラスの作成方法については、本ガイドラインでは説明を割愛する。

モッククラスを作成せず、モック用ライブラリを使用する方法については、\ :ref:`TestingServiceWithMockito`\を
参照されたい。

.. _TestingServiceWithSpringTest:

spring-testを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

テスト済みの\ ``Repository``\ クラスを使用する場合、DBUnitを使用して\ ``Repository``\ クラスをインジェクションして
テスト対象の\ ``ServiceImpl``\ クラスのテスト作成方法を説明する。

作成するファイルを以下に示す。

.. figure:: ./images/UnitTestServiceSpringTestItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - XxxServiceImplTest.java
      - XxxServiceImpl.javaのテストクラス
    * - MessageSourceMock.java
      - Serviceの単体試験を行う際に使用するMessageSourceのモッククラス。

Serviceテストの実装(DBUnitと連携する場合)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Repositoryをインジェクションしてテストする方法は\ :ref:`ImplementOfRepositoryTest`\ を参照されたい。

.. _TestingServiceWithMockito:

JunitとMockitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Repository``\ クラスなど\ ``ServiceImpl``\ クラスが依存するクラスをモック化する場合のテスト作成方法を説明する。

作成するファイルを以下に示す。

.. figure:: ./images/UnitTestServiceMockItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - XxxServiceTest.java
      - XxxService.javaのテストクラス
    * - XxxMock.java
      - Serviceの単体試験を行う際に使用するXxxのモッククラス。

.. _ImplementOfServiceTest:

Serviceテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

・モッククラスの作成方法（Mockito）

・Serviceのテストクラス作成

|

アプリケーション層の単体テスト
--------------------------------------------------------------------------------

アプリケーション層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、アプリケーション層の単体テストについて説明する。
アプリケーション層の詳細については、開発ガイドラインの\ :ref:`LayerOfApplication`\ を参照されたい。

データの入出力、入力データの妥当性チェックがアプリケーション層のテストスコープとなる。
本節は、アプリケーション層の\ ``Controller``\ クラス、\ ``Helper``\ クラス、\ ``Form(Validation)``\ クラスに対する
テストクラスの作成例を示す。

なお、Viewについては単体テストの対象外とする。

アプリケーション層のテスト対象のコンポーネントを以下に示す。

.. figure:: ./images/UnitTestLayerOfTestTargetApplication.png
   :width: 95%


Controllerの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 30 40

    * - 使用するテストライブラリ(JUnit以外)
      - 説明
      - 使い分けの方針
    * - spring-test + mockMvc
      - テスト済みのService、Repositoryを使用してControllerをテストする。
      - 依存クラスがテスト済みでモック化する必要がない場合
    * - spring-test + mockMvc + mockito
      - Service、Repositoryをモック化してControllerをテストする。
      - 依存クラスのモック化が必要な場合


Springは\ ``Controller``\ クラスを試験するためのサポートクラス
(\ ``org.springframework.test.web.servlet.setup.MockMvcBuilders``\ など)を用意している。
これらのクラスを利用することでJUnitから\ ``Controller``\ クラスのメソッドを実行して試験をすることができる。

spring-test + MockMVCを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Controllerテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


spring-test + MockMVC + Mockitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Controller``\ がインジェクションしている\ ``Service``\ クラスはモック用ライブラリを使用する。
Serviceクラスがテスト済みの場合は、テスト済みのServiceクラスを使用する。

作成するファイルを以下に示す。

.. figure:: ./images/UnitTestControllerStandaloneSetupItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - XxxControllerTest.java
      - XxxController.javaのテストクラス
    * - XxxServiceImplMock
      - Controller,Formの単体テストを行う際に使用するServiceのモッククラス。

Controllerテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Serviceのモッククラスの作成方法については、\ :ref:`ImplementOfServiceTest`\ を参照されたい。

ここでは、Controllerの単体テストクラスの作成方法を説明する。

* ``MemberRegisterControllerTest.java``

.. code-block:: java

    public class MemberRegisterControllerTest {

        @InjectMocks
        MemberRegisterController target;

        MockMvc mockMvc;

        @Before
        public void setUp() throws Exception {

            // コントローラにモックをインジェクションする。
            // なお、Mockオブジェクトの初期化には以下の方法でも可能。
            // ・RunWith アノテーションに MockeitoJUnitRunner を指定する。
            // ・JUnit の MethodRule を実装した MockitoRule を使う。(JUnit4.7以降)
            MockitoAnnotations.initMocks(this); // 徹底入門スタイル (p.405参考)

            // 試験対象コントローラからMockMvcを生成する。
            this.mockMvc = MockMvcBuilders.standaloneSetup(target).build();
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 


* ``MemberRegisterControllerTest.java``

.. code-block:: java

    package jp.co.ntt.atrs.app.c1;

    @Test
    public void testRegisterForm() throws Exception {
        // Modelに格納するデータ
        String birthMinDate = "BirthMinDate";
        String birthMaxDate = "BirthMinDate";
        
        // Helperの動作を設定
        when(memberHelper.getDateOfBirthMinDate()).thenReturn(birthMinDate);
        when(memberHelper.getDateOfBirthMaxDate()).thenReturn(birthMaxDate);

        // テストを実行し、HTTPステータスコード、遷移先JSPパス、Modelの妥当性を検証
        ResultActions results = mockMvc.perform(
                MockMvcRequestBuilders.get("/member/register")
                .param("form", "form"))
                .andExpect(status().isOk())
                .andExpect(forwardedUrl("C1/memberRegisterForm"))
                .andExpect(model().attributeHasNoErrors(memberRegisterFormName));

        // Modelにオブジェクトが格納されていることを確認する。
        results.andExpect(model().attribute(birthMinDateObjectName, isA(String.class)));
        results.andExpect(model().attribute(birthMaxDateObjectName, isA(String.class)));

        // Modelに格納されたオブジェクトを取得し確認する。
        ModelAndView mav = results.andReturn().getModelAndView();
        String actualDateOfBirthMinDate = (String) mav.getModel().get(
                birthMinDateObjectName);
        String actualDateOfBirthMaxDate = (String) mav.getModel().get(
                birthMaxDateObjectName);
        assertThat(actualDateOfBirthMinDate, equalTo(birthMinDate));
        assertThat(actualDateOfBirthMaxDate, equalTo(birthMaxDate));
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 


.. note:: **@AuthenticationPrincipalアノテーションを利用している場合**

    コントローラのメソッドが\ ``@AuthenticationPrincipal``\ アノテーションが付与された引数を持つ場合、そのままでは
    試験できない。例えば以下のようなクラスは、テスト時にAtrsUserDetailsのインスタンスを生成するのに失敗してしまう。

    * \ ``@AuthenticationPrincipal``\ アノテーションを利用したメソッドの例

     .. code-block:: java

        @RequestMapping(method = RequestMethod.GET, params = "form")
        public String reserveForm(ReservationFlightForm reservationFlightForm,
                @AuthenticationPrincipal AtrsUserDetails userDetails, Model model) {

            // omitted
        }


    この場合は、setUpメソッドの中でMockMvcを生成する際に以下のメソッドを追加する。

    * テストコードの例

     .. code-block:: java

        @InjectMocks
        TicketReserveController target;

        @Before
        public void setUp() throws Exception {

            // omitted

            // 試験対象コントローラからMockMvcを生成する。
            mockMvc =
                    MockMvcBuilders
                            .standaloneSetup(target)
                            .setCustomArgumentResolvers(
                                    new AuthenticationPrincipalArgumentResolver())
                            .build();  // (1)
        }

     .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 10 90

         * - 項番
           - 説明
         * - | (1)
           - | \ ``setCustomArgumentResolvers``\ メソッドでリゾルバを設定する。
             | \ ``MockMvc``\ 生成時に\ ``setCustomArgumentResolvers``\ メソッドで
               \ ``org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver``\ 
               を設定する。 

|

.. note:: **Sessionを利用する場合**

    ControllerクラスがSessionを利用している場合は\ ``org.springframework.mock.web.MockHttpSession``\ を使って試験を行う。

    * \ ``MockHttpSession``\ を利用したテストメソッドの例

         .. code-block:: java

            @Test
            public void testSession() throws Exception {

                // (1)
                MockHttpSession mockSession = new MockHttpSession();
                mockSession.setAttribute("userId", "0001");

                // (2)
                MockHttpServletRequestBuilder getRequest = MockMvcRequestBuilders.get(
                    "/checkSession").session(mockSession);

                // (3)
                ResultActions results = mockMvc.perform(getRequest);
                
                // (4)
                results1.andExpect(request().sessionAttribute("userId", equalTo("0001")));
                
                // omitted
            }

         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
             :header-rows: 1
             :widths: 10 90

             * - 項番
               - 説明
             * - | (1)
               - | セッションのモックを生成し、オブジェクトを格納する。
             * - | (2)
               - | セッションを登録したリクエストのモックを生成する。
                 | \ ``org.springframework.test.web.servlet.request.MockMvcRequestBuilders``\ の\ ``get``\ メソッドで
                   リクエストのモックを生成し、生成したリクエストに\ ``session``\ メソッドでセッションのモックを
                   登録する。例では\ ``/checkSession``\へのGETリクエストにセッションのモックを登録している。
             * - | (3)
               - | \ ``MockMvc``\ にリクエストを渡してコントローラのメソッドを実行する。
             * - | (4)
               - | セッションに格納されていることを確認する。

|

Helperの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Helperの単体テストで、特別に意識すべきことはない。通常のPOJO(Plain Old Java Object)と同様にJUnitによる
単体テストを実施する。

実装方法については、\ :ref:`UnitTestOfService`\ を参照されたい。


Validatorの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 30 40

    * - 使用するテストライブラリ(JUnit以外)
      - 説明
      - 使い分けの方針
    * - -（JUnitのみ）
      - カスタムバリデーションをテストする。
      - BeanValidationを使用している場合
    * - -（JUnitのみ）
      - 相関項目チェックをテストする。
      - SpringValidationを使用している場合

JUnitを使用した試験（Bean Validation）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Validator(Bean Validation)の単体テストについては、JUnitを使用して試験を実施する。
カスタムバリデーションの試験を行う。HibernateValidatorが用意する入力チェックのアノテーションについては
フレームワーク側で担保しているので、単体テストを行う必要はない。

作成するファイルを以下に示す。

.. figure:: ./images/UnitTestBeanValidationItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - HalfWidthNumberTest.java
      - HalfWidthNumber.javaのテストクラス

Validator(Bean Validation)テストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Validator(Bean Validation)のテストクラスとして、\ ``HalfWidthNumberTest``\ を作成する。

* ``HalfWidthNumberTest.java``

.. code-block:: java

    public class HalfWidthNumberTest {

        private static Validator validator;

        @BeforeClass
        public static void setUpBeforeClass() throws Exception {
            ValidatorFactory validatorFacotry = Validation
                    .buildDefaultValidatorFactory();
            validator = validatorFacotry.getValidator();
        }

        @Test
        public void testValidate01() {

            String membershipNumber = "0123456789";

            PassengerForm form = new PassengerForm();

            // ダミー情報を設定
            form.setFamilyName("ミョウジ");
            form.setGivenName("ナマエ");
            form.setAge(20);
            form.setGender(Gender.F);
            // テスト対象のフィールドに正常値をセット
            form.setMembershipNumber(membershipNumber);

            Set<ConstraintViolation<PassengerForm>> violations = validator.validate(
                    form);

            // エラーがないことを確認
            assertEquals(violations.size(), (0));
        }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 

JUnitを使用した試験（Spring Validation）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Validator(Spring Validation)の単体テストについては、JUnitを使用して試験を実施する。
相関項目チェックの試験を行う。

作成するファイルを以下に示す。

.. figure:: ./images/UnitTestSpringValidationItems.png

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - ReservationFlightValidatorTest.java
      - ReservationFlightValidator.javaのテストクラス

Validator(Spring Validation)テストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Validator(Bean Validation)のテストクラスとして、\ ``ReservationFlightValidatorTest.java``\ を作成する。

* ``ReservationFlightValidatorTest.java``

.. code-block:: java

    public class ReservationFlightValidatorTest {

        private ReservationFlightValidator target;

        private ReservationFlightForm reservationFlightForm;

        private BindingResult result;

        @Before
        public void setUp() throws Exception {
            MockitoAnnotations.initMocks(this);

            target = new ReservationFlightValidator();
            reservationFlightForm = new ReservationFlightForm();
            result = new DirectFieldBindingResult(reservationFlightForm, "reservationFlightForm");
        }

        @Test
        public void testValidate04() {

            // ダミー情報を設定
            reservationFlightForm.setFlightType(FlightType.OW);
            reservationFlightForm.setSelectFlightFormList(
                    getSelectFlightFormList());

            // バリデータの実行
            target.validate(reservationFlightForm, result);

            // エラーがないことを確認
            assertEquals(result.hasErrors(), false);
        }
    }

