URL

```
http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=%2C0%2C${username}%40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh
```

CURL(CMD)

```
curl ^"http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=^%^2C0^%^2C${username}^%^40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh^" ^
  -H "Accept: */*" ^
  -H "Accept-Language: zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5" ^
  -H "Connection: keep-alive" ^
  -H "Referer: http://10.63.63.63/" ^
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0" ^
  --insecure
```

CURL Bash

```
curl 'http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=%2C0%2C${username}%40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh' \
  -H 'Accept: */*' \
  -H 'Accept-Language: zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5' \
  -H 'Connection: keep-alive' \
  -H 'Referer: http://10.63.63.63/' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0' \
  --insecure
```

PowerShell

```
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.UserAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0"
Invoke-WebRequest -UseBasicParsing -Uri "http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=%2C0%2C${username}%40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh" `
-WebSession $session `
-Headers @{
"Accept"="*/*"
  "Accept-Encoding"="gzip, deflate"
  "Accept-Language"="zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5"
  "Referer"="http://10.63.63.63/"
}
```

Fetch

```
fetch("http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=%2C0%2C${username}%40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh", {
  "headers": {
    "accept": "*/*",
    "accept-language": "zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5"
  },
  "referrer": "http://10.63.63.63/",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": null,
  "method": "GET",
  "mode": "cors",
  "credentials": "omit"
});
```

Nodejs

```
fetch("http://10.63.63.63:801/eportal/portal/login?callback=dr1003&login_method=1&user_account=%2C0%2C${username}%40telecom&user_password=${password}&wlan_user_ip=${IP}&wlan_user_ipv6=&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.2.1&terminal_type=1&lang=zh-cn&v=10200&lang=zh", {
  "headers": {
    "accept": "*/*",
    "accept-language": "zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5",
    "Referer": "http://10.63.63.63/",
    "Referrer-Policy": "strict-origin-when-cross-origin"
  },
  "body": null,
  "method": "GET"
});
```



# logout

URL

```
http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214%40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh
```

CMD

```sh
curl ^"http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214^%^40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh^" ^
  -H "Accept: */*" ^
  -H "Accept-Language: zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5" ^
  -H "Connection: keep-alive" ^
  -H "Referer: http://10.63.63.63/" ^
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0" ^
  --insecure
```

bash

```sh
curl 'http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214%40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh' \
  -H 'Accept: */*' \
  -H 'Accept-Language: zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5' \
  -H 'Connection: keep-alive' \
  -H 'Referer: http://10.63.63.63/' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0' \
  --insecure
```

PowerShell

```
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.UserAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 Edg/128.0.0.0"
Invoke-WebRequest -UseBasicParsing -Uri "http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214%40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh" `
-WebSession $session `
-Headers @{
"Accept"="*/*"
  "Accept-Encoding"="gzip, deflate"
  "Accept-Language"="zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5"
  "Referer"="http://10.63.63.63/"
}
```

Fetch

```
fetch("http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214%40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh", {
  "headers": {
    "accept": "*/*",
    "accept-language": "zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5"
  },
  "referrer": "http://10.63.63.63/",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": null,
  "method": "GET",
  "mode": "cors",
  "credentials": "omit"
});
```

Nodejs

```
fetch("http://10.63.63.63:801/eportal/portal/mac/unbind?callback=dr1003&user_account=202232110214%40telecom&wlan_user_mac=000000000000&wlan_user_ip=171813309&jsVersion=4.2.1&v=576&lang=zh", {
  "headers": {
    "accept": "*/*",
    "accept-language": "zh-TW,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-HK;q=0.5",
    "Referer": "http://10.63.63.63/",
    "Referrer-Policy": "strict-origin-when-cross-origin"
  },
  "body": null,
  "method": "GET"
});
```