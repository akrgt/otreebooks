# Sessions



oTreeにおいて，sessionとは，複数のプレイヤーが参加している一連のタスク，もしくはゲームのことを指します．以下はsessionの一例です．

> 多数の参加者が実験室に集まり，公共財ゲームを以下のような質問文による実験を実施します．参加者は10.00ユーロを参加費として受け取り，さらにゲームの結果に応じて報酬を受け取ります．

### Subsessions

sessionは複数のsubsessionの集まりです．つまり，subsessionとは，sessionを形作るセクション，もしくはモジュールということです．例えば，公共財ゲームの後にアンケートを実施するとすれば，公共財ゲームはsubsessionの1つ目であり，アンケートはsubsessionの2つ目，ということができます．それぞれのsubsessionは複数の"page"からなります．例えば，公共財ゲームのsubsessionの中に4ページあり，アンケートのsubsessionの中に2ページあるイメージです．





例えば，複数期繰り返すゲームであれば，各期をsubsessionとしてみなすことができます．

### Groups

各subsessionはプレイヤーのグループに分けることができます．例えば，30人プレイヤーのsubsessionがあれば，2人組を15グループに分けることができます．

※グループはsubsessionの間でシャッフルすることができます．

### Object hierarchy

oTreeには以下のような階層性があります．

```text
Session
       Subsession
          Group
               Player
                 Page
```

* sessionとは，subsessionの集合です．
* subsessionには複数のgroupを含んでいます．
* groupには，複数のプレイヤーを含んでいます．
* 各プレイヤーは複数のページで進めていきます．

### Participant

oTreeにおいては，"player"と"participant"は違うものです．"player"と"participant"の関係は，sessionとsubsessionの関係と同じようなものです．

playerは特定のsubsessionにおけるparticipantの呼び方です．playerはparticipantが果たす一時的な役のようなものです．あるparticipantは最初のsubsessionでplayer2に割り振られた後に，次のsubsessionではplayer1に割り振られると言ったようなものです．

### What is "self"?

Pythonにおいて，`self`は現在あなたが使用しているクラスのインスタンスを示しています．もし，今までに`self`がある文脈においてどのような役割を果たしているのかわからなければ，classの名前を見つけるまでスクロールアップしてください．

以下の例では，`self`は`player`オブジェクトを反映しています．

```text
class Player(object):

    def my_method(self):
        return self.my_field
```

次の例では，`self`は`Group`オブジェクトを反映しています．

```text
class Group(object):

    def my_method(self):
            return self.my_field
```

`self`は概念的には，`me`という言葉と同様の意味です．あなたはあなた自身を`me`と呼んでいますが，他の人はあなたのことをあなたの名前で呼んでいます．そして，あなたの友達が`me`という言葉を言った時には，あなたが`me`という言葉を言った時とは違うことを意味しています．

例えば，もしあなたが`player`クラスにいる場合，そのplayerのpayoffは`self.payoff`として示すことができます．（なぜならば，`self`はplayerを意味しているからです）．しかし，もしあなたが`views.py`の中にある`Page`クラスにいる場合，同じことを意味するには，`self.player.payoff`になります．ポインタはpageからplayerに移動します．

### Self: extended examples

以下にはいくつかのコードの例を紹介します．

```text
class Session(...) # this class is defined in oTree-core
    def example(self):

        # current session object
        self

        self.config

        # child objects
        self.get_subsessions()
        self.get_participants()

class Participant(...) # this class is defined in oTree-core
    def example(self):

        # current participant object
        self

        # parent objects
        self.session

        # child objects
        self.get_players()
```

そして，`models.py`は以下の通りです．

```text
class Subsession(BaseSubsession):
    def example(self):

        # current subsession object
        self

        # parent objects
        self.session

        # child objects
        self.get_groups()
        self.get_players()

        # accessing previous Subsession objects
        self.in_previous_rounds()
        self.in_all_rounds()

class Group(BaseGroup):
    def example(self):

        # current group object
        self

        # parent objects
        self.session
        self.subsession

        # child objects
        self.get_players()

class Player(BasePlayer):

    def example(self):

        # current player object
        self

        # method you defined on the current object
        self.my_custom_method()

        # parent objects
        self.session
        self.subsession
        self.group
        self.participant

        self.session.config

        # accessing previous player objects
        self.in_previous_rounds()

        # equivalent to self.in_previous_rounds() + [self]
        self.in_all_rounds()
```

最後に，`views.py`は以下の通りです．

```text
class MyPage(Page):
    def example(self):

       # current page object
       self

       # parent objects
        self.session
        self.subsession
       self.group
        self.player
         self.participant
         self.session.config
```

[https://otree.readthedocs.io/en/latest/conceptual\_overview.html](https://otree.readthedocs.io/en/latest/conceptual_overview.html)

