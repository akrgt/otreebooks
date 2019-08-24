# Templates

アプリの中の`templates/`フォルダはプレイヤーに表示するためのHTMLテンプレートを含む必要があります．

### Template の構文

変数を以下のように示すことができます．

```text
あなたの利益は {{ player.payoff }} です．
```

以下の変数がテンプレートの中では利用可能です．

* `player`：今表示されているページを見ているplayer
* `group`：現在プレイヤーが属しているgroup
* `subsession`：現在プレイヤーが属しているsubsession
* `participant`：現在プレイヤーが属しているparticipant
* `session`：現在のsession
* `Constants`：`models.py`で定義したconstants
* `vars_for_template()`を通じて渡された変数

#### Conditions\("if"\)

```text
{% if player.is_winner %} あなたの勝ち！ {% endif %}
```

もし，`else`節を用いるのであれば次の通り．

```text
{% if some_number >= 0 %}
    ポジティブ
{% else %}
    ネガティブ
{% endif %}
```

#### Loops\(for\)

```text
{% for item in some_list %}
    {{ item }}
{% endfor %}
```

#### Method calls

modelsからmethodを呼び出すために，\(\)を除くようにしてください（通常のpythonコードとは異なります）．

```text
class Player(BasePlayer):
    def doubled_payoff(self):
        return self.payoff * 2
```

```text
あなたの利益は2倍になり {{ player.doubled_payoff }}となりました．
```

#### list，もしくはdictへアクセスするアイテム

Pythonコードの中で`my_list[0]`と`my_dict['foo']`として記述したものは，テンプレートの中では`{{ my_list.0 }}`と`{{ my_dict.foo }}`として記述することになります．

#### Comments

```text
{% comment %}
複数行の
コメントは
こんな感じ．
{% endcomment %}
```

#### Template filters

Djangoテンプレート言語で利用可能なフィルタに加えて，oTreeでは`c()`と同様の`|c`フィルターを用いることができます．例えば， `{{ 20 | c }}`は`20ポイント`として表示されます．

同様に，`|abs`フィルターは絶対値を返します． `{{ -20 | c }}`と打ち込めば`20ポイント`として表示されます．

もし，"Invalid filter"エラーが表示されたら，テンプレートの上に`{% load otree %}`と設定できているかどうかを確認してください．


#### できないこと

テンプレート言語は単純に表示と値をループさせるように設計されています．ほとんどのその他のことはできるようになっていません．例えば，足し算，引き算，掛け算，割り算などの計算やその他の数字，リスト，ストリングスなどの修正はできません．

### How templates work: an example

oTreeテンプレートは2つの言語が混ざっています．

* _HTML_（`<this>`や`</this>`のように，角括弧で表記する形式）
* _Django template tags_（`{% this %}`や`{{ this }}`のように，中括弧で表記する形式）

以下には2つの言語を同時に動かす例を示します．この例では，テンプレートはこのように見えます．

```text
<p>今期のあなたの利益は {{ player.payoff }}です．</p>

{% if subsession.round_number > 1 %}
    <p>
        1期前のあなたの利益は {{ last_round_payoff }}です．
    </p>
{% endif %}

{% next_button %}
```

### Step1：oTreeはDjangoタグを読み込んで，HTMLを生成します（a.k.a. "サーバサイド"）

oTreeは（vars\_for\_template\(\)から提供された）変数の現在の値を，DjangoコードからプレーンHTMLに変換します．

```text
<p>あなたの利益は 10 　ポイントでした．</p>

    <p>
        あなたの前期の利益は $5 ポイントでした．
    </p>

<button class="otree-btn-next btn btn-primary">Next</button>
```



### Step2：ブラウザはHTMLタグを読み込んで，webページを生成します（a.k.a. "クライアントサイド"）

oTreeサーバはブラウザがコードを読み込んで，形式的に整ったページとして表記できるようにしたHTMLをユーザのコンピュータに送ります．

ただし，Djangoタグはブラウザからは決して見えません．

#### キーポイント

この例から得られるポイントは，もしページがあなたの思ったように表示されなければ，上記のステップのどこかが間違っていることになります．ブラウザ上で右クリックをして，"ソースの表示"をしてみてください（"ソースの表示"は分割表示モードではうまくいかないかもしれません）

