# Core

Source: [https://www.fastht.ml/docs/api/core.html](https://www.fastht.ml/docs/api/core.html)

---

1. [Source](https://www.fastht.ml/docs/ref/../api/core.html)
2. [Core](https://www.fastht.ml/docs/ref/../api/core.html)

# Core

This is the source code to fasthtml. You won’t need to read this unless you want to understand how things are built behind the scenes, or need full details of a particular API. The notebook is converted to the Python module [fasthtml/core.py](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py) using [nbdev](https://nbdev.fast.ai/).

## Imports and utils

```python
import time

from IPython import display
from enum import Enum
from pprint import pprint

from fastcore.test import *
from starlette.testclient import TestClient
from starlette.requests import Headers
from starlette.datastructures import UploadFile
```

We write source code *first*, and then tests come *after*. The tests serve as both a means to confirm that the code works and also serves as working examples. The first exported function, [parsed_date](https://www.fastht.ml/docs/api/core.html#parsed_date), is an example of this pattern.

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L45)

### parsed_date
> `parsed_date (s:str)`

*Convertsto a datetime*

```python
parsed_date('2pm')
```

```
datetime.datetime(2025, 7, 2, 14, 0)
```

```python
isinstance(date.fromtimestamp(0), date)
```

```
True
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L50)

### snake2hyphens
> `snake2hyphens (s:str)`

*Convertsfrom snake case to hyphenated and capitalised*

```python
snake2hyphens("snake_case")
```

```
'Snake-Case'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L67)

### HtmxHeaders
> `HtmxHeaders (boosted:str|None=None, current_url:str|None=None,
> history_restore_request:str|None=None, prompt:str|None=None,
> request:str|None=None, target:str|None=None,
> trigger_name:str|None=None, trigger:str|None=None)`

```python
def test_request(url: str='/', headers: dict={}, method: str='get') -> Request:
    scope = {
        'type': 'http',
        'method': method,
        'path': url,
        'headers': Headers(headers).raw,
        'query_string': b'',
        'scheme': 'http',
        'client': ('127.0.0.1', 8000),
        'server': ('127.0.0.1', 8000),
    }
    receive = lambda: {"body": b"", "more_body": False}
    return Request(scope, receive)
```

```python
h = test_request(headers=Headers({'HX-Request':'1'}))
_get_htmx(h.headers)
```

```
HtmxHeaders(boosted=None, current_url=None, history_restore_request=None, prompt=None, request='1', target=None, trigger_name=None, trigger=None)
```

## Request and response

```python
test_eq(_fix_anno(Union[str,None], 'a'), 'a')
test_eq(_fix_anno(float, 0.9), 0.9)
test_eq(_fix_anno(int, '1'), 1)
test_eq(_fix_anno(int, ['1','2']), 2)
test_eq(_fix_anno(list[int], ['1','2']), [1,2])
test_eq(_fix_anno(list[int], '1'), [1])
```

```python
d = dict(k=int, l=List[int])
test_eq(_form_arg('k', "1", d), 1)
test_eq(_form_arg('l', "1", d), [1])
test_eq(_form_arg('l', ["1","2"], d), [1,2])
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L106)

### HttpHeader
> `HttpHeader (k:str, v:str)`

```python
_to_htmx_header('trigger_after_settle')
```

```
'HX-Trigger-After-Settle'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L117)

### HtmxResponseHeaders
> `HtmxResponseHeaders (location=None, push_url=None, redirect=None,
> refresh=None, replace_url=None, reswap=None,
> retarget=None, reselect=None, trigger=None,
> trigger_after_settle=None, trigger_after_swap=None)`

*HTMX response headers*

```python
HtmxResponseHeaders(trigger_after_settle='hi')
```

```
HttpHeader(k='HX-Trigger-After-Settle', v='hi')
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L139)

### form2dict
> `form2dict (form:starlette.datastructures.FormData)`

*Convert starlette form data to a dict*

```python
d = [('a',1),('a',2),('b',0)]
fd = FormData(d)
res = form2dict(fd)
test_eq(res['a'], [1,2])
test_eq(res['b'], 0)
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L145)

### parse_form
> `parse_form (req:starlette.requests.Request)`

*Starlette errors on empty multipart forms, so this checks for that situation*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L168)

### JSONResponse
> `JSONResponse (content:Any, status_code:int=200,
> headers:collections.abc.Mapping[str,str]|None=None,
> media_type:str|None=None,
> background:starlette.background.BackgroundTask|None=None)`

*Same as starlette’s version, but auto-stringifies non serializable types*

