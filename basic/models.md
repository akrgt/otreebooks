# Models

### Models

`models.py`はアプリケーション内で用いられるデータを定義しているものです．以下の3つからなります．

* Subsession
* Group
* Player

1人のPlayerはグループの構成要員であり，subsessionの一部です．詳細は"Conceptual overview"でご確認ください．

### Model fields

`models.py`の主な定義は，データベース上における列を定義するものです．以下のような表を作ってみましょう．

| name | age | is\_student |
| :--- | ---: | :---: |
| John | 30 | False |
| Alice | 22 | True |
| Bob | 35 | False |
| ... |  |  |

このようなテーブル構造は，以下のように定義することができます．

```text
class Player(BasePlayer):
    ...
    name = models.CharField()
    age = models.PositiveIntegerField()
    is_student = models.BooleanField()
```

`otree resetdb`を実行した時，`models.py`を分析して，その通りにデータベーステーブルを作成します．（だから，`models.py`に追加したり，削除したり，項目に変更を加えた際に`resetdb`を実行する必要があります．）

利用可能な`fields`のリストはDjangoドキュメンテーションの[ココ](https://docs.djangoproject.com/ja/1.9/ref/models/fields/)にあります．最もよく使われるのは`CharField`/`TextField`（文字列），`FloatField`（実数），`BooleanField`（真偽値），`IntegerField`と`PositiveIntegerField`です．`FloatField`と`DecimalField`の違いを十分に理解しない限り，そして使う必要がない限りは使わないようにしてください．

さらに，oTreeは`CurrencyField`があります．詳細は[お金と利得](http://otree.readthedocs.io/en/latest/currency.html#currency)を御覧ください．

#### Settng a field's initial/default value

あなたが定義したどんなフィールドも，初期値は`None`であり，何も定義されていません．もし，何らかの初期値を定義したいのであれば，`initial =`を使ってください．

```text
class Player(BasePlayer):
    some_number = models.IntegerField(initial=0)
```

#### min, max. choices

`min`（最小値），`max`（最大値）や`choices`（選択肢）を設定したいのであれば，[コチラ](http://otree.readthedocs.io/en/latest/forms.html#form-validation)をごらんください．

### Constants

`Constants`クラスは，プレイヤーに共通するアプリケーションのパラメータと定数を設定するのに適したものです．

以下がアプリケーションに必要となる定数です．

* `name_in_url`：この名前は実験参加者のURLにおいて，アプリケーションを特定するために用いられます．例えば，`public_goods`を設定したのであれば，実験参加者のURLは以下のようになります．`http://otree-demo.herokuapp.com/p/zuzepona/public_goods/Introduction/1/`
* `players_per_group`：\(Groupsの欄で述べます．\)
* `num_rounds`：\(Roundsの欄で述べます．\)

### Subsession

以下には，`Subsession`オブジェクトの属性とメソッドのリストを示します．

#### session

`session`の中には`subsession`があります．詳細は"Self"に関する説明を確認してください．

#### round\_number

現在のラウンド数を表示します．ただし，これは複数ラウンド実施するゲーム実験のみに関係します．\(Constants.num\_roundsで定義されます．）詳細は"Rounds"に関する説明を確認してください．

#### before\_session\_starts

`before_session_starts`は[`creating_session`](http://otree.readthedocs.io/en/latest/models.html#creating-session)からotree-core1.3.2\(June 2017\)より変更したものです．しかしながら，`before_session_starts`も下位互換性を保つために，現在でも利用可能です．

#### creating\_session

> このメソッドは`before_session_starts`と呼ばれていたものです．詳細はそちらをご確認ください．

このメソッドは管理者が"create session"をクリックした際に実行されます．

`creating_session`は`player`，`Groups`，`subsessions`などのフィールドを初期値に戻すものです．例は以下のとおりです．

```text
class Subsession(BaseSubsession):

def creating_session(self):
    for p in self.get_players():
        p.some_field = some_value
```

詳細ない情報は[treatments](http://otree.readthedocs.io/en/latest/treatments.html#treatments)と[group shuffling](http://otree.readthedocs.io/en/latest/multiplayer/groups.html#shuffling)にあります．

もし，アプリケーションが1ラウンドだけであれば，`creating_session`は一度だけ実行されます．もし，アプリケーションがNラウンドあれば，N回実行されます．すなわち，各subsessionに1つずつ必要となるのです．

> この方法は各ラウンドの始まりには実行されます．そのために，ページの進行を止めるためには[`after_all_players_arrive()`](http://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#after-all-players-arrive)を使ってください．

#### group\_randomly\(\)

[Group matching](http://otree.readthedocs.io/en/latest/multiplayer/groups.html#shuffling)を確認してください．

#### group\_like\_round\(\)

[Group matching](http://otree.readthedocs.io/en/latest/multiplayer/groups.html#shuffling)を確認してください．

#### get\_group\_matrix\(\)

[Group matching](http://otree.readthedocs.io/en/latest/multiplayer/groups.html#shuffling)を確認してください．

#### get\_groups\(\)

subsessionにおける全てのグループのリストを返します．

#### get\_players\(\)

subsessionにおける全てのプレイヤーのリストを返します．

#### in\_previous\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_all\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_round\(round\_number\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_rounds\(self, first, last\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

### Group

以下には，`Group`オブジェクトの属性とメソッドのリストを示します．

#### sessions/subsession

`session`の中には`subsession`があります．詳細は"Self"に関する説明を確認してください．

#### get\_players\(\)

[Groups](http://otree.readthedocs.io/en/latest/multiplayer/groups.html)を確認してください．

#### get\_player\_by\_role\(role\)

[Groups](http://otree.readthedocs.io/en/latest/multiplayer/groups.html)を確認してください．

#### get\_player\_by\_id\(id\_in\_group\)

[Groups](http://otree.readthedocs.io/en/latest/multiplayer/groups.html)を確認してください．

#### set\_players\(players\_list\)

[Group matching](http://otree.readthedocs.io/en/latest/multiplayer/groups.html#shuffling)を確認してください．

#### in\_previous\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_all\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_round\(round\_number\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_rounds\(self, first, last\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

### Player

以下には，`Player`オブジェクトの属性とメソッドのリストを示します．

#### id\_in\_group

整数であり，1から始まります．複数人ゲームの場合，この数値はプレイヤー1に割り振られているのか，プレイヤー2に割り振られているのかなどを示します．

#### payoff

プレイヤーのそのラウンドにおける利得を示します．詳細は[payoffs](http://otree.readthedocs.io/en/latest/currency.html#payoff)を確認してください．

#### session/subsession/group/participant

#### get\_others\_in\_group\(\)

[Groups](http://otree.readthedocs.io/en/latest/multiplayer/groups.html)を確認してください．

#### get\_others\_in\_subsession\(\)

[Groups](http://otree.readthedocs.io/en/latest/multiplayer/groups.html)を確認してください．

#### role\(\)

#### in\_previous\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_all\_rounds\(\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_round\(round\_number\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

#### in\_rounds\(self, first, last\)

詳細は[Passing data between rounds or apps](http://otree.readthedocs.io/en/latest/rounds.html#in-rounds)を確認してください．

### Session

#### num\_participants

そのセッションの参加者の人数を示しています．

#### config

[Configure sessions](http://otree.readthedocs.io/en/latest/admin.html#edit-config)と[Choosing which treatment to play](http://otree.readthedocs.io/en/latest/treatments.html#session-config-treatments)を確認してください．

#### vars

[session.vars](http://otree.readthedocs.io/en/latest/rounds.html#session-vars)を確認してください．

### Participant

#### vars

[participants.vars](http://otree.readthedocs.io/en/latest/rounds.html#vars)を確認してください．

#### label

[participants.labels](http://otree.readthedocs.io/en/latest/admin.html#participant-label)を確認してください．

#### id\_in\_session

セッションにおける実験参加者のIDを示しています．このIDはプレイヤーの`id_in_subsession`と同じものです．

#### payoff

[payoffs](http://otree.readthedocs.io/en/latest/currency.html#payoff)を確認してください．

#### payoff\_plus\_participation\_fee

[payoffs](http://otree.readthedocs.io/en/latest/currency.html#payoff)を確認してください．

#### oTreeコードの実行方法

メソッド内にないコードは基本的には全てglobalであり，サーバ起動時に一度だけ実行されます．

一部の人々は新しいセッションが実行される度にコードが実行されると勘違いしているようです．例えば，コイントスで表が出るようなランダム確率を発生させたい人は以下のようなコードをmodels.pyに書くかもしれません．

```text
class Constants(BaseConstants):
    heads_probability = random.random() # wrong
```

サーバが起動した時にmodels.pyを起動します．そして`random.random()`を一度だけ実行します．そしてランダムな数字，例えば“0.257291”を評価します．

```text
class Constants(BaseConstants):
    heads_probability = 0.257291
```

なぜならば，`Constants`はglobal変数であり，“0.257291”が全てのセッションにおいて全プレイヤーに共有されます．

同じ理由で，以下のコードも起動しません．

```text
class Player(BasePlayer):

    heads_probability = models.FloatField(
        # wrong
        initial=random.random()
    )
```

この解決策は，[creating\_session](http://otree.readthedocs.io/en/latest/models.html#creating-session)のように，メソッドの内部でランダム変数を発生させることです．

