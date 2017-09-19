★：要検討

単体テストの実装
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

.. _UnitTestOverview:

Overview ★節追加
--------------------------------------------------------------------------------

本節の実装例として記載しているサンプル及び、OSSライブラリ構成は一例である。
実際に採用される際には、業務要件に従って実装いただきたい。

|

単体テスト対象のクラス、試験方法及びその試験方法の詳細の一覧を以下に示す。

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 25 60

    * - 試験対象
      - 試験方法
      - 詳細
    * - Repository
      - spring-test
      - DB操作にSpring JDBCを使用する場合
    * - 
      - spring-test + DBUnit
      - DB操作にDBUnitを使用する場合
    * - Service
      - spring-test
      - モックを使用しない場合
    * - 
      - Junit + Mocito
      - モックを使用する場合
    * - Controller
      - spring-test + MockMVC + Mockito
      - モックを使用する場合
    * - Helper
      - spring-test
      - DB操作にSpring JDBCを使用する場合
    * - 
      - spring-test + DBUnit
      - DB操作にDBUnitを使用する場合
    * - Validation
      - JUnit
      - Bean Validationの場合
    * - 
      - JUnit
      - Spring Validationの場合

|

利用するOSSライブラリ
--------------------------------------------------------------------------------

OSSのバージョン★
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

本節で利用するOSS一覧を以下に示す。