```python
async def f(req):
    def _f(p:HttpHeader): ...
    p = first(_params(_f).values())
    result = await _from_body(req, p)
    return JSONResponse(result.dict)

client = TestClient(Starlette(routes=[Route('/', f, methods=['POST'])]))

d = dict(k='value1',v=['value2','value3'])
response = client.post('/', data=d)
print(response.json())
```

```
{'k': 'value1', 'v': 'value3'}
```

```python
async def f(req): return Response(str(req.query_params.getlist('x')))
client = TestClient(Starlette(routes=[Route('/', f, methods=['GET'])]))
client.get('/?x=1&x=2').text
```

```
"['1', '2']"
```

```python
def g(req, this:Starlette, a:str, b:HttpHeader): ...

async def f(req):
    a = await _wrap_req(req, _params(g))
    return Response(str(a))

client = TestClient(Starlette(routes=[Route('/', f, methods=['POST'])]))
response = client.post('/?a=1', data=d)
print(response.text)
```

```
[<starlette.requests.Request object>, <starlette.applications.Starlette object>, '1', HttpHeader(k='value1', v='value3')]
```

```python
def g(req, this:Starlette, a:str, b:HttpHeader): ...

async def f(req):
    a = await _wrap_req(req, _params(g))
    return Response(str(a))

client = TestClient(Starlette(routes=[Route('/', f, methods=['POST'])]))
response = client.post('/?a=1', data=d)
print(response.text)
```

```
[<starlette.requests.Request object>, <starlette.applications.Starlette object>, '1', HttpHeader(k='value1', v='value3')]
```

**Missing Request Params**

If a request param has a default value (e.g. `a:str=''`), the request is valid even if the user doesn’t include the param in their request.

```python
def g(req, this:Starlette, a:str=''): ...

async def f(req):
    a = await _wrap_req(req, _params(g))
    return Response(str(a))

client = TestClient(Starlette(routes=[Route('/', f, methods=['POST'])]))
response = client.post('/', json={}) # no param in request
print(response.text)
```

```
[<starlette.requests.Request object>, <starlette.applications.Starlette object>, '']
```

If we remove the default value and re-run the request, we should get the following error `Missing required field: a`.

```python
def g(req, this:Starlette, a:str): ...

async def f(req):
    a = await _wrap_req(req, _params(g))
    return Response(str(a))

client = TestClient(Starlette(routes=[Route('/', f, methods=['POST'])]))
response = client.post('/', json={}) # no param in request
print(response.text)
```

```
Missing required field: a
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L218)

### flat_xt
> `flat_xt (lst)`

*Flatten lists*

```python
x = ft('a',1)
test_eq(flat_xt([x, x, [x,x]]), (x,)*4)
test_eq(flat_xt(x), (x,))
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L228)

### Beforeware
> `Beforeware (f, skip=None)`

*Initialize self. See help(type(self)) for accurate signature.*

## Websockets / SSE

```python
def on_receive(self, msg:str): return f"Message text was: {msg}"
c = _ws_endp(on_receive)
cli = TestClient(Starlette(routes=[WebSocketRoute('/', _ws_endp(on_receive))]))
with cli.websocket_connect('/') as ws:
    ws.send_text('{"msg":"Hi!"}')
    data = ws.receive_text()
    assert data == 'Message text was: Hi!'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L290)

### EventStream
> `EventStream (s)`

*Create a text/event-stream response froms*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L295)

### signal_shutdown
> `signal_shutdown ()`

## Routing and application

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L306)

### uri
> `uri (_arg, **kwargs)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L310)

### decode_uri
> `decode_uri (s)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L321)

### StringConvertor.to_string
> `StringConvertor.to_string (value:str)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L329)

### HTTPConnection.url_path_for
> `HTTPConnection.url_path_for (name:str, **path_params)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L367)

### flat_tuple
> `flat_tuple (o)`

*Flatten lists*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L378)

### noop_body
> `noop_body (c, req)`

*Default Body wrap function which just returns the content*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L383)

### respond
> `respond (req, heads, bdy)`

*Default FT response creation function*

Render fragment if `HX-Request` header is *present* and `HX-History-Restore-Request` header is *absent.*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L392)

### is_full_page
> `is_full_page (req, resp)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L446)

### Redirect
> `Redirect (loc)`

*Use HTMX or Starlette RedirectResponse as required to redirect toloc*

The FastHTML `exts` param supports the following:

```python
print(' '.join(htmx_exts))
```

