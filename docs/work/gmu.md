# 公共財ゲーム

続いて，単純な公共財ゲーム作成しましょう．公共財ゲームとは，経済学における古典的なゲームの1つです．

ここでは初期保有額が100ポイントであり，3人プレイヤーのゲームであるとします．各プレイヤーは独立して，手持ちのポイントをグループに貢献するのかを決めます．各プレイヤーの貢献額を2倍したものを3人に平等に分配します．

公共財ゲームのすべてのコードは[コチラ](https://github.com/oTree-org/oTree/tree/master/public_goods_simple)にあります．

以下の9つのセットアップを行っていきます．

* アップグレード
* アプリケーションの作成
* model.pyの定義
* templateの定義
* views.pyの定義
* settings.pyでのsession configの定義
* データベースをリセットして再起動
* サーバを動かしている間の変更
* ボットの作成

  それではやっていきましょう．

アップグレード 最新バージョンのoTreeを使うために，コマンドラインを開いて，以下のコードを実行しましょう

```text
pip3 install -U otree-core
#再度インストールします．

otree resetdb
#インストールをした際にはデータベースをリセットしましょう．
```

### アプリケーションの作成

cdコマンドを使って，`"requirements_base.txt"`のある，あなたが作ったoTreeプロジェクトフォルダに移動してください．

このディレクトリの中に，公共財ゲーム用のディレクトリを作ってあげます．

`otree startapp my_public_goods`

これによってoTreeプロジェクトフォルダの中に"my\_public\_goods"というフォルダができたことを確認してください．

### model.pyの定義

"model.py"ファイルを開きます．このファイルにはプレイヤー，グループ，サブセッションなどのゲームに関連するモデルとパラメータなどを定義します．

はじめに，公共財ゲームにあわせて，すべてのプレイヤーがすべてのゲームで同様のパラメータになるようにConstantsクラスを修正します．

1つのグループにつき，3人のプレイヤーが必要です．したがって，"players\_per\_group"を3に変更します．oTreeでは自動的にプレイヤーを3人グループに分けてくれます． 各プレイヤーの初期保有額は100ポイントです．したがって，"endowment"を"c\(100\)"として定義します．\("c\(\)"は金額を示しています．\) それぞれの貢献額は2倍にします．したがって，multiplierを2にセットします． そうすると，以下のようなコードが用意できます．

```text
class Constants(BaseConstants):
    name_in_url = 'my_public_goods'
    players_per_group = 3
    num_rounds = 1

    endowment = c(100)
    multiplier = 2
```

ただし，Pythonは大文字と小文字を区別します．したがって，大文字を使うときは気をつけてください．また，インデントも適切に用いてください．例えばif，for，defなどのコードブロック内にあるとき．4つのスペースをインデントする必要があります．

続いて，このゲームで重要な点であるプレイヤーとグループについて検討します．

ゲームを実施した後，各プレイヤーにはどのようなデータが必要でしょうか？重要なのはどのプレイヤーがいくら貢献したかです．したがって，通貨を示す"contribution"フィールドを定義しましょう．

```text
class Player(BasePlayer):
    contribution = models.CurrencyField(min=0, max=Constants.endowment)
```

各グループについて記録を残すとき，どのようなデータに興味があるのでしょうか？もしかしたら，全てのプレイヤーの貢献額\(total contribution\)に興味があるかもしれません．そして貢献額に基づいて算出された金額\(individual share\)が各プレイヤーに分配されます．したがって，これらの2つを定義します．

```text
class Group(BaseGroup):

    total_contribution = models.CurrencyField()
    individual_share = models.CurrencyField()
```

それでは，利得の計算式や，関連する数値を計算します．ここでは"set\_payoffs"と呼びましょう．

```text
class Group(BaseGroup):

    total_contribution = models.CurrencyField()
    individual_share = models.CurrencyField()

    def set_payoffs(self):
        self.total_contribution = sum([p.contribution for p in self.get_players()])
        self.individual_share = self.total_contribution * Constants.multiplier / Constants.players_per_group
        for p in self.get_players():
            p.payoff = Constants.endowment - p.contribution + self.individual_share
```

### templateの定義

このゲームでは2つのページを用意します．

ページ1：プレイヤーはいくら貢献するか決定します． ページ2：プレイヤーは結果を伝えられます． このセクションではゲームを表示するためにHTMLテンプレートを定義します．はじめに，"templates/my\_public\_goods/"ディレクトリ内に2つのHTMLファイルを作成しましょう．

1つは"Contribute.html"です．これはゲームの単純な説明と，プレイヤーが貢献額を入力するフォームフィールドを用意します．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %} Contribute {% endblock %}

{% block content %}

<p>
    これは1グループあたり{{ Constants.players_per_group }} 人の公共財ゲームです．,
    初期保有額は {{ Constants.endowment }}であり，
    全プレイヤーの貢献額の合計が {{ Constants.multiplier }}倍されて，各プレイヤーに平等に分配されます．
</p>


{% formfield player.contribution with label="How much will you contribute?" %}

{% next_button %}

{% endblock %}
```

ちなみに，Pycharmを使って"

"と移動的に入力されます．そして，その間に何を打つべきかサジェッションが表示されます．

そして，もう1つのテンプレートは"Result.html"です．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %} Results {% endblock %}

{% block content %}

<p>
    あなたの初期保有額は {{ Constants.endowment }}です．
    その中から {{ player.contribution }}を貢献しました．
    あなたのグループ全体では {{ group.total_contribution }}が貢献されました．
    その結果，各個人には{{ group.individual_share }}が分配されます．
    したがって，あなたの利得は {{ player.payoff }}です．
</p>

{% next_button %}

{% endblock %}
```

