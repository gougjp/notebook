Python调用Jenkins的API
=============================

这里举例获取Jenkins上某个Job的最后一次build状态

.. code::

    import requests
    import json

    api_url = 'http://<serverip>:<serverport>/job/<jobname>/lastBuild/api/json'
    headers = {'Authorization': "Basic anVucGluZy5nb3U6QGdqcDEyMzQ==",}
    querystring = {"pretty": "true"}

    response = requests.request("GET", api_url, headers=headers, params=querystring)
    if response.text:
        result = json.loads(response.text)
        status = result['result'].strip()
        print(status)
        if status == 'SUCCESS':
            exit(0)
        else:
            exit(1)
    exit(0)

其中headers中的"Basic anVucGluZy5nb3U6QGdqcDEyMzQ=="是HTTP基本认证, 
详见：https://blog.csdn.net/wochunyang/article/details/78675325; 它是通过BASE64编码后的字符：生成方式如下

.. code::

    import base64
    #编码
    base64.b64encode('junping.gou:@gjp1234') #冒号前面是用户名，冒号后面是密码
    #输出anVucGluZy5nb3U6QGdqcDEyMzQ==
    #解码
    base64.b64decode('anVucGluZy5nb3U6QGdqcDEyMzQ==')