```
morph head-support preload class-tools loading-states multi-swap path-deps remove-me ws chunked-transfer
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L481)

### get_key
> `get_key (key=None, fname='.sesskey')`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L502)

### qp
> `qp (p:str, **kw)`

*Add parameters kw to path p*

[qp](https://www.fastht.ml/docs/api/core.html#qp) adds query parameters to route path strings

```python
vals = {'a':5, 'b':False, 'c':[1,2], 'd':'bar', 'e':None, 'ab':42}
```

```python
res = qp('/foo', **vals)
test_eq(res, '/foo?a=5&b=&c=1&c=2&d=bar&e=&ab=42')
```

[qp](https://www.fastht.ml/docs/api/core.html#qp) checks to see if each param should be sent as a query parameter or as part of the route, and encodes that properly.

```python
path = '/foo/{a}/{d}/{ab:int}'
res = qp(path, **vals)
test_eq(res, '/foo/5/bar/42?b=&c=1&c=2&e=')
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L514)

### def_hdrs
> `def_hdrs (htmx=True, surreal=True)`

*Default headers for a FastHTML app*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L536)

### FastHTML
> `FastHTML (debug=False, routes=None, middleware=None, title:str='FastHTML
> page', exception_handlers=None, on_startup=None,
> on_shutdown=None, lifespan=None, hdrs=None, ftrs=None,
> exts=None, before=None, after=None, surreal=True, htmx=True,
> default_hdrs=True, sess_cls=<class
> 'starlette.middleware.sessions.SessionMiddleware'>,
> secret_key=None, session_cookie='session_', max_age=31536000,
> sess_path='/', same_site='lax', sess_https_only=False,
> sess_domain=None, key_fname='.sesskey', body_wrap=<function
> noop_body>, htmlkw=None, nb_hdrs=False, canonical=True,
> **bodykw)`

*Creates an Starlette application.*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L573)

### FastHTML.add_route
> `FastHTML.add_route (route)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L618)

### FastHTML.ws
> `FastHTML.ws (path:str, conn=None, disconn=None, name=None,
> middleware=None)`

*Add a websocket route atpath*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L633)

### nested_name
> `nested_name (f)`

*Get name of function `f` using ’_’ to join nested function names*

```python
def f():
    def g(): ...
    return g
```

```python
func = f()
nested_name(func)
```

```
'f_g'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L654)

### FastHTML.route
> `FastHTML.route (path:str=None, methods=None, name=None,
> include_in_schema=True, body_wrap=None)`

*Add a route atpath*

```python
app = FastHTML()
@app.get
def foo(a:str, b:list[int]): ...

foo.to(a='bar', b=[1,2])
```

```
'/foo?a=bar&b=1&b=2'
```

```python
@app.get('/foo/{a}')
def foo(a:str, b:list[int]): ...

foo.to(a='bar', b=[1,2])
```

```
'/foo/bar?b=1&b=2'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L663)

### FastHTML.set_lifespan
> `FastHTML.set_lifespan (value)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L668)

### serve
> `serve (appname=None, app='app', host='0.0.0.0', port=None, reload=True,
> reload_includes:list[str]|str|None=None,
> reload_excludes:list[str]|str|None=None)`

*Run the app in an async server, with live reload set as the default.*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L691)

### Client
> `Client (app, url='http://testserver')`

*A simple httpx ASGI client that doesn’t requireasync*

```python
app = FastHTML(routes=[Route('/', lambda _: Response('test'))])
cli = Client(app)

cli.get('/').text
```

```
'test'
```