### views.pyの定義

続いて，viewファイルを定義します．これはHTMLテンプレートを表示する方法を示しているものです．

ここでは2つのテンプレートを用意したので，"views.py"の中に2つの"Page"クラスを用意する必要があります．

最初に，"Contribute"を定義します．このページは入力フォームを含んでいるので，"form\_model"と"form\_fields"を定義する必要があります．特に，このフォームは各プレイヤーの"contribution"フィールドを含んでいる必要があります．

```text
class Contribute(Page):

    form_model = models.Player
    form_fields = ['contribution']
```

続いて，"Results"を定義します．このページにはフォームはありません．したがって，何もクラスを定義しません．このような場合は"pass"を使います．

```text
class Results(Page):
    pass
```

これでほとんどできましたが，最後にもう1ページ必要です．プレイヤーが貢献額を決めた後，すぐに結果を知ることはできません．他のプレイヤーが貢献額を決めるのを待つ必要があります．そのために，"WaitPage"という待機画面のページを追加します．

プレイヤーが待機画面にたどり着いた時，他のプレイヤーが待機画面にたどり着くのを待つ必要があります．そして，次ぐのページに進むことになります．

最初に"self.group.set\_payoffs\(\)"と記入します．なぜならば，先程，利得の計算方法を"set\_payoffs"として定義しました．そしてその方法は"Group"クラスの中にあります．これが最初に"self.group"と書く理由です．

```text
class ResultsWaitPage(WaitPage):

    def after_all_players_arrive(self):
        self.group.set_payoffs()
```

そして，"page\_sequence"でページ遷移の順番を並べ替えます．

```text
page_sequence = [
    Contribute,
    ResultsWaitPage,
    Results
]
```

### settings.pyでのsession configの定義

最後に，このプロジェクトのルートディレクトリにある，settings.pyを定義します．SESSION\_CONFIGSの中に以下の内容を追記します．

実験室実験では，実験の後に調査で終わるのが慣例です．そして，そのためにいくらを実験参加者に渡すか示す必要があります．そこで，「実験後調査」と「支払情報」のアプリケーションを"app\_sequence"\(アプリケーション遷移\)に追加する必要があります．

```text
SESSION_CONFIGS = [
    {
        'name': 'my_public_goods',
        'display_name': "My Public Goods (Simple Version)",
        'num_demo_participants': 3,
        'app_sequence': ['my_public_goods', 'survey', 'payment_info'],
    },
    # other session configs ...
]
```

### データベースをリセットして再起動

コマンドラインに以下の通り入力してデータベースをリセットします．

```text
otree resetdb
otree runserver
```

起動したら，`http://127.0.0.1:8000`にアクセスしてください．

### print\(\)を使ったトラブルシューティング

もしコードが適切に動いていなそうであれば，"print\(\)"を使ってPythonプログラムのデバッグをすることができます．ここでは，printを"set\_payoffs"の中に追記します．

```text
def set_payoffs(self):
    self.total_contribution = sum([p.contribution for p in self.get_players()])
    self.individual_share = self.total_contribution * Constants.multiplier / Constants.players_per_group
    for p in self.get_players():
        p.payoff = Constants.endowment - p.contribution + self.individual_share
        print('@@@@@@p.payoff is', p.payoff)
```

アウトプットは"otree runserver"を走らせているコンソールに表示されます．ブラウザには表示されません．

### サーバを動かしている間の変更

一度サーバを動かし始めたら，"Contribution.html"，もしくは"Results.html"を変更してみてください．そして，保存をしてからページを読み込み直してください．変更が即座に反映されます．

### ボットの作成

最後に，今作った実験をシミュレーションするボットを作成しましょう．ボットを作るとタスクを減らすことができます．なぜならば，ゲームに変更があった際に，しっかりと動くかどうか確認することができるからです．

はじめに，"test.py"に移動して，以下のコードを"PlayerBot"クラスに追記してください．

```text
class PlayerBot(Bot):

    def play_round(self):
        yield (views.Contribute, {'contribution': c(42)})
        yield (views.Results)
このボットは最初に貢献額を決定するページで42ポイントを貢献します．その結果が結果を表示する画面に反映されます．
```

コマンドラインでは，以下のコードを入力しましょう．

```text
otree test my_public_goods
```

そして，コマンドラインで実行した結果を確認することができます．ブラウザ上でボットを動かす場合には，"settings.py"に移動して，"'use\_browser\_bots': True"とSESSION CONFIGに入力してください．

こんな感じです．

```text
SESSION_CONFIGS = [
    {
        'name': 'my_public_goods',
        'display_name': "My Public Goods (Simple Version)",
        'num_demo_participants': 3,
        'app_sequence': ['my_public_goods', 'survey', 'payment_info'],
        'use_browser_bots': True
    },
    # other session configs ...
]
```

これで新たなセッションを始めた時，自動的に始めることができます．

....長っ！！

でも，この調子で書き進めよう．

今，WordPressで書いているけどこれで良いのかしら？Qiitaとかの方が．．とも思ったりします．．

本家本元のURL（このページを翻訳＋αをしています）：

[http://otree.readthedocs.io/en/latest/tutorial/part1.html](http://otree.readthedocs.io/en/latest/tutorial/part1.html)

