テーブルオブジェクト
######################

.. php:namespace:: Cake\ORM

.. php:class:: Table
    :noindex:

テーブルオブジェクトは特定のテーブルに保存されたエンティティーのコレクションへのアクセスを提供します。
それぞれのテーブルは、テーブルと連動するために使われる、関連付けられたテーブルクラスを持つべきです。
もし、テーブルのふるまいをカスタマイズする必要がないなら、CakePHPはテーブルクラスのインスタンスを生成するでしょう。

テーブルオブジェクトとORMを使う前に、 :ref:`データベース接続 <database-configuration>` を定義しているか確かめましょう。

基本的な使い方
================
始めるには、テーブルクラスを作ります。
テーブルクラスは **src/Model/Table** に作ります。テーブルはリレーショナルデータベースに特化したモデルコレクションであり、CakePHPのORMにおいてデータベースへの主なアクセス手段です。
もっとも基本的なテーブルクラスは以下のようになるでしょう。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
    }

定義したクラスにどのテーブルを使うのか、ORMに伝えなかったことに注意してください。
規約により、テーブルオブジェクトはクラス名を小文字とアンダースコアに置き換えた名前に一致するテーブルを使うでしょう。
上記の例では ``articles`` テーブルが使われるでしょう。
テーブルクラスが ``BlogPosts`` と名付けられているなら、テーブルは ``blog_posts`` という名前になるでしょう。
``table()`` メソッドを使うことでテーブルを指定できます。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {

        public function initialize(array $config)
        {
            $this->table('my_table');
        }

    }

テーブル名を指定するとき、名前の変換規約は適用されません。
ORMの規約はまた、それぞれのテーブルが主キーを ``id`` という名前で持っていると想定します。
もし変更する必要があるならば ``primaryKey()`` メソッドで変えられます。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public function initialize(array $config)
        {
            $this->primaryKey('my_id');
        }
    }


テーブルが使うエンティティーをカスタマイズする
----------------------------------------------

デフォルトでは、テーブルオブジェクトは命名規約に基づいてエンティティークラスを使います。
たとえば、テーブルクラスが ``ArticlesTable`` という名前だったらエンティティーは ``Article`` でしょう。
``PurchaseOrdersTable`` という名前だったらエンティティーは ``PurchaseOrder`` になります。
命名規約に従わない場合は、 ``entityClass()`` メソッドで設定を変えられます。::

    class PurchaseOrdersTable extends Table
    {
        public function initialize(array $config)
        {
            $this->entityClass('App\Model\PO');
        }
    }

上記の例えでは、テーブルオブジェクトはコンストラクターの最後で呼ばれる ``initialize()`` メソッドを持ちます。
コンストラクターをオーバーライドする代わりに、初期化するためにこのメソッドを使うことが推奨されます。

テーブルクラスのインスタンスを取得する
--------------------------------------

テーブルにクエリを送る前に、テーブルインスタンスを取得する必要があります。
``TableRegistry`` クラスでできます。::

    // コントローラーかテーブルメソッドで
    use Cake\ORM\TableRegistry;

    $articles = TableRegistry::get('Articles');

TableRegistry クラスはテーブルを作るための様々な依存関係を提供します。
そして、作られたすべてのテーブルインスタンスの登録を維持し、リレーションの構築とORMの設定を簡単にしてくれます。
詳細は :ref:`table-registry-usage` を参照してください。

.. _table-callbacks:

コールバックのライフサイクル　
============================

上で見てきたように、テーブルオブジェクトはいくつかのイベントを引き起こします。
イベントは、ORMをフックしたり、サブクラスやメソッドのオーバーライドを使わずにロジックを加えたいときに便利です。
イベントリスナーはテーブルクラスかビヘイビアクラスで定義できます。
また、リスナーをバインドするためにテーブルのイベントマネージャーを使うことができます。

コールバックメソッドを使うとき、 ``initialize()`` メソッドに付けられているふるまいは、テーブルのコールバックメソッドが実行される **前に** リスナーを開始させるというものです。
これはコントローラーおよびコンポーネントと同じ順序です。

