★：要検討

単体テストの実装
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

Overview ★節追加
--------------------------------------------------------------------------------

本節の実装例として記載しているサンプル及び、OSSライブラリ構成は一例である。
実際に採用される際には、業務要件に従って実装いただきたい。


単体テスト対象のクラスと試験方法
--------------------------------------------------------------------------------

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
      - spring-testとDbUnit
      - DB操作にDbUnitを使用する場合
    * - Service
      - spring-test
      - モックを使用しない場合
    * - 
      - JunitとMocito
      - モックを使用する場合
    * - Controller
      - spring-test+MockMVC+Mockito
      - モックを使用する場合
    * - Helper
      - spring-test
      - DB操作にSpring JDBCを使用する場合
    * - 
      - spring-testとDbUnit
      - DB操作にDbUnitを使用する場合
    * - Validation
      - ???
      - Bean Validationの場合
    * - 
      - ???
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
    * - Spring
      - org.springframework
      - spring-test
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-test
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.mockito
      - mockito-core
      - 4.3.5.RELEASE
      - \*
      -
    * - Spring
      - org.springframework
      - spring-test
      - 4.3.5.RELEASE
      - \*
      -

|

ライブラリの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

単体試験用のライブラリを\ ``pom.xml``\ の\ ``dependency``\ に追加を行う。以下に追加するライブラリを示す。

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
      <groupId>org.DbUnit</groupId>
      <artifactId>DbUnit</artifactId>
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

インフラストラクチャ層の単体テスト
--------------------------------------------------------------------------------

インフラストラクチャ層のテスト全体観点 ★節不要では。
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ここでは、インフラストラクチャ層の単体テストについて説明する。
| インフラストラクチャ層の詳細については、開発ガイドライン\ :ref:`LayerOfInfrastructure`\を参照されたい。

・インフラストラクチャ層のコンポーネントテストの観点
　2.4.1.3　インフラストラクチャ層の三層レイヤーの図を用いて、
　DBとのアクセス部分がコンポーネントテストの対象となることを説明。
　Mybatis、RepositoryImplは作成不要となるため、Repositoryインタフェースとxmlファイルから自動生成された
　RepositoryImplが試験対象となることを説明。

・本章で取り扱うテスト対象のコンポーネントの種類
　RepositoryImpl（Interfaceではないことを強調）"