* もしHTMLコードがあなたの思ったようなものでなければ，サーバサイドで何か不具合があるのかもしれません．`vars_for_template`か，djangoテンプレートタグにミスがないか確認してください．
* もし生成したHTMLコードにエラーがなければ，恐らく使っているHTML，もしくはJavaScriptの構成の問題です．問題があると思われる部分のHTMLをテンプレートの後ろの方に，Djangoタグなしに貼り付けてみてください．そして，正しいアウトプットが出てくるまで試してみてください．そうしたら，djangoタグの中に戻して，再び動学的に動かしてみてください．

### Template blocks

完璧なHTMLをページに打ち込む代わりに：

```text
<!DOCTYPE html>
    <html lang="en">
        <head>
            <!-- and so on... -->
```

2つのブロックを定義してください．

```text
{% block title %} タイトルはここへ {% endblock %}

{% block content %}
    Body HTML goes here.

    {% formfield player.contribution label="あなたの貢献額はいくらですか？" %}

    {% next_button %}
{% endblock %}
```



### Images, videos, CSS, JavaScript, etc\(static files\)

ここにはstatic file\(.png, .jpg, .mp4, .css, .js, etc.\)をページに入れる方法を紹介します．

oTreeプロジェクトのルートには，`_static/`フォルダがあります．ここに例えば`puppy.jpg`などのファイルを入れてください．そしてテンプレートには，そのファイルへのURLを`{% static 'puppy.jpg' %}`として表示することができます．

画像を表示するためには，以下のよう`<img>`にタグを使ってください．

```text
<img src="{% static "puppy.jpg" %}"/>
```

上記の通り，`_static/puppy.jpg`に画像を保存しましたが，実際には`_static/your_app_name/puppy.jpg`といったように使っているアプリと同じ名前のフォルダを作って保存をした方が良いです．これはファイルを整理するため，さらに名前の競合を防ぐためです．

そうすれば，HTMLコードは以下の通りになります．

```text
<img src="{% static "your_app_name/puppy.jpg" %}"/>
```

もし，望むのであれば，アプリの中に`static/your_app_name`のようにstaticファイルを置くこともできます．もしstaticファイルが変更を加えた後にも変わらないようであれば，ブラウザがファイルをキャッシュしているからです．そのような場合は完全にリロードしてください（たいてい，Ctrl + F5でできます）．

### Dynamic images

もし画像や動画のパスが変数であれば（各ラウンドで画像が変わるなど），`pages.py`でそのようにすることができますし，テンプレートに渡すことも可能です．

```text
class MyPage(Page):

    def vars_for_template(self):
        return {
            'image_path': 'my_app/{}.png'.format(self.round_number)
        }
```

そしてテンプレートは以下の通り．

```text
<img src="{% static image_path %}"/>
```



### JavaScript and CSS

#### JavaScript/CSSのコードはどこに置くべきか

JS/CSSコードを\(a\)グローバルに動かすか，\(b\)一つのアプリのみに適用するか，それとも\(c\)1つのページのみに適用するかによって異なります．

#### Globally

oTreeプロジェクトのルートには，`_templates/`フォルダがあります（各アプリにある`templates/`フォルダと間違えないように気をつけてください）．スタイルやスクリプトをすべてのゲームのすべてのページに適用するためには，`_templates/global/Page.html`を修正してください．どんなスクリプトも`{% block global_scripts %}`の中に記述し，どんなスタイルも`{% block global_styles %}`の中に記述すれば動きます．

#### For one app

スタイルやスクリプトを1つのアプリのすべてのページに適用するためには，基礎となるテンプレートを作成して，`{% block app_styles %}` の中，もしくは`{% block app_scripts %}`の中に記述してください．

例えば，アプリの名前が`public_goods`だとしたら，その時は`public_goods/templates/public_goods/Page.html`を作成し，その中に次のように書きます．

```text
{% extends "global/Page.html" %}
{% load otree %}

{% block app_styles %}

    <style>
        CSS はここへ
    </style>

{% endblock %}
```