イベントリスナーをテーブルクラスやビヘイビアに追加するには、下記で示すように単純にメソッドのシグネチャを実装します。
イベントサブシステムの使い方における詳細については :doc:`/core-libraries/events` を参照してください。

beforeMarshal
-------------

.. php:method:: beforeMarshal(Event $event, ArrayObject $data, ArrayObject $options)

``Model.beforeMarshal`` イベントは、リクエストデータがエンティティーに変換される前に呼ばれます。
詳細は :ref:`before-marshal` を参照してください。

beforeFind
----------

.. php:method:: beforeFind(Event $event, Query $query, ArrayObject $options, boolean $primary)

``Model.beforeFind`` イベントは find する前に呼ばれます。イベントを止めて戻り値を返すことで
findを完全にバイパスできます。
$query インスタンスによってなされた全ての変更はfindに影響します。 ``$primary`` はルートクエリ
である場合やそうでない場合もあります、また関連付けられたクエリである場合もあります。
全てのアソシエーションは ``Model.beforeFind`` が呼ばれた時にクエリに反映されます。
アソシエーションがJOINを使うためにダミークエリが用意されています。
イベントリスナーで追加のフィールド、検索条件、JOINや結果のフォーマットを設定出来ます。
これらのオプションや機能はルートクエリにコピーされます。

このコールバックを、findをACLなどで設定されたユーザーロールによって制限するためや、
現在のロードした情報にしたがってキャッシュをするために使います。

前のCakeでは　 ``afterFind`` コールバックがありましたが、 :ref:`map-reduce`
機能とエンティティーコンストラクターに置き換えられました。

buildValidator
---------------

.. php:method:: buildValidator(Event $event, Validator $validator, $name)

``Model.buildValidator`` イベントは ``$name`` バリデーターが作られた時に呼ばれます。
ビヘイビアはこのメソッドを呼ぶために使えます。

buildRules
----------

.. php:method:: buildRules(Event $event, RulesChecker $rules)

``Model.buildRules`` イベントはルールインスタンスが作られた後 ``beforeRules()`` メソッドが呼ばれる前
に呼ばれます。

ビフォアルール
--------------

.. php:method:: beforeRules(Event $event, Entity $entity, ArrayObject $options, $operation)

``Model.beforeRules`` イベントはエンティティにルールが適用される前に呼ばれます。
イベントが止まると、Cakeによるチェックが入る前の戻り値を得られます。

afterRules
--------------

.. php:method:: afterRules(Event $event, Entity $entity, bool $result, $operation)

``Model.afterRules`` イベントはルールがエンティティーに適用された後に呼ばれます。
イベントが止まると、設定したルールによってチェックした後の戻り値を得られます。

beforeSave
----------

.. php:method:: beforeSave(Event $event, Entity $entity, ArrayObject $options)

``Model.beforeSave`` イベントはエンティティーが保存する前に呼ばれます。
イベントを止めることによって、保存を停止できます。イベントが停止すると、このイベントの結果が
返されます。

afterSave
---------

.. php:method:: afterSave(Event $event, Entity $entity, ArrayObject $options)

``Model.afterSave`` は保存した後に呼ばれます。

afterSaveCommit
---------------

.. php:method:: afterSaveCommit(Event $event, Entity $entity, ArrayObject $options)

``Model.afterSaveCommit`` はトランザクション処理でラップされた保存がコミットされた後に、
これはまた、明示的でないコミットで原子性でない保存のために呼ばれます。
このイベントは ``save()`` が直接読んでいるプライマリテーブルのためだけに呼ばれます。
このイベントは、トランザクション処理が保存を開始する前に呼ばれない。

beforeDelete
------------

.. php:method:: beforeDelete(Event $event, Entity $entity, ArrayObject $options)

``Model.beforeDelete`` は削除する前に呼ばれる。
イベントを停止することによって、削除を中止できる。

afterDelete
-----------

.. php:method:: afterDelete(Event $event, Entity $entity, ArrayObject $options)

``Model.afterDelete`` はエンティティーが削除された後に呼ばれる。

afterDeleteCommit
-----------------