Note that you can also use Starlette’s `TestClient` instead of FastHTML’s [Client](https://www.fastht.ml/docs/api/core.html#client). They should be largely interchangable.

## FastHTML Tests

```python
def get_cli(app): return app,TestClient(app),app.route
```

```python
app,cli,rt = get_cli(FastHTML(secret_key='soopersecret'))
```

```python
app,cli,rt = get_cli(FastHTML(title="My Custom Title"))
@app.get
def foo(): return Div("Hello World")

print(app.routes)

response = cli.get('/foo')
assert '<title>My Custom Title</title>' in response.text

foo.to(param='value')
```

```
[Route(path='/foo', name='foo', methods=['GET', 'HEAD'])]
```

```
'/foo?param=value'
```

```python
app,cli,rt = get_cli(FastHTML())

@rt('/xt2')
def get(): return H1('bar')

txt = cli.get('/xt2').text
assert '<title>FastHTML page</title>' in txt and '<h1>bar</h1>' in txt and '<html>' in txt
```

```python
@rt("/hi")
def get(): return 'Hi there'

r = cli.get('/hi')
r.text
```

```
'Hi there'
```

```python
@rt("/hi")
def post(): return 'Postal'

cli.post('/hi').text
```

```
'Postal'
```

```python
@app.get("/hostie")
def show_host(req): return req.headers['host']

cli.get('/hostie').text
```

```
'testserver'
```

```python
@app.get("/setsess")
def set_sess(session):
   session['foo'] = 'bar'
   return 'ok'

@app.ws("/ws")
def ws(self, msg:str, ws:WebSocket, session): return f"Message text was: {msg} with session {session.get('foo')}, from client: {ws.client}"

cli.get('/setsess')
with cli.websocket_connect('/ws') as ws:
    ws.send_text('{"msg":"Hi!"}')
    data = ws.receive_text()
assert 'Message text was: Hi! with session bar' in data
print(data)
```

```
Message text was: Hi! with session bar, from client: Address(host='testclient', port=50000)
```

```python
@rt
def yoyo(): return 'a yoyo'

cli.post('/yoyo').text
```

```
'a yoyo'
```

```python
@app.get
def autopost(): return Html(Div('Text.', hx_post=yoyo()))
print(cli.get('/autopost').text)
```

```
<!doctype html>
 <html>
   <div hx-post="a yoyo">Text.</div>
 </html>
```

```python
@app.get
def autopost2(): return Html(Body(Div('Text.', cls='px-2', hx_post=show_host.to(a='b'))))
print(cli.get('/autopost2').text)
```

```
<!doctype html>
 <html>
   <body>
     <div class="px-2" hx-post="/hostie?a=b">Text.</div>
   </body>
 </html>
```

```python
@app.get
def autoget2(): return Html(Div('Text.', hx_get=show_host))
print(cli.get('/autoget2').text)
```

```
<!doctype html>
 <html>
   <div hx-get="/hostie">Text.</div>
 </html>
```

```python
@rt('/user/{nm}', name='gday')
def get(nm:str=''): return f"Good day to you, {nm}!"
cli.get('/user/Alexis').text
```

```
'Good day to you, Alexis!'
```

```python
@app.get
def autolink(): return Html(Div('Text.', link=uri('gday', nm='Alexis')))
print(cli.get('/autolink').text)
```

```
<!doctype html>
 <html>
   <div href="/user/Alexis">Text.</div>
 </html>
```

```python
@rt('/link')
def get(req): return f"{req.url_for('gday', nm='Alexis')}; {req.url_for('show_host')}"

cli.get('/link').text
```

```
'http://testserver/user/Alexis; http://testserver/hostie'
```

```python
@app.get("/background")
async def background_task(request):
    async def long_running_task():
        await asyncio.sleep(0.1)
        print("Background task completed!")
    return P("Task started"), BackgroundTask(long_running_task)

response = cli.get("/background")
```

```
Background task completed!
```

```python
test_eq(app.router.url_path_for('gday', nm='Jeremy'), '/user/Jeremy')
```

```python
hxhdr = {'headers':{'hx-request':"1"}}

@rt('/ft')
def get(): return Title('Foo'),H1('bar')

txt = cli.get('/ft').text
assert '<title>Foo</title>' in txt and '<h1>bar</h1>' in txt and '<html>' in txt

@rt('/xt2')
def get(): return H1('bar')

txt = cli.get('/xt2').text
assert '<title>FastHTML page</title>' in txt and '<h1>bar</h1>' in txt and '<html>' in txt

assert cli.get('/xt2', **hxhdr).text.strip() == '<h1>bar</h1>'

@rt('/xt3')
def get(): return Html(Head(Title('hi')), Body(P('there')))

txt = cli.get('/xt3').text
assert '<title>FastHTML page</title>' not in txt and '<title>hi</title>' in txt and '<p>there</p>' in txt
```

```python
@rt('/oops')
def get(nope): return nope
test_warns(lambda: cli.get('/oops?nope=1'))
```

```python
def test_r(cli, path, exp, meth='get', hx=False, **kwargs):
    if hx: kwargs['headers'] = {'hx-request':"1"}
    test_eq(getattr(cli, meth)(path, **kwargs).text, exp)

ModelName = str_enum('ModelName', "alexnet", "resnet", "lenet")
fake_db = [{"name": "Foo"}, {"name": "Bar"}]
```

```python
@rt('/html/{idx}')
async def get(idx:int): return Body(H4(f'Next is {idx+1}.'))
```

```python
@rt("/models/{nm}")
def get(nm:ModelName): return nm

@rt("/files/{path}")
async def get(path: Path): return path.with_suffix('.txt')

@rt("/items/")
def get(idx:int|None = 0): return fake_db[idx]

@rt("/idxl/")
def get(idx:list[int]): return str(idx)
```

```python
r = cli.get('/html/1', headers={'hx-request':"1"})
assert '<h4>Next is 2.</h4>' in r.text
test_r(cli, '/models/alexnet', 'alexnet')
test_r(cli, '/files/foo', 'foo.txt')
test_r(cli, '/items/?idx=1', '{"name":"Bar"}')
test_r(cli, '/items/', '{"name":"Foo"}')
assert cli.get('/items/?idx=g').text=='404 Not Found'
assert cli.get('/items/?idx=g').status_code == 404
test_r(cli, '/idxl/?idx=1&idx=2', '[1, 2]')
assert cli.get('/idxl/?idx=1&idx=g').status_code == 404
```

```python
app = FastHTML()
rt = app.route
cli = TestClient(app)
@app.route(r'/static/{path:path}.jpg')
def index(path:str): return f'got {path}'
@app.route(r'/static/{path:path}')
def foo(path:str, a:int): return f'also got {path},{a}'
cli.get('/static/sub/a.b.jpg').text
```

```
'got sub/a.b'
```

```python
cli.get('/static/sub/a.b?a=1').text
```

```
'also got sub/a.b,1'
```

```python
app.chk = 'foo'
```

```python
@app.get("/booly/")
def _(coming:bool=True): return 'Coming' if coming else 'Not coming'

@app.get("/datie/")
def _(d:parsed_date): return d

@app.get("/ua")
async def _(user_agent:str): return user_agent

@app.get("/hxtest")
def _(htmx): return htmx.request

@app.get("/hxtest2")
def _(foo:HtmxHeaders, req): return foo.request

@app.get("/app")
def _(app): return app.chk

@app.get("/app2")
def _(foo:FastHTML): return foo.chk,HttpHeader("mykey", "myval")

@app.get("/app3")
def _(foo:FastHTML): return HtmxResponseHeaders(location="http://example.org")

@app.get("/app4")
def _(foo:FastHTML): return Redirect("http://example.org")
```

```python
test_r(cli, '/booly/?coming=true', 'Coming')
test_r(cli, '/booly/?coming=no', 'Not coming')
date_str = "17th of May, 2024, 2p"
test_r(cli, f'/datie/?d={date_str}', '2024-05-17 14:00:00')
test_r(cli, '/ua', 'FastHTML', headers={'User-Agent':'FastHTML'})
test_r(cli, '/hxtest' , '1', headers={'HX-Request':'1'})
test_r(cli, '/hxtest2', '1', headers={'HX-Request':'1'})
test_r(cli, '/app' , 'foo')
```

```python
r = cli.get('/app2', **hxhdr)
test_eq(r.text, 'foo')
test_eq(r.headers['mykey'], 'myval')
```

```python
r = cli.get('/app3')
test_eq(r.headers['HX-Location'], 'http://example.org')
```

```python
r = cli.get('/app4', follow_redirects=False)
test_eq(r.status_code, 303)
```

```python
r = cli.get('/app4', headers={'HX-Request':'1'})
test_eq(r.headers['HX-Redirect'], 'http://example.org')
```

```python
@rt
def meta():
    return ((Title('hi'),H1('hi')),
        (Meta(property='image'), Meta(property='site_name'))
    )

t = cli.post('/meta').text
assert re.search(r'<body>\s*<h1>hi</h1>\s*</body>', t)
assert '<meta' in t
```

```python
@app.post('/profile/me')
def profile_update(username: str): return username

test_r(cli, '/profile/me', 'Alexis', 'post', data={'username' : 'Alexis'})
test_r(cli, '/profile/me', 'Missing required field: username', 'post', data={})
```

```python
# Example post request with parameter that has a default value
@app.post('/pet/dog')
def pet_dog(dogname: str = None): return dogname

# Working post request with optional parameter
test_r(cli, '/pet/dog', '', 'post', data={})
```

```python
@dataclass
class Bodie: a:int;b:str

@rt("/bodie/{nm}")
def post(nm:str, data:Bodie):
    res = asdict(data)
    res['nm'] = nm
    return res

@app.post("/bodied/")
def bodied(data:dict): return data

nt = namedtuple('Bodient', ['a','b'])

@app.post("/bodient/")
def bodient(data:nt): return asdict(data)

class BodieTD(TypedDict): a:int;b:str='foo'

@app.post("/bodietd/")
def bodient(data:BodieTD): return data

class Bodie2:
    a:int|None; b:str
    def init(self, a, b='foo'): store_attr()

@rt("/bodie2/", methods=['get','post'])
def bodie(d:Bodie2): return f"a: {d.a}; b: {d.b}"
```

```python
from fasthtml.xtend import Titled
```

```python
d = dict(a=1, b='foo')

test_r(cli, '/bodie/me', '{"a":1,"b":"foo","nm":"me"}', 'post', data=dict(a=1, b='foo', nm='me'))
test_r(cli, '/bodied/', '{"a":"1","b":"foo"}', 'post', data=d)
test_r(cli, '/bodie2/', 'a: 1; b: foo', 'post', data={'a':1})
test_r(cli, '/bodie2/?a=1&b=foo&nm=me', 'a: 1; b: foo')
test_r(cli, '/bodient/', '{"a":"1","b":"foo"}', 'post', data=d)
test_r(cli, '/bodietd/', '{"a":1,"b":"foo"}', 'post', data=d)
```

```python
# Testing POST with Content-Type: application/json
@app.post("/")
def index(it: Bodie): return Titled("It worked!", P(f"{it.a}, {it.b}"))

s = json.dumps({"b": "Lorem", "a": 15})
response = cli.post('/', headers={"Content-Type": "application/json"}, data=s).text
assert "<title>It worked!</title>" in response and "<p>15, Lorem</p>" in response
```

```python
# Testing POST with Content-Type: application/json
@app.post("/bodytext")
def index(body): return body

response = cli.post('/bodytext', headers={"Content-Type": "application/json"}, data=s).text
test_eq(response, '{"b": "Lorem", "a": 15}')
```

```python
files = [ ('files', ('file1.txt', b'content1')),
         ('files', ('file2.txt', b'content2')) ]
```

```python
@rt("/uploads")
async def post(files:list[UploadFile]):
    return ','.join([(await file.read()).decode() for file in files])

res = cli.post('/uploads', files=files)
print(res.status_code)
print(res.text)
```

```
200
content1,content2
```

```python
res = cli.post('/uploads', files=[files[0]])
print(res.status_code)
print(res.text)
```

```
200
content1
```

```python
@rt("/setsess")
def get(sess, foo:str=''):
    now = datetime.now()
    sess['auth'] = str(now)
    return f'Set to {now}'

@rt("/getsess")
def get(sess): return f'Session time: {sess["auth"]}'

print(cli.get('/setsess').text)
time.sleep(0.01)

cli.get('/getsess').text
```

```
Set to 2025-05-29 08:31:48.235262
```

```
'Session time: 2025-05-29 08:31:48.235262'
```

```python
@rt("/sess-first")
def post(sess, name: str):
    sess["name"] = name
    return str(sess)

cli.post('/sess-first', data={'name': 2})

@rt("/getsess-all")
def get(sess): return sess['name']

test_eq(cli.get('/getsess-all').text, '2')
```

```python
@rt("/upload")
async def post(uf:UploadFile): return (await uf.read()).decode()

with open('../../CHANGELOG.md', 'rb') as f:
    print(cli.post('/upload', files={'uf':f}, data={'msg':'Hello'}).text[:15])
```

```
# Release notes
```

```python
@rt("/form-submit/{list_id}")
def options(list_id: str):
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST',
        'Access-Control-Allow-Headers': '*',
    }
    return Response(status_code=200, headers=headers)
```

```python
h = cli.options('/form-submit/2').headers
test_eq(h['Access-Control-Allow-Methods'], 'POST')
```

```python
from fasthtml.authmw import user_pwd_auth
```

```python
def _not_found(req, exc): return Div('nope')

app,cli,rt = get_cli(FastHTML(exception_handlers={404:_not_found}))

txt = cli.get('/').text
assert '<div>nope</div>' in txt
assert '<!doctype html>' in txt
```

```python
app,cli,rt = get_cli(FastHTML())

@rt("/{name}/{age}")
def get(name: str, age: int):
    return Titled(f"Hello {name.title()}, age {age}")

assert '<title>Hello Uma, age 5</title>' in cli.get('/uma/5').text
assert '404 Not Found' in cli.get('/uma/five').text
```

```python
auth = user_pwd_auth(testuser='spycraft')
app,cli,rt = get_cli(FastHTML(middleware=[auth]))

@rt("/locked")
def get(auth): return 'Hello, ' + auth

test_eq(cli.get('/locked').text, 'not authenticated')
test_eq(cli.get('/locked', auth=("testuser","spycraft")).text, 'Hello, testuser')
```

```python
auth = user_pwd_auth(testuser='spycraft')
app,cli,rt = get_cli(FastHTML(middleware=[auth]))

@rt("/locked")
def get(auth): return 'Hello, ' + auth

test_eq(cli.get('/locked').text, 'not authenticated')
test_eq(cli.get('/locked', auth=("testuser","spycraft")).text, 'Hello, testuser')
```

## APIRouter

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L703)

