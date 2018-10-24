---
title: "Flask를 reverse proxy 서버로 사용하기"
date: 2017-01-24T00:21:00+09:00
tags : ["python", "flask"]
categories : ["develop"]

aliasese:
    - /develop/2017/01/24/flask-reverse-proxy.html
---

 지금 만들고 있는 개인 프로젝트는 [flask](http://flask.pocoo.org/)와 [angular](https://angular.io/)를 사용한 SPA 기반 프로젝트입니다.

 프로젝트를 만들면서 SPA의 ajax 요청을 처리할 데이터가 필요했습니다. angular에서 [service](https://angular.io/docs/ts/latest/tutorial/toh-pt4.html)라는 기능으로 샘플 데이터를 넣어 테스트해볼 수 있지만, flask에서 전송되는 데이터를 직접 읽고 싶어서 proxy로 사용해 둘을 같이 동작시킬 방법이 없나 찾아보았고(물론 service로 구현하는게 테스트에 좋습니다), stackoverflow에서 [requests](http://docs.python-requests.org/en/master/)를 이용해 구현하는 [방법](http://stackoverflow.com/a/36601467/6564250)을 찾을 수 있었습니다.
 
 제가 적용한 코드는 아래와 같습니다. stackoverflow와는 달리 replace로 포트만 변경합니다. flask의 기본 포트인 5000을 `ng serve`의 기본 포트인 4200으로 바꿔서 요청을 전송해줍니다.

{{< highlight python >}}
import requests
from flask import request, Response

@web.route('/', defaults={'path': 'index.html'})
@web.route('/<path:path>')
def spa_apps(path):
    resp = requests.request(
        method=request.method,
        url=request.url.replace(':5000', ':4200'),
        headers={key: value for (key, value) in request.headers if key != 'Host'},
        data=request.get_data(),
        cookies=request.cookies,
        allow_redirects=False)
    excluded_headers = ['content-encoding', 'content-length', 'transfer-encoding', 'connection']
    headers = [(name, value) for (name, value) in resp.raw.headers.items()
               if name.lower() not in excluded_headers]
    response = Response(resp.content, resp.status_code, headers)
    return response
{{< / highlight >}}