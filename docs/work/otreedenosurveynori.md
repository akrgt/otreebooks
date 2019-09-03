# oTreeでのSurveyの作り方：

すみません，仕事が慌ただしく．．．

oTreeの強みはオンライン上でインタラクションのある経済ゲーム実験を実施できる，という点にあるのですが，それだけではありません．

単純にインタラクションのない個人の意思決定を調査する際にも使えます．

ここでは，インタラクションのある経済ゲーム実験を作るための基礎，という側面も含めてSurveyを作ってみましょう．

アプリケーションの作成

最初にアプリケーションを作ります．

はじめに，"requirements\_base.txt"があるディレクトリに移動します．

移動する際にはoTreeディレクトリの中に入る必要があるので，

`cd oTree` で移動しましょう．

このディレクトリの中に，調査アプリ用のディレクトリを作ってあげます．

`otree startapp my_survey`

これによって調査用アプリを作ることができます．

※ただし，普段私は手抜きなので，他のアプリをコピペして作り始めることが多いです．ただ，幾つか落とし穴はあるので，気をつけないといけないかもしれません．

こうすると，「my\_survey」というフォルダが出来上がります．

ここから，

models.py template views.py settings.py の4つをセットアップしていきます．

### models.pyを定義する

models.pyを開き，class Player\(BasePlayer\):をスクロールして探してください．ここに各プレイヤー（実験参加者）について，データベースに保存するフィールドを作成します．ここでは以下の2つのフィールドを保存します．

name：これはcharfield，文字列を保存します． age：ここでは正の整数を保存します．

```text
class Player(BasePlayer):
    name = models.CharField()
    age = models.PositiveIntegerField()
```

templateを定義する この調査は2ページに渡るものとします：

Page 1：プレイヤーが名前と年齢を入力します． Page 2：プレイヤーが前のページで入力したデータを確認します． このセクションでは，実験参加者は実験者に見えるHTMLテンプレートを定義します．

この2つのHTMLファイルをtemplates/my\_simple\_survey/の中に作成します．

最初のページの名前をMyPage.htmlとします．そして，以下の内容をファイルの中に記入します．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %}
    Enter your information
{% endblock %}

{% block content %}

    {% formfield player.name with label="Enter your name" %}

    {% formfield player.age with label="Enter your age" %}

    {% next_button %}

{% endblock %}
```

そして，2つ目のファイルはResult.htmlという名前にします．

```text
{% extends "global/Page.html" %}
{% load staticfiles otree_tags %}

{% block title %}
    Results
{% endblock %}

{% block content %}

    <p>Your name is {{ player.name }} and your age is {{ player.age }}.</p>

    {% next_button %}
{% endblock %}
```

### views.pyを定義する

続いて，viewファイルを定義します．これはHTMLテンプレートを表示する方法を示しているものです．

先程，2つのテンプレートを容易いたしました．そのために2つのPageクラスをviews.pyの中に定義する必要があります．定義する際の名前は2つのHTMLテンプレート\(MyPageとResults\)に一致させる必要があります．

最初に，MyPageを定義しましょう．このページはフォームを含んでいます．したがって，form\_modelとform\_fieldsを定義する必要があります．

特に，このフォームはnameフィールドとageフィールドをプレイヤーに準備する必要があります．

```text
class MyPage(Page):
    form_model = models.Player
    form_fields = ['name', 'age']
```

続いて，Resultsページを定義します．このページはフォームを含んでいませんので，ただパスする宣言をするだけで構いません．

```text
class Results(Page):
    pass
```

もし，views.pyの中に既にWaitPageがあるならば，そのページは消してしまってください．なぜならば，WaitPageは複数人プレイヤーのゲームや，より複雑なゲームにおいて必要とされるからです．

その後，page\_sequenceを設定します．これにより，MyPageの後にResultsを表示できるようにします．これについては，views.pyの中に定義する必要があります．

したがって，最終的にviews.pyはこのような内容になります．

```text
from otree.api import Currency as c, currency_range
from . import models
from ._builtin import Page, WaitPage
from .models import Constants


class MyPage(Page):
    form_model = models.Player
    form_fields = ['name', 'age']


class Results(Page):
    pass


page_sequence = [
    MyPage,
    Results
]
```

### settings.pyを定義する

最後に，このプロジェクトのルートディレクトリにある，settings.pyを定義します．SESSION\_CONFIGSの中に以下の内容を追記します．

```text
SESSION_CONFIGS = [
    {
        'name': 'my_simple_survey',
        'display_name': "調査を作ってみた",
        'num_demo_participants': 1,
        'app_sequence': ['my_simple_survey'],
    },
    # other session configs go here ...
]
```

データベースをセットして起動する

```text
otree resetdb
otree runserver
```

起動したら，`http://127.0.0.1:8000`にアクセスしてください．

エラーの修正

もしエラーがあれば．コマンドラインに"traceback"（エラーメッセージ）が表示されます．