そして，それぞれの`public_goods`テンプレートはこのテンプレートを継承します．

```text
{% extends "public_goods/Page.html" %}
{% load otree %}
...
```

#### Just one page

もしJavaScriptと，もしくはCSSを1つのページに適用する必要があるならば，直接`content`ブロックに記述するか，よりよく整理するためには，`scripts`と`styles`と呼ばれるブロックの中に記述する必要があります．それらは以下のように，`content`ブロックの外に置く必要があります．

```text
{% block content %}
    <p>This is some HTML.</p>
{% endblock %}

{% block scripts %}

    <script>
        alert('hello world');
    </script>

    <script src="{% static "my_app/script.js" %}"></script>
{% endblock %}
```

これらは義務ではありません．しかし，以下の点からメリットは大きいと思われます．

* コードを整理できる．
* 物事を正しい順番に整理できる（CSS，ページのコンテンツ，最後にJavaScript）

#### テーマのカスタマイズ

もしoTreeの各要素をカスタマイズする必要があるならば，以下のリストがCSSのセレクターです．

| Element | CSS/jQueryセレクター |
| :--- | :--- |
| ページボディ | `.otree-body` |
| ページタイトル | `.otree-title` |
| Wait page\(ダイアログすべて\) | `.otree-wait-page` |
| Wait pageダイアログタイトル | `.otree-wait-page__title`\(\_\_であり，\_ではない\) |
| Wait pageダイアログボディ | `.otree-wait-page__body` |
| タイマー | `.otree-timer` |
| 次へボタン | `.otree-btn-next` |
| フォームエラー警告 | `.otree-form-errors` |

例えば，ページの幅を変えるには，基礎テンプレートに以下のCSSを記述してください．

```text
<style>
    .otree-body {
        max-width:800px
    }
</style>
```

もっと情報を得たければ，ブラウザで修正したいところの要素を右クリックをして，"検証"を選んでください．そうすれば，異なった要素とそれらのスタイルを修正することができます．

可能であれば，公式なセレクターの1つを選んでください．決して`_otree`からはじまるセレクターを使ってはいけませんし，`btn-primary`や`card`のようなブートストラップクラスを選んではいけません．なぜならばそれらは不安定だからです．

アンダーバーは2つです（`__`であり，`_`ではない）．

#### PythonからJavaScript\(json\)にデータを移すには

JavaScriptコードに変数を入れる必要があるならば，`{{ my_variable }}`という書き方よりも，`{{ my_variable|json }}`という書き方の方が望ましいです．

例えば，もしプレイヤーの利得をJavaSciptに移す必要があるならば，次のようにあ表すことができます．

```text
<script>
    var payoff = {{ player.payoff|json }};
    ...
</script>
```

もし`|json`を使わないならば，変数は正しくJavaScriptでは機能しないかもしれません．

| Python | `|json`を使わないテンプレート | `|json` |
| :--- | :--- | :--- |
| `None` | `None` | `null` |
| `3.14` | `3,14` | `3.14` |
| `c(3.14)` | `$3.14` or `$3,14` | `3.14` |
| `True` | `True` | `true` |
| `"a"` | `a` | `"a"` |
| `{'a': 1}` | `{&#39;a&#39;: 1}` | `{'a': 1}` |
| `['a']` | `[&#39;a&#39;]` | `['a']` |

`|json`は`1`のような単純な値として扱われますが，辞書や`{'a': [1,2]}`のようなリストとしても扱われます．

`|json`はJOSONや問題のない（信頼のある）データに変換されます．したがって，Djangoは自動的に間接表現に置き換えることがありません．

上記の表に示したように，`|json`は自動的にストリング周りに引用符をつけるので，手動でそれらを追加する必要がありません．

```text
// correct
var my_string = {{ my_string|json }};

// incorrect
var my_string = "{{ my_string|json }}";
```

もし，"無効なフィルター"エラーが生じたら，`{% load otree %}`をテンプレートの一番上に記述していることを確認してください．

注意：`|json`テンプレートフィルターは古い`safe_json`関数を置き換えました．しかしながら，`safe_json`は今でも機能します．必ず一方を利用して，両者を同時に利用することがないようにしてください，

