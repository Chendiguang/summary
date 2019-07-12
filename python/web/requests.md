# [requests 模块](https://2.python-requests.org//zh_CN/latest/user/quickstart.html)

* post 方法</br>
常见的几种提交数据方式
    1. multipart/form-data用于上传文件

        ```py
        url = 'http://example.com'
        files = {'file': open(filename, 'wb')}
        requests.post(url=url, files=files)
        ```

    2. application/json

        ```py
        headers = {'Content-Type':'application/json'}
        params = {}
        # 可以传递json, data
        requests.post(url=url, json=post_dict, headers=headers)
        ```

    3. application/x-www-form-urlencoded

推荐用法，尽量考虑一些异常情况

```py
    try:
        ret = requests.post(url=url, files=files, headers=headers)
    except requests.exceptions.ReadTimeout:
       err_msg = "timeout"
    except requests.exceptions.HTTPError as exc:
        err_msg = 'HTTP {res.status_code} - {res.reason}'
        err_msg = err_msg.format(res=exc.response)
    except requests.exceptions.ConnectionError as exc:
        err_msg = 'Connection error'
    else:
        err_msg = ''
```