### RouteFuncs
> `RouteFuncs ()`

*Initialize self. See help(type(self)) for accurate signature.*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L713)

### APIRouter
> `APIRouter (prefix:str|None=None, body_wrap=<function noop_body>)`

*Add routes to an app*

```python
ar = APIRouter()
```

```python
@ar("/hi")
def get(): return 'Hi there'
@ar("/hi")
def post(): return 'Postal'
@ar
def ho(): return 'Ho ho'
@ar("/hostie")
def show_host(req): return req.headers['host']
@ar
def yoyo(): return 'a yoyo'
@ar
def index(): return "home page"

@ar.ws("/ws")
def ws(self, msg:str): return f"Message text was: {msg}"
```

```python
app,cli,_ = get_cli(FastHTML())
ar.to_app(app)
```

```python
assert str(yoyo) == '/yoyo'
# ensure route functions are properly discoverable on `APIRouter` and `APIRouter.rt_funcs`
assert ar.prefix == ''
assert str(ar.rt_funcs.index) == '/'
assert str(ar.index) == '/'
with ExceptionExpected(): ar.blah()
with ExceptionExpected(): ar.rt_funcs.blah()
# ensure any route functions named using an HTTPMethod are not discoverable via `rt_funcs`
assert "get" not in ar.rt_funcs._funcs.keys()
```