### Bootstrap

oTreeはウェブサイトのUIをカスタマイズするためのライブラリである\[Bootstrap\]\([https://getbootstrap.com/docs/4.0/components/alerts/](https://getbootstrap.com/docs/4.0/components/alerts/)\)を用いています．

> oTree2.0\(2018年1月\)より，Bootstrap3からBootstrap4にアップデートしています．

カスタマイズしたスタイルやテーブルやアラート，プログレスバー，ラベルなどの特別な要素を使うこともできます．さらに，ポップアップやユーザが応答しなければ反応しないようにするモーダル，それに折りたたみ式のテキストを入れることができます．

Bootstrapを使うために，基本的にはHTMLに`class=`を追加する必要があります．

例えば，下のHTMLは"サクセス"アラートを入れることができます．

```text
<div class="alert alert-success">Great job!</div>
```

#### モバイル端末

Bootstrapはスマートフォンやタブレットで見た時には，モバイルバージョンを表示することができます．

#### Charts

チャートをアプリに追加するために，HTML/JavaScriptのライブラリを使うことができます．

特に円グラフや折れ線グラフ，棒グラフ，時系列等を表示するためにHighChartsを使うことを勧めます．一部のoTreeのサンプルゲームではHighChartsを使っています．

最初に，HighChartsのJavaScriptを入れる必要があります．

```text
<script src="https://code.highcharts.com/highcharts.js"></script>
```

　もし，HighChartsを複数の場所に使うのであれば，`app_scripts`や`global_scripts`に記述することもできます．もっと詳しい情報は上記をご確認ください（しかし，HighChartsはページの動きを遅くします．

HighChartsの\[デモページ\]\([https://www.highcharts.com/demo](https://www.highcharts.com/demo)\)に行き，作りたい形のシャートを見つけてください．そして，"edit in JSFiddle"をクリックして，好みのプログラムで変更されることがない，置き換えられることのない変数を使って編集できます．

その後，JSとHTMLをあなたのテンプレートに貼り付けて，ページを読み込んでください．もし，チャートが見えないようであれば，HTMLはJSコードがチャートを挿入しようとする`<div>`クラスが抜けている可能性があります．

一度適切にチャートを挿入することができれば，動的に生成される`series`や`categories`のような変更されないデータに置き換えることができます．

例えば，これを

```text
series: [{
    name: 'Tokyo',
    data: [7.0, 6.9, 9.5, 14.5, 18.2, 21.5, 25.2, 26.5, 23.3, 18.3, 13.9, 9.6]
}, {
    name: 'New York',
    data: [-0.2, 0.8, 5.7, 11.3, 17.0, 22.0, 24.8, 24.1, 20.1, 14.1, 8.6, 2.5]
}]
```

以下のように変えられます．

```text
series: {{ highcharts_series|json }}
```

pageクラスの`vars_for_template`では，Pythonの中でネストされた構造のデータを作り出し（上記の例は辞書リストの例です），テンプレートに渡します．ただし，JavaScriptを挿入する時には，どんな変数であっても`|json`フィルターを使うことを忘れないでください．

もしチャートがロードできない場合は，"ソースを見る"をブラウザで確認し，動的に生成されるデータにどこか誤りがないか確認をしてください．もし`{&#39;a&#39;: 1}`のような文字化けをしているのであれば，`|json`フィルターを使い忘れているかもしれません．

### PyCharm Professionalを使う場合の注意：

もしPyCharmの通常バージョン\(Community Edition\)を使われているのであれば，PyCharm Professional Editionにアップグレードすることを視野に入れてください．なぜならば，DjangoテンプレートとJavaScriptの記法にハイライトすることができます．

PyCharm Professionalは学生や教員，教授であれば無料で使えます．

Professional Editionをインストールしたら，settingsで`Languages & Frameworks -> Django`に移動してください．そして，"Enable Django Support"にチェックを入れてください．そして`manage.py`と`settings.py`のあるoTreeフォルダーをDjangoプロジェクトのルートとしてください．



[https://otree.readthedocs.io/en/latest/templates.html](https://otree.readthedocs.io/en/latest/templates.html)