.. tabularcolumns:: |p{0.15\linewidth}|p{0.27\linewidth}|p{0.25\linewidth}|p{0.15\linewidth}|p{0.05\linewidth}|p{0.08\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 15 27 25 15 5 8

    * - Type
      - GroupId
      - ArtifactId
      - Version
      - Spring IO platform
      - Remarks
    * - Spring
      - org.springframework
      - spring-test
      - 4.3.5.RELEASE
      - \*
      -
    * - SpringDBUnit
      - com.github.springtestdbunit
      - spring-test-dbunit
      - 1.3.0
      - \*
      -
    * - DBUnit
      - org.dbunit
      - dbunit
      - 2.5.4
      - \*
      -
    * - Mockito
      - org.mockito
      - mockito-core
      - 1.10.19
      - \*
      -
    * - JUnit
      - junit
      - junit
      - 4.12
      - \*
      -

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

.. _UnitTestOfInfrastructureLayer:

インフラストラクチャ層の単体テスト
--------------------------------------------------------------------------------

インフラストラクチャ層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、インフラストラクチャ層の単体テストについて説明する。
インフラストラクチャ層の詳細については、開発ガイドラインの\ :ref:`LayerOfInfrastructure`\を参照されたい。

DBとのアクセス部分がインフラストラクチャ層のテストスコープとなる。
本節は、インフラストラクチャ層の\ ``Repository``\ クラスに対するテストの作成例を示す。

なお、Macchinetta Server Framework 適用システムで、MyBatis3を使用して\ ``Repository``\ を実装している場合、
\ ``RepositoryImpl``\ はMapperインタフェース（\ ``Repository``\）とマッピングファイルから自動生成される。
本節のテスト対象は正確には\ ``Repository``\ インタフェースではなく、自動生成された\ ``RepositoryImpl``\ となることに
注意すること。

インフラストラクチャ層のテスト対象のコンポーネントを以下に示す。

.. figure:: ./images/UnitTestLayerOfTestTargetRepository.png
   :width: 95%


Repositoryの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60

    * - テストパターン
      - 特徴
      - 使い分けの方針
    * - spring-test
      - 基本??
      - DBUnitが使用できない場合（Spring JDBCを使用する場合）
    * - spring-test+DBUnit
      - 基本??
      - DBUnitを使用できる場合


Macchinetta Server Framework 適用システムで、MyBatis3を使用して\ ``Repository``\ を実装している場合、
\ ``RepositoryImpl``\ を実装する必要はない。
サンプルでは、\ ``Repository``\ インタフェースに対してテストを作成しているが、
MyBatis3によりMapperインタフェース（\ ``Repository``\）とマッピングファイルから自動生成された\ ``RepositoryImpl``\ が
テスト対象となることに注意すること。
詳細は、\ :ref:`repository-mybatis3-label`\ を参照されたい。


spring-testを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Repositoryの単体テストは、JUnitを使用して実施する。
プロジェクト要件などでDBUnitが使用できない場合、\ ``org.springframework.jdbc.core.JdbcTemplate``\ を用いて
データアクセスを行う。
また、Repositoryの単体テストを行う際は単体テスト用の設定ファイルを用意すること。

作成するファイル例を以下に示す。

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - XxxRepositoryTest.java
      - XxxRepository.javaのテストクラス
    * - test-context.xml
      - Repositoryの単体テストを行う際に使用する設定ファイル
    * - route-dataset.sql
      - テストで使用する初期データファイル
    * - schema.sql
      - テスト用のDDLファイル

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

      <!-- (2) -->
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

      <!-- (3) -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="typeAliasesPackage" value="jp.co.ntt.atrs.domain.model, jp.co.ntt.atrs.domain.repository" />
      </bean>

      <!-- (4) -->
      <mybatis:scan base-package="jp.co.ntt.atrs.domain.repository" />

      <!-- (5) -->
      <bean class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg ref="dataSource" />
      </bean>
      <bean class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
        <constructor-arg ref="dataSource" />
      </bean>

      <!-- (6) -->
      <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
      </bean>

      <!-- (7) -->
      <tx:annotation-driven />

      <!-- (8) -->
      <context:annotation-config />
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
      - | データソースの実装クラスを指定する。
          例では、Apache Commons DBCPから提供されているデータソースクラス
          (\ ``org.apache.commons.dbcp2.BasicDataSource``\ )を指定する。
        | データソースを定義する際に設定するドライバクラス名やURLなどの接続情報は、メンテナンス性向上のため
          プロパティファイルに定義すること。
    * - | (3)
      - | \ ``SqlSessionFactory`` \を生成するためのコンポーネントとして\ ``org.mybatis.spring.SqlSessionFactoryBean`` \
          をBean定義する。
    * - | (4)
      - | MyBatisがマッパーを自動スキャンするパッケージを設定。
        | Repositoryのメソッドが呼び出されるとマッパーのSQLが実行される。
    * - | (5)
      - | \ ``org.springframework.jdbc.core.JdbcTemplate``\ クラスをBean定義する。
    * - | (6)
      - | \ ``org.springframework.jdbc.datasource.DataSourceTransactionManager`` \クラスをBean定義する。
          \ ``dataSource`` \プロパティには、設定済みのデータソースのbeanを指定する。
    * - | (7)
      - | \ ``<tx:annotation-driven>``\ を追加することで、\ ``@Transactional``\ アノテーションを使った
          トランザクション境界の指定が有効となる。
    * - | (8)
      - | \ ``jp.co.ntt.atrs.domain.repository``\ パッケージ配下をcomponent-scan対象とする。

.. _ImplementOfRepositoryTest:

Repositoryテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Repositoryの単体テストクラスの作成方法を説明する。

* ``RouteRepositoryTest.java``

.. code-block:: java

    @RunWith(SpringJUnit4ClassRunner.class) // (1)
    @ContextConfiguration(locations = {
            "classpath:META-INF/spring/test-context.xml" }) // (2)
    @Transactional // (3)
    @Rollback // (4)
    public class RouteRepositoryTest {

        @Inject
        RouteRepository target; // (5)

        @Inject
        JdbcTemplate jdbctemplate; // (6)

        // ommited

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``@RunWith``\ アノテーションを付与する。
        | \ ``@RunWith``\ に\ ``SpringJUnit4ClassRunner``\ を指定することによって、Spring固有のアノテーションを
          テストクラスで利用できる。
    * - | (2)
      - | \ ``@ContextConfiguration``\ アノテーションを付与する。
        | \ ``@ContextConfiguration``\ アノテーションにテスト用の設定ファイルを指定することによって、テストを行う際は
          テスト用の設定ファイルを読み込むようにできる。classpathを指定することによって、resource直下を参照できる。
    * - | (3)
      - | \ ``@Transactional``\ アノテーションを付与する。
        | テストクラスに\ ``@Transactional``\ アノテーションを宣言することで、テストクラスが持つテストメソッドは
          トランザクション制御の対象となる。
    * - | (4)
      - | \ ``@Rollback``\ アノテーションを付与する。
        | テストクラスに\ ``@Rollback``\ アノテーションを宣言することで、各テストメソッドの終了時にトランザクションが
          ロールバックされるようになる。これによって、テストの実行によるDBの内容の変更を防ぐことができる。
    * - | (5)
      - | 試験対象のクラスをインジェクションする。
        | 試験対象である\ ``RouteRepository``\ クラスをインジェクションする。
    * - | (6)
      - | \ ``JdbcTemplate``\ クラスをインジェクションする。
        | \ ``JdbcTemplate``\ とはSpring JDBCサポートのコアクラスである。JDBC APIではデータソースからコネクションの取得、
          PreparedStatementの作成、ResultSetの解析、コネクションの解放などを行う必要があるが、\ ``JdbcTemplate``\ 
          を使うことでこれらの処理の多くが隠蔽され、より簡単にデータアクセスを行うことができる。
          DBUnitを使用しない場合は、\ ``JdbcTemplate``\ を使用してテストデータの投入を行うことを推奨する。

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

次にテスト用データを投入するメソッドを追加する。★@Sqlを使用するのであれば、上で説明する

* ``RouteRepositoryTest.java``

.. code-block:: java

    @Before // (1)
    public void setUp() throws Exception {

    }



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 

.. note:: **JdbcTemplateの使い方(INSERT/UPDATE/DELETE文)**

    JdbcTemplateにて、INSERT/UPDATE/DELETE文を発行する際はupdateメソッドを使用する。
    INSERT/UPDATE/DELETE文はいずれも更新系のSQLなので、1つのメソッドに集約されている。
    メソッド名の「update」は、UPDATE文を意味するわけではないので、注意すること。
    使用法としては、第1引数にSQL文を指定し、第2引数以降にパラメータの値を指定すること。
    SELECT文の使用法については次の参照系のテストメソッドの作成例にて説明を行う。
    
    ※@Sqlをメインで書く場合、JdbcTemplateが出てこないので、noteの位置と内容を変更

|

参照系のテストメソッドの作成例を以下に示す。

* ``RouteRepositoryTest.java``

.. code-block:: java

    package jp.co.ntt.atrs.domain.repository.route;

    @Test
    public void testFindAll() {

        // (1)
        List<Route> routeList = target.findAll();

        // (2)
        assertEquals(routeList.size(), 2);

        // (3)
        assertEquals(routeList.get(0).getRouteNo().intValue(), 1);
        assertEquals(routeList.get(1).getRouteNo().intValue(), 2);
        assertEquals(routeList.get(0).getBasicFare().intValue(), 30600);
        assertEquals(routeList.get(1).getBasicFare().intValue(), 40700);

        Airport DepAirport_0 = routeList.get(0).getDepartureAirport();
        Airport DepAirport_1 = routeList.get(1).getDepartureAirport();
        Airport ArrAirport_0 = routeList.get(0).getArrivalAirport();
        Airport ArrAirport_1 = routeList.get(1).getArrivalAirport();

        assertEquals(DepAirport_0.getCode(), "HND");
        assertEquals(DepAirport_0.getName(), "東京（羽田）");
        assertEquals(DepAirport_1.getCode(), "HND");
        assertEquals(DepAirport_1.getName(), "東京（羽田）");

        assertEquals(ArrAirport_0.getCode(), "ITM");
        assertEquals(ArrAirport_0.getName(), "大阪（伊丹）");
        assertEquals(ArrAirport_1.getCode(), "MBE");
        assertEquals(ArrAirport_1.getName(), "オホーツク紋別");
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | テスト対象メソッドを実行する。
    * - | (2)
      - | 期待した結果件数が返却されることの確認する。
    * - | (3)
      - | 期待した結果が取得できていることを確認する。


更新系のテストメソッドの作成例を以下に示す。

* ``RouteRepositoryTest.java``

.. code-block:: java

    package jp.co.ntt.atrs.domain.repository.member;

    @Test
    public void testUpdate() {

        // (1)
        MemberLogin memberLogin = new MemberLogin();
        String updatePW = "update";
        memberLogin.setPassword(updatePW);
        // omitted

        Member member = new Member();
        String updateMemShipNum = "08";
        member.setMembershipNumber(updateMemShipNum);
        // omitted
        member.setMemberLogin(memberLogin);

        // (2)
        int actualNum = target.updateMemberLogin(member);

        // (3)
        assertEquals(actualNum, 1);

        // (4)
        String cntSql = "SELECT COUNT(*) FROM member_login";
        int resultCnt = jdbctemplate.queryForObject(cntSql, Integer.class);
        assertEquals(resultCnt, 10);

        // (5)
        String sql = "SELECT customer_no, password FROM member_login WHERE customer_no = '08'";
        List<Member> actualList = jdbctemplate.query(sql,
                new MemberRowMapper());
        Member actualMember = actualList.get(0);
        assertEquals(actualMember.getMembershipNumber(), updateMemShipNum);
        assertEquals(actualMember.getMemberLogin().getPassword(), updatePW);
    }

    // (6)
    private static class MemberRowMapper implements RowMapper<Member> {

        @Override
        public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
            Member m = new Member();
            MemberLogin ml = new MemberLogin();

            m.setMembershipNumber(rs.getString("CUSTOMER_NO"));
            ml.setPassword(rs.getString("PASSWORD"));
            m.setMemberLogin(ml);

            return m;
        }
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
      - | テスト対象メソッド実行後のテストデータ件数を取得し、変更がないことを確認する。
    * - | (5)
      - | テスト対象メソッド実行後のテストデータを取得し、変更されていることを確認する。
    * - | (6)
      - | RowMapperを使用することで、DBから取得した\ ``ResultSet``\ を特定のPOJOクラス（\ ``Member``\クラスと
          \ ``MemberLogin``\ クラス）にマッピングすることができる。


spring-testとDBUnitを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

データアクセスにDBUnitを使用する場合のRepositoryの単体テスト実装方法について説明する。

DBUnitとは、データベースに依存するクラスのテストを行うためのJUnit拡張フレームワークである。
以下のような機能を利用することで試験工数を削減できるため、基本的にはDBUnitを用いて実装することを推奨する。

 * 事前のテストデータのセットアップ機能
 * テスト実施後の期待結果データとの比較によるデータベースの状態の検証機能

DBUnitを利用したRepositoryの単体テストにおいて、作成するファイルを以下に示す。

.. figure:: ./images/UnitTest_project_configuration_dbunit.png
   :width: 95%

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - 作成するファイル名
      - 説明
    * - XxxRepositoryTest.java
      - XxxRepository.javaのテストクラス(DBUnitと連携する場合)
    * - test-context-dbunit.xml
      - Repositoryの単体テストを行う際に使用する設定ファイル(DBUnitと連携する場合)
    * - afterdelete_data.xml
      - 削除のテスト実行後の期待結果データファイル
    * - afterinsert_data.xml
      - 登録のテスト実行後の期待結果データファイル
    * - afterupdate_data.xml
      - 更新のテスト実行後の期待結果データファイル
    * - test_data.xml
      - テストで使用する試験前提条件データファイル
    * - route-dataset.sql
      - テストで使用する初期データファイル
    * - schema.sql
      - テスト用のDDLファイル

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
    @ContextConfiguration(locations = { "classpath*:META-INF/spring/test-context-dbunit.xml" }) // (1)
    @Transactional
    public class RouteRepositoryDbUnitTest extends DataSourceBasedDBTestCase { //(2)

        // omitted

        @Inject
        DataSource dataSource;  //(3)

        @Before
        public void setUp() throws Exception {
            super.setUp();
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ :ref:`TestGuideSettingOfDbUnit`\ で作成した設定ファイルを読み込む
    * - | (2)
      - | \ ``org.dbunit.DataSourceBasedDBTestCase``\ を継承する。
    * - | (3)
      - | \ ``javax.sql.DataSource``\ をインジェクションする。

|

次にテスト用データを投入する方法の例を示す。

* ``RouteRepositoryTest.java``

.. code-block:: java






参照系のテストメソッドの作成例を以下に示す。

* ``RouteRepositoryTest.java``

.. code-block:: java

    package jp.co.ntt.atrs.domain.repository.route;

    @Test
    public void testfindAll() throws Exception {

        // テスト対象の実行
        List<Route> routes = target.findAll();

        // DBにアクセスして、現在の登録されている情報を全て取得
        String sql = "SELECT r.route_no, r.basic_fare, a_dep.airport_cd AS dep_airport_cd, 
                + a_dep.airport_name AS dep_airport_name, "
                + "a_arr.airport_cd AS aar_airport_cd, a_arr.airport_name AS aar_airport_name "
                + "FROM route r, airport a_dep, airport a_arr " + "WHERE r.dep_airport_cd = a_dep.airport_cd "
                + "AND r.arr_airport_cd = a_arr.airport_cd";

        List<Route> actualList = jdbctemplate.query(sql, new RouteRowMapper());

        // 期待した結果が返却されてくることの確認
        assertEquals(routes.size(), actualList.size());

        // expectedListとactualListの内容が合っているかの確認 （追加部分）
        // 中身も同じであることを確認したほうがいいと思ったため。
        for (int i = 0; i < actualList.size(); i++) {

            Route route = routes.get(i);
            Route actualRoute = actualList.get(i);

            assertEquals(actualRoute.getRouteNo(), route.getRouteNo());

            assertEquals(actualRoute.getDepartureAirport().getCode(), route.getDepartureAirport().getCode());
            assertEquals(actualRoute.getDepartureAirport().getName(), route.getDepartureAirport().getName());
            assertEquals(actualRoute.getDepartureAirport().getDisplayOrder(),
                    route.getDepartureAirport().getDisplayOrder());

            assertEquals(actualRoute.getArrivalAirport().getCode(), route.getArrivalAirport().getCode());
            assertEquals(actualRoute.getArrivalAirport().getName(), route.getArrivalAirport().getName());
            assertEquals(actualRoute.getArrivalAirport().getDisplayOrder(),
                    route.getArrivalAirport().getDisplayOrder());

            assertEquals(actualRoute.getBasicFare(), route.getBasicFare());
        }

        // 比較用データ （追加部分）
        IDataSet expectedDataSet = new FlatXmlDataSetBuilder()
                .build(new File("src/test/resources/META-INF/data/after_data.xml"));

        // データが変わっていないことの確認 （追加部分）
        Assertion.assertEquals(getDataSet(), expectedDataSet);

    }

    private static class RouteRowMapper implements RowMapper<Route> {

        @Override
        public Route mapRow(ResultSet rss, int rowNum) throws SQLException {
            Route r = new Route();
            Airport arr = new Airport();
            arr.setCode(rs.getString("AAR_AIRPORT_CD"));
            arr.setName(rs.getString("AAR_AIRPORT_NAME"));

            Airport dep = new Airport();
            dep.setCode(rs.getString("DEP_AIRPORT_CD"));
            dep.setName(rs.getString("DEP_AIRPORT_NAME"));

            r.setRouteNo(rs.getInt("ROUTE_NO"));
            r.setBasicFare(rs.getInt("BASIC_FARE"));
            r.setArrivalAirport(arr);
            r.setDepartureAirport(dep);

            return r;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 
    * - | (2)
      - | 
    * - | (3)
      - | 


.. note:: **DbJUnitのExcelバージョンについて**

    DBUnitでは、FlatXML以外にExcel形式（.xlsx）のデータ定義ファイルをテストデータや期待結果データとして用いることが出来る。

    spring-test-dbunitでは、データ定義ファイルの読込機能をDataSetLoaderというインタフェースを実装したクラスに委譲しており、
    Excel形式のデータ定義ファイル読込ロジックを定義したDataSetLoaderを実装し、spring-test-dbunitが利用するように設定すれば
    実現できる。

    以下、実装例を示す。※spring-test-dbunitを使用しない場合は、別途実装方法の調査が必要

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

     ../_images/xlsxdataset.png 

    Excel形式のデータ定義ファイルでは、各シートが各テーブルに対応する。
    シート名にはテーブル名、シートの一行目にはカラム名を設定する。 二行目以降にテーブルに挿入されるデータを記述する。


.. note:: **シーケンスの初期化**

    シーケンスは、トランザクションをロールバックしても進んだ値は戻らないという特徴を持つ。
    そのため、DBUnitでシーケンスから採番したカラムを持つレコードを検証する場合、シーケンスから採番したカラムは
    検証対象外とするか、以下のように明示的にシーケンスの初期化を行うSQLを実行し、テストの実施前に初期化する必要がある。

    * シーケンスの初期化（PostgreSQLの例）

     .. code-block:: java

        @Inject
        private JdbcTemplate jdbcTemplate;

        @Test
        public void testUpdate() throws Exception {

            // ID払い出し用のシーケンスをリセット
            jdbcTemplate.execute("ALTER SEQUENCE record_id_seq RESTART WITH 1");

            // シーケンスに依存した処理の呼び出し
        }

    * テストクラス内の全テストメソッドでシーケンスの初期化が必要な場合の共通化（PostgreSQLの例）

    テストクラス内の全テストメソッドでシーケンスの初期化が必要な場合、 @Beforeアノテーションを付与したメソッド内で
    シーケンスの初期化処理を呼び出すことで、共通化を行うことが可能である。

     .. code-block:: java

        @Inject
        private JdbcTemplate jdbcTemplate;

        @Before
        public void setUp() {
            // ID払い出し用のシーケンスをリセット
            jdbcTemplate.execute("ALTER SEQUENCE SQ_MEMBER_1 RESTART WITH 1");
        }

        @Test
        public void testUpdate1() throws Exception {

            // シーケンスに依存した処理の呼び出し
        }

        @Test
        public void testUpdate2() throws Exception {

            // シーケンスに依存した処理の呼び出し
        }


|

.. _UnitTestOfDomainLayer:

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


.. _UnitTestOfServiceLayer:

Serviceの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60

    * - テストパターン
      - 特徴
      - 使い分けの方針
    * - spring-test
      - 基本??
      - 依存クラスがテスト済みでモック化する必要がない場合
    * - Junit + Mockito
      - 基本??
      - モック化が必要な場合

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

・フォルダ構成の図

テスト済みの\ ``Repository``\ クラスを使用する場合、DBUnitを使用して\ ``Repository``\ クラスをインジェクションして
テスト対象の\ ``ServiceImpl``\ クラスのテスト作成方法を説明する。

作成するファイルを以下に示す。

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

JunitとMocitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

・フォルダ構成の図

\ ``Repository``\ クラスなど\ ``ServiceImpl``\ クラスが依存するクラスをモック化する場合のテスト作成方法を説明する。

作成するファイルを以下に示す。

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

.. _UnitTestOfAplicationLayer:

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

Springは\ ``Controller``\ クラスを試験するためのサポートクラス
(\ ``org.springframework.test.web.servlet.setup.MockMvcBuilders``\ など)を用意している。
これらのクラスを利用することでJUnitから\ ``Controller``\ クラスのメソッドを実行して試験をすることができる。

spring-test + MockMVC + Mockitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

\ ``Controller``\ がインジェクションしている\ ``Service``\ クラスはモック用ライブラリを使用する。
Serviceクラスがテスト済みの場合は、テスト済みのServiceクラスを使用する。

作成するファイルを以下に示す。

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

                // (2)
                mockSession.setAttribute("userId", "0001");

                // (3)
                MockHttpServletRequestBuilder getRequest = MockMvcRequestBuilders.get(
                    "/checkSession").session(mockSession);

                ResultActions results = mockMvc.perform(getRequest); // (4)

                // omitted
            }

         .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
         .. list-table::
             :header-rows: 1
             :widths: 10 90

             * - 項番
               - 説明
             * - | (1)
               - | セッションのモックを生成する。
             * - | (2)
               - | (1)で生成したモックセッションにオブジェクトを格納する。
             * - | (3)
               - | セッションを登録したリクエストのモックを生成する。
                 | \ ``org.springframework.test.web.servlet.request.MockMvcRequestBuilders``\ の\ ``get``\ メソッドで
                   リクエストのモックを生成し、生成したリクエストに\ ``session``\ メソッドでセッションのモックを登録する。
                   例では\ ``/checkSession``\へのGETリクエストにセッションのモックを登録している。
             * - | (4)
               - | \ ``MockMvc``\ にリクエストを渡してコントローラのメソッドを実行する。
                   結果の確認方法は\ ``@AuthenticationPrincipal``\ アノテーションを利用している場合を参照。 

|

Helperの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60

    * - テストパターン
      - 特徴
      - 使い分けの方針
    * - Junit + Mockito
      - 基本??
      - 

Helperの単体テストで、特別に意識すべきことはない。通常のPOJO(Plain Old Java Object)と同様にJUnitによる
単体テストを実施する。

テスト実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

実装方法については、\ :ref:`UnitTestOfServiceLayer`\ を参照されたい。



Validatorの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 40 60

    * - テストパターン
      - 特徴
    * - BeanValidation
      - カスタムバリデーションのテスト
    * - SpringValidation
      - 相関項目チェックのテスト

JUnitを使用した試験（Bean Validation）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Validator(Bean Validation)の単体テストについては、JUnitを使用して試験を実施する。
カスタムバリデーションの試験を行う。HibernateValidatorが用意する入力チェックのアノテーションについては
フレームワーク側で担保しているので、単体テストを行う必要はない。

作成するファイルを以下に示す。

・フォルダ構成の図

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

・フォルダ構成の図

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