```python
test_eq(cli.get('/hi').text, 'Hi there')
test_eq(cli.post('/hi').text, 'Postal')
test_eq(cli.get('/hostie').text, 'testserver')
test_eq(cli.post('/yoyo').text, 'a yoyo')

test_eq(cli.get('/ho').text, 'Ho ho')
test_eq(cli.post('/ho').text, 'Ho ho')
```

```python
with cli.websocket_connect('/ws') as ws:
    ws.send_text('{"msg":"Hi!"}')
    data = ws.receive_text()
    assert data == 'Message text was: Hi!'
```

```python
ar2 = APIRouter("/products")
```

```python
@ar2("/hi")
def get(): return 'Hi there'
@ar2("/hi")
def post(): return 'Postal'
@ar2
def ho(): return 'Ho ho'
@ar2("/hostie")
def show_host(req): return req.headers['host']
@ar2
def yoyo(): return 'a yoyo'
@ar2
def index(): return "home page"

@ar2.ws("/ws")
def ws(self, msg:str): return f"Message text was: {msg}"
```

```python
app,cli,_ = get_cli(FastHTML())
ar2.to_app(app)
```

```python
assert str(yoyo) == '/products/yoyo'
assert ar2.prefix == '/products'
assert str(ar2.rt_funcs.index) == '/products/'
assert str(ar2.index) == '/products/'
assert str(ar.index) == '/'
with ExceptionExpected(): ar2.blah()
with ExceptionExpected(): ar2.rt_funcs.blah()
assert "get" not in ar2.rt_funcs._funcs.keys()
```

