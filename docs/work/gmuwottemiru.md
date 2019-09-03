# 信頼ゲームを作ってみる

さて，それでは信頼ゲームを作ってみましょう．そしてoTreeについてもっと詳しく理解していきましょう．

信頼ゲームとは何か，というのは[こちら](https://www.blwisdom.com/marketing/series/behavioral/item/9379-06.html)を御覧ください．私のお師匠さまが書いてます．

でも，概要はこんな感じ．

> はじめに，プレイヤー1が10ポイント受け取ります．そしてプレイヤー2は何も受け取りません．プレイヤー1はプレイヤー2に自身の持っているポイントの一部，もしくは全部を渡すことができます．ただし，プレイヤー2が受け取る前に，その渡したポイントは3倍になります．プレイヤー2が3倍になったポイントを受け取ったら，プレイヤー2はそのうちのいくらをプレイヤー1に渡すか決定することになります．

ここでは以下の工程を経て，信頼ゲームのアプリケーションを作成していきます．

* アップグレード
* 初期アプリケーションの作成
* models.pyの定義
* templates.htmlとviews.pyの定義
* settings.pyの中のSESSION\_CONFIGSの追加
* データベースのリセットと再起動

  の流れで行きます．

### アップグレード

最新バージョンのoTreeを使うために，コマンドラインを開いて，以下のコードを実行しましょう

```text
pip3 install -U otree-core
#再度インストールします．

otree resetdb
#インストールをした際にはデータベースをリセットしましょう．
```

### 初期アプリケーションの作成

cdコマンドを使って，"requirements\_base.txt"のある，あなたが作ったoTreeプロジェクトフォルダに移動してください．

このディレクトリの中に，公共財ゲーム用のディレクトリを作ってあげます．

```text
otree startapp my_trust
```

これによってoTreeプロジェクトフォルダの中に"my\_trust"というフォルダができたことを確認してください．

### model.pyの定義

最初に，アプリケーションにおけるモデルの定数を定義しましょう．初期保有額は10ポイントであり，支払額は3倍になります．

```text
class Constants(BaseConstants):
    name_in_url = 'my_trust'
    players_per_group = 2
    num_rounds = 1

    endowment = c(10)
    multiplication_factor = 3
```

続いて，プレイヤーフィールドとグループフィールドを追加しましょう．ここには2つの重要なデータを用意します．"sent"としてプレイヤー1からプレイヤー2への支払額を定義し，"sent back"はプレイヤー2からプレイヤー1への支払額です．

直感的には，プレイヤーフィールドは以下のように定義するかと思われるかもしれません．

```text
# Don't copy paste this...see below
class Player(BasePlayer):

    sent_amount = models.CurrencyField()
    sent_back_amount = models.CurrencyField()
```

しかし，このモデルには問題があります．"sent\_amount"はプレイヤー1だけに適用されなければならず，"sent\_back\_amount"はプレイヤー2だけに適用されなければなりません．そして，プレイヤー1は"sent\_back\_amount"を呼び出せなければなりません．どのようにすれば，データモデルはより適切になるでしょうか？

これらのフィールドは"Group"レベルで定義する必要があります．

```text
class Group(BaseGroup):

    sent_amount = models.CurrencyField()
    sent_back_amount = models.CurrencyField()
```

そして，プレイヤー1の支払額の選択は，自由に数字を入力できる型式ではなく，ドロップダウンメニューで選べるようにしましょう．この機能を実現するために，"currency\_range"関数とともに"choice"という引数を用います．

```text
sent_amount = models.CurrencyField(
    choices=currency_range(0, Constants.endowment, c(1)),
)
```

同様に，プレイヤー2の支払額の選択をドロップダウンメニューから選べるようにします．しかし，"choices"を特定することができません．なぜならば，プレイヤー1の支払額に応じてプレイヤー2の選択肢が変わるからです．後ほど，これにどのように対応するのかを示したいと思います．

それでは，Groupクラスに関数を定義しましょう．

```text
def set_payoffs(self):
    p1 = self.get_player_by_id(1)
    p2 = self.get_player_by_id(2)
    p1.payoff = Constants.endowment - self.sent_amount + self.sent_back_amount
    p2.payoff = self.sent_amount * Constants.multiplication_factor - self.sent_back_amount
```

### templates.htmlとviews.pyの定義

続いて，以下の3ページを定義する必要があります．

プレイヤー1の支払額を決める"Send"ページ プレイヤー2の支払額を決める"Send back"ページ 両プレイヤーが見られる"Result"ページ 各ページにゲームのインストラクションが表示されるように設定すると，どのようにゲームを進めることができるのか確認できると思うので，そのようにしましょう．

#### Instruction.html

各ページに表示するインストラクションを作るために，"instruction.html"を作りましょう．

```text
{% load otree_tags staticfiles %}

<div class="instructions well well-lg">

    <h3 class="panel-sub-heading">
        インストラクション
    </h3>
<p>
    これは2人プレイヤーの信頼ゲームです．
</p>
<p>
    はじめにプレイヤーAは {{ Constants.endowment }}を受け取ります．
    プレイヤーBは何も受け取りません．
    プレイヤーAは {{ Constants.endowment }} の中からいくらをプレイヤーBに渡すかを決めます．
    プレイヤーBは，プレイヤーがAの支払額を3倍にしたポイントを受け取ります．
    プレイヤーBが3倍になったポイントを受け取ったら，プレイヤーAにいくら渡すかを決めてください．
</p>
</div>
```

#### Send.html

このページは今まで見てきたテンプレートと似ています．ただ，"

"を使うと，自動的に他のテンプレートを挿入することができます．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %}
    Trust Game: Your Choice
{% endblock %}

{% block content %}

    {% include 'my_trust/Instructions.html' %}

    <p>
    あなたはプレイヤーAです．あなたは {{Constants.endowment}}を持っています．
    </p>

    {% formfield group.sent_amount with label="プレイヤーBに何ポイント渡しますか？" %}

    {% next_button %}

{% endblock %}
```

続いて，"views.py"を定義します．

```text
class Send(Page):

    form_model = models.Group
    form_fields = ['sent_amount']

    def is_displayed(self):
        return self.player.id_in_group == 1
```

テンプレート内の"

"はview.py内の"form\_model"および"form\_fields"と一致している必要があります．

また，"is.displayed\(\)"を使って，"Send.html"がプレイヤー1のみに表示されるようにします．プレイヤー2は"Send.html"を表示しないようにする必要があります．

#### SendBack.html

続いて，プレイヤー2はお金をプレイヤー1に支払う画面を用意します．以下がテンプレートです．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %}
    Trust Game: Your Choice
{% endblock %}

{% block content %}

    {% include 'my_trust/Instructions.html' %}

    <p>
        あなたはプレイヤーBです． プレイヤーAはあなたに {{group.sent_amount}}を渡す決定をしました．
        and you received {{tripled_amount}}.
    </p>

    {% formfield group.sent_back_amount with label="あなたはいくらをプレイヤーAに渡しますか？" %}

    {% next_button %}

{% endblock %}
```

以下をviews.pyに記入しなければいけません．

3倍にしたポイント示す変数"tripled\_amount"をテンプレートに通すために，"vars\_for\_template\(\)"を使います．Djangoはテンプレート上では直接計算をすることはありません．したがって，この数値はPython上で計算をする必要があり，テンプレートに渡してあげる必要があります． プレイヤー1の決定にしたがって，ドロップダウンメニューに動的に変化させるために"sent\_back\_amount\_choices"を定義します．これは"{field\_name}\_choices"と呼ばれるものです．

```text
class SendBack(Page):

    form_model = models.Group
    form_fields = ['sent_back_amount']

    def is_displayed(self):
        return self.player.id_in_group == 2

    def vars_for_template(self):
        return {
            'tripled_amount': self.group.sent_amount * Constants.multiplication_factor
        }

    def sent_back_amount_choices(self):
        return currency_range(
            c(0),
            self.group.sent_amount * Constants.multiplication_factor,
            c(1)
        )
```

#### Results.html

結果のページはプレイヤーAとプレイヤーBで少し違いがあります．そこで，今のプレイヤーの"id\_in\_group"によって条件わけをするために，Djangoで用いられている"

"文を使います．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %}
    Results
{% endblock %}

{% block content %}

{% if player.id_in_group == 1 %}
    <p>
        あなたはプレイヤーBに {{ group.sent_amount }}を渡しました．
        プレイヤーBはあなたに {{group.sent_back_amount}}を渡しました．
    </p>
{% else %}
    <p>
        プレイヤーAはあなたに {{ group.sent_amount }}を渡しました．
        あなたはプレイヤーAに {{group.sent_back_amount}}を渡しました．
    </p>

{% endif %}

    <p>
    したがって，あなたの利得は {{ player.payoff }}です．
    </p>

    {% include 'my_trust/Instructions.html' %}

{% endblock %}
```

views.pyでは，単純に以下のように定義します．

```text
class Results(Page):
    pass
```

#### 待機画面とページ遷移

このゲームでは，2つの待機画面が必要となります．

プレイヤー2はプレイヤー1が支払額を決めるのを待つ必要があります． プレイヤー1はプレイヤー2が支払額を決めるのを待つ必要があります． お互いに決定したあと，利得を計算する必要があります．したがって，"after\_all\_players\_arrive"を使う必要があります．

そこで，これらのページを以下の通り定義します．

```text
class WaitForP1(WaitPage):
    pass

class ResultsWaitPage(WaitPage):

    def after_all_players_arrive(self):
        self.group.set_payoffs()
```

そして，ページ遷移を以下のように定義します．

```text
page_sequence = [
    Send,
    WaitForP1,
    SendBack,
    ResultsWaitPage,
    Results,
]
```

#### settings.pyの中のSESSION\_CONFIGSの追加

```text
{
    'name': 'my_trust',
    'display_name': "My Trust Game (simple version from tutorial)",
    'num_demo_participants': 2,
    'app_sequence': ['my_trust'],
},
```

### データベースのリセットと再起動

コマンドラインに以下の通り入力してデータベースをリセットします．

```text
otree resetdb
otree runserver
```

起動したら，`http://127.0.0.1:8000`にアクセスしてください．

resetdbは新しいアプリケーションを入れたり，models.pyに変更を加えた場合には必ず実施する必要があります．

これは，models.pyは新たにSQLデータベースを作り直す必要があるためです．

ただし，resetdbをするとデータベースが全てリセットされてしまうので，その前にデータを保存しておく必要があります．実用的なレベルで言えば，csvファイルでダウンロードしておくと良いでしょう．

本家本元のURL（このページを翻訳＋αをしています）：

[http://otree.readthedocs.io/en/latest/tutorial/part2.html](http://otree.readthedocs.io/en/latest/tutorial/part2.html)