```text
C:\oTree\chris> otree resetdb
Traceback (most recent call last):
  File "C:\oTree\chris\manage.py", line 10, in <module>
    execute_from_command_line(sys.argv, script_file=__file__)
  File "c:\otree\core\otree\management\cli.py", line 170, in execute_from_command_line
    utility.execute()
  File "C:\oTree\venv\lib\site-packages\django\core\management\__init__.py", line 328, in execute
    django.setup()
  File "C:\oTree\venv\lib\site-packages\django\__init__.py", line 18, in setup
    apps.populate(settings.INSTALLED_APPS)
  File "C:\oTree\venv\lib\site-packages\django\apps\registry.py", line 108, in populate
    app_config.import_models(all_models)
  File "C:\oTree\venv\lib\site-packages\django\apps\config.py", line 198, in import_models
    self.models_module = import_module(models_module_name)
  File "C:\Python27\Lib\importlib\__init__.py", line 37, in import_module
    __import__(name)
  File "C:\oTree\chris\public_goods_simple\models.py", line 40
    self.total_contribution = sum([p.contribution for p in self.get_players()])
       ^
IndentationError: expected an indented block
```

こんな画面が出てきたときに，最初にすべきことは「最後の一文」に注目しましょう．

ここでは`""C:\oTree\chris\public_goods_simple\models.py", line 40"`に何らかのエラーが生じていることを示しています．そして，tracebackが指摘しているエラーを修正してください．

`"IndentationError: expected an indented block"`は「インデントが必要な場所に入力されていない」ということを示しています．PyCharmのようなPythonエディタはエラーが生じた場所に赤線などが入るのでエラーを見つけやすくなっています．エラーを直したら，再度走らせてみましょう．

時折，tracebackの最後の行に必要なコードが入っていないことを指摘してくれる場合があります．

例えば，以下のtracebackでは最後の行で"/site-packages/easymoney.py"について言及されています．これは，作成したアプリケーションではなく，Pythonの外部パッケージにエラーが生じていることを意味しています．

```text
Traceback:
File "/usr/local/lib/python3.5/site-packages/django/core/handlers/base.py" in get_response
  132.                     response = wrapped_callback(request, *callback_args, **callback_kwargs)
File "/usr/local/lib/python3.5/site-packages/django/views/generic/base.py" in view
  71.             return self.dispatch(request, *args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/django/utils/decorators.py" in _wrapper
  34.             return bound_func(*args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/django/views/decorators/cache.py" in _wrapped_view_func
  57.         response = view_func(request, *args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/django/utils/decorators.py" in bound_func
  30.                 return func.__get__(self, type(self))(*args2, **kwargs2)
File "/usr/local/lib/python3.5/site-packages/django/utils/decorators.py" in _wrapper
  34.             return bound_func(*args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/django/views/decorators/cache.py" in _cache_controlled
  43.             response = viewfunc(request, *args, **kw)
File "/usr/local/lib/python3.5/site-packages/django/utils/decorators.py" in bound_func
  30.                 return func.__get__(self, type(self))(*args2, **kwargs2)
File "/usr/local/lib/python3.5/site-packages/otree/views/abstract.py" in dispatch
  315.                 request, *args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/django/views/generic/base.py" in dispatch
  89.         return handler(request, *args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/otree/views/abstract.py" in get
  814.         return super(FormPageMixin, self).get(request, *args, **kwargs)
File "/usr/local/lib/python3.5/site-packages/vanilla/model_views.py" in get
  294.         context = self.get_context_data(form=form)
File "/usr/local/lib/python3.5/site-packages/otree/views/abstract.py" in get_context_data
  193.         vars_for_template = self.resolve_vars_for_template()
File "/usr/local/lib/python3.5/site-packages/otree/views/abstract.py" in resolve_vars_for_template
  212.         context.update(self.vars_for_template() or {})
File "/Users/chris/oTree/public_goods/views.py" in vars_for_template
  108.             'total_payoff': self.player.payoff + Constants.fixed_pay}
File "/usr/local/lib/python3.5/site-packages/easymoney.py" in <lambda>
  36.     return lambda self, other, context=None: self.__class__(method(self, _to_decimal(other)))
File "/usr/local/lib/python3.5/site-packages/easymoney.py" in _to_decimal
  24.         return Decimal(amount)

Exception Type: TypeError at /p/j0p7dxqo/public_goods/ResultsFinal/8/
Exception Value: conversion from NoneType to Decimal is not supported
```

これにハマるとしんどい．．．．

このような場合，外部のパッケージではなく，自分で作成したコードに対する言及がないかどうかを調べてみます．

探してみると，自身で作成したパッケージの一部である"/Users/chris/oTree/public\_goods/views.py"に問題があることが推測されます．バグは108行目にいることが示唆されます．

・・・頻度を上げたいけど，学期が始まるとなかなか手が回りません．．．

頑張ります．

本家本元のURL（このページを翻訳＋αをしています）：

[http://otree.readthedocs.io/en/latest/tutorial/part0.html](http://otree.readthedocs.io/en/latest/tutorial/part0.html)