```python
test_eq(cli.get('/products/hi').text, 'Hi there')
test_eq(cli.post('/products/hi').text, 'Postal')
test_eq(cli.get('/products/hostie').text, 'testserver')
test_eq(cli.post('/products/yoyo').text, 'a yoyo')

test_eq(cli.get('/products/ho').text, 'Ho ho')
test_eq(cli.post('/products/ho').text, 'Ho ho')
```

```python
with cli.websocket_connect('/products/ws') as ws:
    ws.send_text('{"msg":"Hi!"}')
    data = ws.receive_text()
    assert data == 'Message text was: Hi!'
```

```python
@ar.get
def hi2(): return 'Hi there'
@ar.get("/hi3")
def _(): return 'Hi there'
@ar.post("/post2")
def _(): return 'Postal'

@ar2.get
def hi2(): return 'Hi there'
@ar2.get("/hi3")
def _(): return 'Hi there'
@ar2.post("/post2")
def _(): return 'Postal'
```

## Extras

```python
app,cli,rt = get_cli(FastHTML(secret_key='soopersecret'))
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L756)

### cookie
> `cookie (key:str, value='', max_age=None, expires=None, path='/',
> domain=None, secure=False, httponly=False, samesite='lax')`

*Create a ‘set-cookie’HttpHeader*

```python
@rt("/setcookie")
def get(req): return cookie('now', datetime.now())

@rt("/getcookie")
def get(now:parsed_date): return f'Cookie was set at time {now.time()}'

print(cli.get('/setcookie').text)
time.sleep(0.01)
cli.get('/getcookie').text
```

```
<!doctype html>
 <html>
   <head>
     <title>FastHTML page</title>
     <link rel="canonical" href="http://testserver/setcookie">
     <meta charset="utf-8">
     <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<script src="https://cdn.jsdelivr.net/npm/[email protected]/dist/htmx.min.js"></script><script src="https://cdn.jsdelivr.net/gh/answerdotai/[email protected]/fasthtml.js"></script><script src="https://cdn.jsdelivr.net/gh/answerdotai/surreal@main/surreal.js"></script><script src="https://cdn.jsdelivr.net/gh/gnat/css-scope-inline@main/script.js"></script><script>
    function sendmsg() {
        window.parent.postMessage({height: document.documentElement.offsetHeight}, '*');
    }
    window.onload = function() {
        sendmsg();
        document.body.addEventListener('htmx:afterSettle',    sendmsg);
        document.body.addEventListener('htmx:wsAfterMessage', sendmsg);
    };</script>   </head>
   <body></body>
 </html>