.. php:method:: afterDeleteCommit(Event $event, Entity $entity, ArrayObject $options)

``Model.afterDeleteCommit`` イベントはトランザクション処理でラップされた削除処理が
コミットされた後に呼ばれます。これはまた、明示的でないコミットで原子性でない保存のために呼ばれます。
このイベントは ``delete()`` が直接呼んでいるプライマリテーブルのためだけに呼ばれます。
このイベントは、トランザクション処理が削除を開始する前に呼ばれない。

Behaviors
=========

.. php:method:: addBehavior($name, array $options = [])

.. start-behaviors

ビヘイビアは水平に再利用可能なテーブルに関連付けられたロジックの部品を作るための
簡単な方法を提供します。なぜビヘイビアは通常のクラスやトレイトではないかと考えて
いませんか？第一の理由はイベントリスナーだということです。トレイトが再利用可能な
ロジックの部品を許可しているので、トレイとであることを許可することはイベントの
作成を複雑にします。

ビヘイビアをテーブルに追加するために ``addBehavior()`` メソッドが使えます。
一般的に、これを ``initialize()`` でやるのがもっともよいです。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public function initialize(array $config)
        {
            $this->addBehavior('Timestamp');
        }
    }

アソシエーションには :term:`plugin syntax` と追加の設定オプションが使えます。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public function initialize(array $config)
        {
            $this->addBehavior('Timestamp', [
                'events' => [
                    'Model.beforeSave' => [
                        'created_at' => 'new',
                        'modified_at' => 'always'
                    ]
                ]
            ]);
        }
    }

.. end-behaviors

ビヘイビアの詳細は :doc:`/orm/behaviors`　こちら。ビヘイビアに関連することも含みます。


.. _configuring-table-connections:

接続設定
=======================

デフォルトでは、全てのテーブルインスタンスは ``default`` データベス接続を使用します。
もし、複数のデータベース設定を使い分けたいなら、 ``defaultConnectionName()`` で設定できます。::

    namespace App\Model\Table;

    use Cake\ORM\Table;

    class ArticlesTable extends Table
    {
        public static function defaultConnectionName() {
            return 'slavedb';
        }
    }

.. note::

    The ``defaultConnectionName()`` method **must** be static.

.. _table-registry-usage:

Using the TableRegistry
=======================

.. php:class:: TableRegistry


これまで見てきたように、TableRegistry クラスは　factory/registry を
アプリのテーブルインスタンスに接続するために使うことを簡単にします。
これには他にも使える機能があります。

テーブルオブジェクトの設定
-----------------------------

.. php:staticmethod:: get($alias, $config)

テーブルをレジストリからロードする時に、依存関係をカスタマイズするか、
``$options`` 配列が用意するモックオブジェクトを使います。::

    $articles = TableRegistry::get('Articles', [
        'className' => 'App\Custom\ArticlesTable',
        'table' => 'my_articles',
        'connection' => $connectionObject,
        'schema' => $schemaObject,
        'entityClass' => 'Custom\EntityClass',
        'eventManager' => $eventManager,
        'behaviors' => $behaviorRegistry
    ]);

接続とスキーマー設定に注意して下さい。それらは文字列変数ではなくオブジェクトです。
この接続は ``Cake\Database\Connection`` のオブジェクトと
``Cake\Database\Schema\Collection`` のスキーマを操作します。

.. note::

    テーブルは追加の設定を ``initialize()`` で行えます。それらは
    registry　の設定を上書きします。

また、事前に registry を ``config()`` を使って設定できます。
設定データは *per alias*　に保存され、オブジェクトの
``initialize()`` メソッドで上書きできます。::

    TableRegistry::config('Users', ['table' => 'my_users']);

.. note::

    設定はエイリアスに接続しているかする前の　 **最初** だけ変更できます。
    レジストリが一般化された後に設定しても効果がありません。

レジストリの初期化（追加設定の消去）
-------------------------------------

.. php:staticmethod:: clear()

テストケースで、レジストリを綺麗にする必要があります。
モックオブジェクトを使う時やテーブルの依存関係を設定する時によく使う機会があります。::

    TableRegistry::clear();
