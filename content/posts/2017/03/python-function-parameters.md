---
title: "Python3에서 함수의 인자 다루기"
date: 2017-03-03T20:35:00+09:00
tags : ["python"]
categories : ["develop"]

aliases:
    - /develop/2017/03/03/python-function-parameters.html
---

flask를 써보며 [Variable Rules](http://flask.pocoo.org/docs/0.12/quickstart/#variable-rules)가 어떻게 구현되는지 궁금했었는데,
어제 [전문가를 위한 파이썬](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=88728476)(루시아누 하말류 저, 강권학 역, 원제: [Fluent Python](https://www.amazon.com/Fluent-Python-Concise-Effective-Programming/dp/1491946008/ref=sr_1_1?ie=UTF8&qid=1488470572&sr=8-1&keywords=Fluent+Python))을 읽고 어떤 방법으로 구현될 수 있는지 알 게 되었습니다.

python에는 `inspect`라는 라이브러리가 제공된다. 이 라이브러리의 `signature`를 이용해 어떤 인자가 있는지, 그 인자가 어떤 속성을 가지는지 알 수 있습니다.
    
{{< highlight python >}}
from inspect import signature


def sample(a, b=10, *args, c=None, **kwargs):
    pass

sig = signature(sample)
for param in sig.parameters.values():
    pass
{{< / highlight >}}

`signature`의 프로퍼티 `parameters`는 각 파라미터가 `param_name: inspect.Parameter` 형식으로 들어있는 [OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict) 객체다.
파라미터의 이름은 [inspect.Parameter](https://docs.python.org/3/library/inspect.html#inspect.Parameter) 객체를 통해 확인할 수 있으니 오브젝트만 불러와서 정보를 확인하면 됩니다.

inspect.Parameter에서 확인해야할 프로퍼티는 `name`, `default` 그리고 `kind`입니다.

{{< highlight python >}}
from inspect import signature


def sample(a, b=10, *args, c=None, **kwargs):
    pass

sig = signature(sample)
for param in sig.parameters.values():
    print(param.name)
    print(' -', param.default)
    print(' -', param.kind)
{{< / highlight >}}

위의 코드를 실행하면 결과는 아래와 같이 출력됩니다.

```
a
 - <class 'inspect._empty'>
 - POSITIONAL_OR_KEYWORD
b
 - 10
 - POSITIONAL_OR_KEYWORD
args
 - <class 'inspect._empty'>
 - VAR_POSITIONAL
c
 - None
 - KEYWORD_ONLY
kwargs
 - <class 'inspect._empty'>
 - VAR_KEYWORD
```

`name`과 `default`는 이름만으로도 용도가 짐작가듯이 인자의 이름과 기본값을 뜻합니다. 그런데 `sample` 함수의 경우 `a`, `*args`, `**args`는 기본값이 설정되지 않았습니다. 프로퍼티 값으로는 `inspect._empty`라는 결과가 나오는데 이를 통해 기본값이 설정되지 않은 것을 확인할 수 있습니다.
이 값이 왜 `None`이 아닌지는 `c` 파라미터를 통해 볼 수 있습니다. 기본값으로는 `None`도 사용가능하기 때문에 따로 선언되어 있습니다.

그럼 이 값이 `empty`인지는 어떻게 확인할까요? `inspect._empty`를 `import`해 확인해야 할까요?
다행히 확인하기 쉽게 `inspect.Parameter`의 프로퍼티로 `empty`가 있습니다. 따라서 `param.default is param.empty`만으로 기본값이 있는지 없는지를 확인해 볼 수 있습니다.

다음은 `kind`입니다. `kind`는 파라미터가 어떤 종류인지를 확인할 수 있습니다. 선언해 둔 `def sample(a, b=10, *args, c=None, **kwargs)`을 `*args`와 `**kwargs`를 볼 수 있습니다. 둘은 파이썬에서 지원하는 파라미터 관련 기능인데,
인자명 앞에 `*`을 붙이면 [키워드가 지정되지 않고 파라미터가 정의 되지 않은 인자](https://docs.python.org/3/tutorial/controlflow.html#unpacking-argument-lists)를 모두 가져올 수 있습니다.
그리고 `**`를 붙이면 [키워드가 지정된 파라미터가 정의되지 않은 인자](https://docs.python.org/3/tutorial/controlflow.html#keyword-arguments)를 가져올 수 있습니다. 말로는 이해가 쉽지 않으니 `sample` 함수를 조금 바꾼 뒤 확인 해보도록 하겠습니다.

{{< highlight python >}}
>>> def sample(a, b=10, *args, c=None, **kwargs):
...     print('a', a)
...     print('b', b)
...     print('c', c)
...     print('args', args)
...     print('kwargs', kwargs)
...
>>> sample(1, 2, 3, 4, c=5, d=6, e=7)
a 1
b 2
c 5
args (3, 4)
kwargs {'e': 7, 'd': 6}
{{< / highlight >}}

이 결과를 위에서 본 `kind` 정보와 함께 보겠습니다.

| 인자명 | 값 | kind |
| --- | --- | --- |
| a | 1 | POSITIONAL_OR_KEYWORD |
| b | 2 | POSITIONAL_OR_KEYWORD |
| c | 5 | KEYWORD_ONLY |
| args | (3, 4) | VAR_POSITIONAL |
| kwargs | {'e': 7, 'd': 6} | VAR_KEYWORD |

`a`와 `b`는 1, 2가 순서대로 들어갔습니다.
하지만 `c`는 값을 지정한 5가 들어갔는데, 앞에 `*args`를 선언해서 위치만으로는 입력이 되지 않기 때문입니다. 이러한 정보는 `kind`로 확인할 수 있습니다.
`c`를 보면 `KEYWORD_ONLY`로 지정되어 있습니다. `a`와 `b`는 `POSITIONAL_OR_KEYWORD`인데 이걸로 확인할 수 있듯이 `c`는 키워드로만 인자를 쓸 수 있고, `a`와 `b`는 키워드를 지정하지 않아도 쓸 수 있음을 알 수 있습니다.
`kind` 타입 목록은 [kind에 대한 설명](https://docs.python.org/3/library/inspect.html#inspect.Parameter.kind)에서 확인할 수 있습니다.
단, `POSITION_ONLY`는 아직 [논의중](https://www.python.org/dev/peps/pep-0457/)입니다.

위의 목록을 보면 `args`는 `VAR_POSITIONAL`가, `kwargs`는 `VAR_KEYWORD`로 되어있는 것을 볼 수 있습니다. 여기까지 확인했으면 간단한 데코레이터를 만들어 보겠습니다.

{{< highlight python >}}
from functools import wraps
from inspect import signature

    
def param_info(func):
    sig = signature(func)
    for param in sig.parameters.values():
        print(param.name)
        print(' -', param.default)
        print(' -', param.kind)


def safe_param(func):
    ok_args = False
    ok_kwargs = False
    
    list_params = []
    keyword_params = set()
    
    sig = signature(func)
    for param in sig.parameters.values():
        if param.kind == param.VAR_POSITIONAL:
            ok_args = True
        if param.kind == param.VAR_KEYWORD:
            ok_kwargs = True
            
        if param.kind in [param.POSITIONAL_OR_KEYWORD]:
            list_params.append(param.name)
        if param.kind in [param.POSITIONAL_OR_KEYWORD, param.KEYWORD_ONLY]:
            keyword_params.add(param.name)
            
    def get_default_value(param_name):
        original = sig.parameters[param_name]
        no_default = original.default is original.empty
        return None if original.default is original.empty else original.default
            
    @wraps(func)
    def wrap(*args, **kwargs):
        if not ok_args:
            args = args[:len(list_params)]
        
        if not ok_kwargs:
            temp = {k: v for k, v in kwargs.items() if k in keyword_params}
            kwargs = temp
        
        if len(args) < len(list_params):
            not_set_list_params = list_params[len(args):]
            for param in not_set_list_params:
                if param in kwargs:
                    continue
                    
                kwargs[param] = get_default_value(param)
        
        not_set_keyword_params = keyword_params - set(list_params) - set(kwargs.keys())
        for param in not_set_keyword_params:
            kwargs[param] = get_default_value(param)
        
        return func(*args, **kwargs)
    return wrap
{{< / highlight >}}

`param_info` 함수는 위에서 파라미터 정보를 볼 때 사용한 코드를 함수로 바꾼 겁니다. 아래의 `safe_param` 함수는 인자로 값을 넣지 않아도 자동으로 `None`으로 설정하고, 혹은 `*args`나 `**kwargs`가 없어도 너무 많이 넣거나 정의되지 않은 인자는 제외해주는 함수입니다.

{{< highlight python >}}
>>> def sample(a, b, *args, c, **kwargs):
...     print('a', a)
...     print('b', b)
...     print('c', c)
...     print('args', args)
...     print('kwargs', kwargs)
... 
>>> safe_sample = safe_param(sample)    
>>> param_info(sample)
a
 - <class 'inspect._empty'>
 - POSITIONAL_OR_KEYWORD
b
 - <class 'inspect._empty'>
 - POSITIONAL_OR_KEYWORD
args
 - <class 'inspect._empty'>
 - VAR_POSITIONAL
c
 - <class 'inspect._empty'>
 - KEYWORD_ONLY
kwargs
 - <class 'inspect._empty'>
 - VAR_KEYWORD
>>> param_info(safe_sample)
a
 - <class 'inspect._empty'>
 - POSITIONAL_OR_KEYWORD
b
 - <class 'inspect._empty'>
 - POSITIONAL_OR_KEYWORD
args
 - <class 'inspect._empty'>
 - VAR_POSITIONAL
c
 - <class 'inspect._empty'>
 - KEYWORD_ONLY
kwargs
 - <class 'inspect._empty'>
 - VAR_KEYWORD
{{< / highlight >}}

`safe_param`으로 데코레이팅 해도 [functools.wraps](https://docs.python.org/3/library/functools.html#functools.wraps)를 사용했기 때문에 파라미터 정보는 그대로 추출 가능합니다. 잘 돌아가는지 확인해보겠습니다.

{{< highlight python >}}
>>> safe_sample(1,2,3,4,5,6,7, d=10)
a 1
b 2
c None
args (3, 4, 5, 6, 7)
kwargs {'d': 10}
>>> sample(1,2,3,4,5,6,7, d=10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sample() missing 1 required keyword-only argument: 'c'
{{< / highlight >}}

`safe_param`으로 데코레이팅한 함수는 문제 없이 작동합니다. `c`는 `KEYWORD_ONLY`라서 3이 아닌 기본 값 `None`이 설정되었습니다. 데코레이팅 하지 않은 원본 함수는 `c` 파라미터가 없어서 에러가 납니다. 위의 데코레이터가 잘 작동하는 것을 확인할 수 있습니다.

{{< highlight python >}}
>>> def sample2(a, b, *, c):
...     print('a', a)
...     print('b', b)
...     print('c', c)
...
>>> safe_sample2 = safe_param(sample2)
>>> 
>>> safe_sample2(1,2,3,4,5,6,7, d=10)
a 1
b 2
c None
>>> sample2(1,2,3,4,5,6,7, d=10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: sample2() got an unexpected keyword argument 'd'
{{< / highlight >}}

이번에는 조금 수정해서 `*args`와 `**kwargs`를 제외하고 함수를 작성해 봤습니다. `*args`와 `**kwargs`가 없어도 잘 동작하는 것을 확인할 수 있습니다. 조금 더 수정해서 기본값을 설정할 수 있는 데코레이터로 만들어 보겠습니다.

{{< highlight python >}}
def safe_param(default=None):
    def deco(func):
        ok_args = False
        ok_kwargs = False

        list_params = []
        keyword_params = set()

        sig = signature(func)
        for param in sig.parameters.values():
            if param.kind == param.VAR_POSITIONAL:
                ok_args = True
            if param.kind == param.VAR_KEYWORD:
                ok_kwargs = True

            if param.kind in [param.POSITIONAL_OR_KEYWORD]:
                list_params.append(param.name)
            if param.kind in [param.POSITIONAL_OR_KEYWORD, param.KEYWORD_ONLY]:
                keyword_params.add(param.name)

        def get_default_value(param_name):
            original = sig.parameters[param_name]
            no_default = original.default is original.empty
            return default if original.default is original.empty else original.default

        @wraps(func)
        def wrap(*args, **kwargs):
            if not ok_args:
                args = args[:len(list_params)]

            if not ok_kwargs:
                temp = {k: v for k, v in kwargs.items() if k in keyword_params}
                kwargs = temp

            if len(args) < len(list_params):
                not_set_list_params = list_params[len(args):]
                for param in not_set_list_params:
                    if param in kwargs:
                        continue

                    kwargs[param] = get_default_value(param)

            not_set_keyword_params = keyword_params - set(list_params) - set(kwargs.keys())
            for param in not_set_keyword_params:
                kwargs[param] = get_default_value(param)

            return func(*args, **kwargs)
        return wrap
    return deco

>>> @safe_param('is default')
... def sample3(a, b, *args, c, **kwargs):
...     print('a', a)
...     print('b', b)
...     print('c', c)
...     print('args', args)
...     print('kwargs', kwargs)
... 
>>> sample3(1,2,3,4,5,6,7, d=10)
a 1
b 2
c is default
args (3, 4, 5, 6, 7)
kwargs {'d': 10}
{{< / highlight >}}

잘 작동하는 것을 확인할 수 있습니다. 이로써 원하는 `inspect.signature`로 함수의 파라미터를 읽어 원하는 방식으로 처리해 볼 수 있었습니다.

위의 코드들은 [Github 저장소](https://github.com/vazrupe/python-inspect-signature-sample)에서 확인할 수 있고, [MIT License](https://en.wikipedia.org/wiki/MIT_License)로 사용 가능합니다.