Repositoryの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストパターンの特徴★不要？ 前述している。
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. tabularcolumns:: |p{0.20\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 20 20 60

    * - テストパターン
      - 特徴
      - 使い分けの方針
    * - spring-test
      - 基本??
      - DbUnitが使用できない場合（Spring JDBCを使用する場合）
    * - spring-test+DbUnit
      - 基本??
      - DbUnitを使用できる場合


Macchinetta Server Framework 適用システムで、MyBatis3を使用して\ ``Repository``\ を実装している場合、
\ ``RepositoryImpl``\ を実装する必要はない。
サンプルでは、\ ``Repository``\ インタフェースに対してテストを作成しているが、
MyBatis3により\ ``Repository``\ インタフェースとマッパーファイルから自動生成された\ ``RepositoryImpl``\ が
テスト対象となることに注意すること。
詳細は、\ :ref:`repository-mybatis3-label`\ を参照されたい。


spring-testを使用した試験 ★節名を変更?(Spring JDBCのほうがよい？)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Repositoryの単体テストは、JUnitを使用して実施する。
| プロジェクト要件などでDbUnitが使用できない場合、\ ``org.springframework.jdbc.core.JdbcTemplate``\ を用いて
  データアクセスを行う。
| また、Repositoryの単体テストを行う際は単体テスト用の設定ファイルを用意すること。
| 
| 作成するファイル例を以下に示す。

.. _TestGuideSettingOfSpringTest:

spring-testを使用するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Repositoryの単体テストのための設定ファイルとして  \ ``test-context.xml``\ を作成する

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
      - | \ ``jp.co.ntt.atrs.domain.repository``\ パッケージ配下をcomponent-scan対象にする。
        | これにより、\ ``jp.co.ntt.atrs.domain.repository``\ パッケージ配下のクラスに@Repositoryアノテーションを
          付けることで、DI対象にできる。


Repositoryテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| Repositoryの単体テストクラスの作成方法を説明する。

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
    * - | (1)
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


spring-testとDbUnitを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

データアクセスにDbUnitを使用する場合のRepositoryの単体テスト実装方法について説明する。

DbUnitとは、データベースに依存するクラスのテストを行うためのJUnit拡張フレームワークである。
以下のような機能を利用することで試験工数を削減できるため、基本的にはDbUnitを用いて実装することを推奨する。

 * 事前のテストデータのセットアップ機能
 * テスト実施後の期待結果データとの比較によるデータベースの状態の検証機能

DbUnitを利用したRepositoryの単体テストにおいて、作成するファイルを以下に示す。

.. figure:: images/ComponentTest_project_configuration_dbunit.png
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

.. _TestGuideSettingOfDbUnit:

DbUnitを使用するための設定
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| RepositoryのDBUnitを利用した単体テストのための設定ファイルとして \ ``test-context-dbunit.xml``\ を作成する。
| \ :ref:`TestGuideSettingOfSpringTest`\ で作成したファイルに
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
           DbUnitをSpringのトランザクション管理下にすることができる。


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


.. note::

    DbJUnitのExcelバージョンについて








ドメイン層の単体テスト
--------------------------------------------------------------------------------

ドメイン層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、ドメイン層の単体テストについて説明する。
ドメイン層の詳細については、開発ガイドライン\ :ref:`LayerOfDomain`\ を参照されたい。


・ドメイン層のコンポーネントテストの観点
　2.4.1.2. ドメイン層の三層レイヤーの図を用いて、業務ロジックや、CRUD操作についての部分が
　コンポーネントテストの対象となることを説明。

・本章で取り扱うテスト対象のコンポーネントの種類
　ServiceImpl"



Serviceの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストパターンの特徴
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"・それぞれのコンポーネントごとの試験パターン
　表形式でパターンの特徴の記述
　テストパターン　｜　特徴　｜　使い分けの方針
　spring-test　|　基本的なやつ　｜　zzzzzzz
　Junit+Mockito　｜　yyyyy　｜xxxx

参考ガイドライン
　4.1.1 概要の文章

・一部モックを使うのであれば、合わせて読むことを記載"

spring-testを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''


"・フォルダ構成の図
参考ガイドライン
　4.1.1. 概要　をベースにする。

・作成するファイル名、その説明
参考ガイドライン
　4.1.1. 概要　をベースにする。

・RepositoryImpl実コードを使用した試験"


Serviceテストの実装(DBUnitと連携する場合)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・Serviceテストクラスの作成
　参考ガイドライン
　4.1.4. Serviceのテストクラス作成
　をベースにする。

・参考ガイドラインでは、ソースと説明がそれぞれの項目で記載されているが、
ひとつのソースで説明を行うようにする。"



JunitとMocitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・フォルダ構成の図
参考ガイドライン
　4.1.1. 概要　をベースにする。

・作成するファイル名、その説明
参考ガイドライン
　4.1.1. 概要　をベースにする。
"

Serviceテストの実装(DBUnitと連携する場合)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・Serviceテストクラスの作成
　参考ガイドライン
　4.1.3. モッククラスの作成方法（Mockito）
　4.1.4. Serviceのテストクラス作成
　をベースにする。

・参考ガイドラインでは、ソースと説明がそれぞれの項目で記載されているが、
ひとつのソースで説明を行うようにする。"



アプリケーション層の単体テスト
--------------------------------------------------------------------------------

アプリケーション層のテスト全体観点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでは、アプリケーション層の単体テストについて説明する。
アプリケーション層の詳細については、開発ガイドライン\ :ref:`LayerOfApplication`\ を参照されたい。

・アプリケーション層のコンポーネントテストの観点
　2.4.1.1　アプリケーション層の三層レイヤーの図を用いて、データの入出力、入力データの妥当性チェックがコンポーネントテストの対象となることを説明。

・本章で取り扱うテスト対象のコンポーネントの種類
　Controller，Helper，Form(Validation)
※Viewは対象外の旨記載"


Controllerの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストパターンの特徴
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"・それぞれのコンポーネントごとの試験パターン
　表形式で特徴の記述
　テストパターン　｜　特徴　｜　使い分けの方針
　spring-test、MockMVC、Mockito（standalone）
　spring-test、MockMVC（webappcontextsetup）

参考ガイドライン
　5.5.1 概要の文章
"

spring-test+MockMVC+Mockitoを使用した試験
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・フォルダ構成の図
参考ガイドライン
　5.5.1　概要　をベースにする。

・作成するファイル名、その説明
参考ガイドライン
　5.5.1　概要　をベースにする。
"

Controllerテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・Controllerテストクラスの作成
　参考ガイドライン
　5.5.3. Controllerのテストクラス作成
　5.5.4. Controllerのテストメソッド作成
　をベースにする。

・参考ガイドラインでは、ソースと説明がそれぞれの項目で記載されているが、
ひとつのソースで説明を行うようにする。"


Helperの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストパターンの特徴
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

"・それぞれのコンポーネントごとの試験パターン
　表形式でパターンの特徴の記述
　テストパターン　｜　特徴　｜　使い分けの方針
　Junit　|　基本的なやつ　｜　mockito等のスタックが使用できない場合
　Junit+Mockito　｜　yyyyy　｜xxxx

参考ガイドライン
　5.2.1. 概要　をベースにする。

テスト実装例
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

・他のテストパターンと同じように実装することができる旨記載して飛ばす"


Validatorの単体テスト
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

テストパターンの特徴
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


JUnitを使用した試験（Bean Validation）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・フォルダ構成の図
参考ガイドライン
　5.3.1　概要　をベースにする。
・作成するファイル名、その説明
参考ガイドライン
　5.3.1　概要　をベースにする。"


Validatorテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・Validatorテストクラスの作成
　参考ガイドライン
　5.3.2. Validator(Bean Validation)のテストクラス作成
　5.3.3. Validator(Bean Validation)のテストメソッド作成
　をベースにする。

・参考ガイドラインでは、ソースと説明がそれぞれの項目で記載されているが、
ひとつのソースで説明を行うようにする。"

JUnitを使用した試験（Spring Validation）
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

概要
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・フォルダ構成の図
参考ガイドライン
　5.4.1　概要　をベースにする。
・作成するファイル名、その説明
参考ガイドライン
　5.4.1　概要　をベースにする。"


Validatorテストの実装
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

"・Controllerテストクラスの作成
　参考ガイドライン
　5.4.2. Validator(Spring Validation)のテストクラス作成
　5.4.3. Validator(Spring Validation)のテストメソッド作成
　をベースにする。

・参考ガイドラインでは、ソースと説明がそれぞれの項目で記載されているが、
ひとつのソースで説明を行うようにする。"