```

```
'Cookie was set at time 08:31:49.013668'
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L774)

### reg_re_param
> `reg_re_param (m, s)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L785)

### FastHTML.static_route_exts
> `FastHTML.static_route_exts (prefix='/', static_path='.', exts='static')`

*Add a static route at URL pathprefixwith files fromstatic_pathandextsdefined byreg_re_param()*

```python
reg_re_param("imgext", "ico|gif|jpg|jpeg|webm|pdf")

@rt(r'/static/{path:path}{fn}.{ext:imgext}')
def get(fn:str, path:str, ext:str): return f"Getting {fn}.{ext} from /{path}"

test_r(cli, '/static/foo/jph.me.ico', 'Getting jph.me.ico from /foo/')
```

```python
app.static_route_exts()
assert 'These are the source notebooks for FastHTML' in cli.get('/README.txt').text
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L792)

### FastHTML.static_route
> `FastHTML.static_route (ext='', prefix='/', static_path='.')`

*Add a static route at URL pathprefixwith files fromstatic_pathand singleext(including the ‘.’)*

```python
app.static_route('.md', static_path='../..')
assert 'THIS FILE WAS AUTOGENERATED' in cli.get('/README.md').text
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L798)

### MiddlewareBase
> `MiddlewareBase ()`

*Initialize self. See help(type(self)) for accurate signature.*

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L806)

### FtResponse
> `FtResponse (content, status_code:int=200, headers=None, cls=<class
> 'starlette.responses.HTMLResponse'>,
> media_type:str|None=None,
> background:starlette.background.BackgroundTask|None=None)`

*Wrap an FT response with any StarletteResponse*

```python
@rt('/ftr')
def get():
    cts = Title('Foo'),H1('bar')
    return FtResponse(cts, status_code=201, headers={'Location':'/foo/1'})

r = cli.get('/ftr')

test_eq(r.status_code, 201)
test_eq(r.headers['location'], '/foo/1')
txt = r.text
assert '<title>Foo</title>' in txt and '<h1>bar</h1>' in txt and '<html>' in txt
```

Test on a single background task:

```python
def my_slow_task():
    print('Starting slow task')    
    time.sleep(0.001)
    print('Finished slow task')        

@rt('/background')
def get():
    return P('BG Task'), BackgroundTask(my_slow_task)

r = cli.get('/background')

test_eq(r.status_code, 200)
```

```
Starting slow task
Finished slow task
```

Test multiple background tasks:

```python
def increment(amount):
    amount = amount/1000
    print(f'Sleeping for {amount}s')    
    time.sleep(amount)
    print(f'Slept for {amount}s')
```

```python
@rt
def backgrounds():
    tasks = BackgroundTasks()
    for i in range(3): tasks.add_task(increment, i)
    return P('BG Tasks'), tasks

r = cli.get('/backgrounds')
test_eq(r.status_code, 200)
```

```
Sleeping for 0.0s
Slept for 0.0s
Sleeping for 0.001s
Slept for 0.001s
Sleeping for 0.002s
Slept for 0.002s
```

```python
@rt
def backgrounds2():
    tasks = [BackgroundTask(increment,i) for i in range(3)]
    return P('BG Tasks'), *tasks

r = cli.get('/backgrounds2')
test_eq(r.status_code, 200)
```

```
Sleeping for 0.0s
Slept for 0.0s
Sleeping for 0.001s
Slept for 0.001s
Sleeping for 0.002s
Slept for 0.002s
```

```python
@rt
def backgrounds3():
    tasks = [BackgroundTask(increment,i) for i in range(3)]
    return {'status':'done'}, *tasks

r = cli.get('/backgrounds3')
test_eq(r.status_code, 200)
r.json()
```

```
Sleeping for 0.0s
Slept for 0.0s
Sleeping for 0.001s
Slept for 0.001s
Sleeping for 0.002s
Slept for 0.002s
```

```
{'status': 'done'}
```

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L821)

### unqid
> `unqid (seeded=False)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L834)

### FastHTML.setup_ws
> `FastHTML.setup_ws (app:main.FastHTML, f=<function noop>)`

[source](https://github.com/AnswerDotAI/fasthtml/blob/main/fasthtml/core.py#L848)

### FastHTML.devtools_json
> `FastHTML.devtools_json (path=None, uuid=None)`