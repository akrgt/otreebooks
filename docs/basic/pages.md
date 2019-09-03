# Pages

### Pages

実験参加者が見る各ページは `pages.py` の中にある`Page`クラスで定義されます．

> 2018年1月以前，`pages.py`は`views.py`と呼ばれていました．詳細はoTree2.0のページをご確認ください．

 `pages.py`には，ページの順番を決める`page_sequence`変数があります．例えば，

```text
page_sequence = [Start, Pffer, Accept, Results]
```

もし複数回繰り返しゲームを行うならば，このページ遷移は繰り返されます．詳細はRoundsをご確認ください．



### is\_displayed\(\)

この関数を定義して，ページを表示する必要があるならば`True`に，そして`False`にすればページを飛ばすことができます．省略すると，ページは表示されます．

例えば，ページを各グループのプレイヤー2のみに表示するのであれば，以下のようにします．

```text
class Page1(Page):
    def is_displayed(self):
        return self.player.id_in_group == 2
```

もしくは，第1ラウンドだけ表示するのであれば，以下のようにします．

```text
class Page1(Page):
    def is_displayed(self):
        return self.round_number == 1
```

もし同じルールを複数ページに渡って繰り返す必要があるならば，\[こちら\]\([https://otree.readthedocs.io/en/latest/misc/tips\_and\_tricks.html\#skip-many](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#skip-many)\)を御覧ください．

`id_displayer()`に書いたコードは`True`か`False`を返します．修正されたモデルフィールドのような副次的な効果は持つべきではありません．

### vars\_for\_template\(\)

テンプレートに変数を渡すために，この関数を使ってください．

```text
class Page1(Page):
    def vars_for_template(self):
        return {'a': 1 + 1, 'b': self.player.foo * 10}
```

そうすれば，`a`や`b`には次のようにアクセスすることができます．

```text
Variables {{ a }} and {{ b }} ...
```

oTreeは自動的に以下のオブジェクトをテンプレートに渡しています：`player`，`group`，`subsession`，`participant`，`session`，`Constants`．そして，テンプレートの中にあるそれらのオブジェクトに対して，以下のようにアクセスすることができます：`{{ Constants.blah }}`，\``{{ player.blah }}`．

もし複数のページにある同じ変数にデータを渡す必要があれば，\[こちら\]\([https://otree.readthedocs.io/en/latest/misc/tips\_and\_tricks.html\#vars-for-many-templates](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#vars-for-many-templates)\)を御覧ください．

> `vars_for_template`にランダムな変数を作ってはいけません．なぜならば，もしユーザがページを新しくしてしまったら，`vars_for_template`は再び実行されてしまい，ランダムな計算によって異なる値が返される可能性があります．そのかわり，`creating_session`や`before_next_page`，もしくは`after_all_players_arrive`などを用いれば，ランダムな計算は一度行割れるだけになります．



### before\_next\_page

続いて，プレイヤーが次のステージに進む前にフォームを検証するコードを定義します．

`is_displayed`を用いてページをスキップするのであれば，`before_next_page`も同様にスキップされることになります．

例：

```text
class Page1(Page):
    def before_next_page(self):
        self.player.tripled_payoff = self.player.bonus * 3

```

### template\_name

それぞれのファイルは`template/`の中に同じ名前で保存されます．例えば，アプリが`my_app/page.py`に以下のように記述されたとします．

```text
class Page1(Page):
    pass
```

その時は，`my_app/templates/my_app/Page1.html`にファイルを作る必要があります（my\_appは繰り返されます．）．HTMLテンプレートの書き方はTemplates（準備中）を見てください．

もしテンプレートがあなたのpageクラスとは異なった名前にする必要があるならば（例えば同じテンプレートを複数のページで用いる場合など）`template_name`を設定する必要があります．例えば，以下の通りです．

```text
class Page1(Page):
    template_name = 'app_name/MyView.html'
```

### timeout\_seconds, timeout\_submission, etc...

Timeouts（準備中）をご確認ください．

### Wait pages

Wait pages（準備中）をご確認ください

### Randomizing page sequence

ページの順番をランダマイズすることができます．

※ちなみに，実際にゴトウもやってみました．無事に動きますし，これを使った実験をやってみようと思います．

