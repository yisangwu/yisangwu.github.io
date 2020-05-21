---
layout: post
title: Python版本Google广告admob服务端回调验证SSV
tags: google admob ssv
categories: pyhon
---
<style type="text/css">
    p{text-indent: 20px}
</style>

<p>
    Google的激励广告的Google回调的服务器端验证 (SSV) server side verifiy。Python版本的基于第三方包ecdsa开箱即用。
</p>

### Google公钥的地址：
<https://www.gstatic.com/admob/reward/verifier-keys.json>
> 注意： 
> >> 1. AdMob 密钥服务器提供的公钥会不定期轮换。为确保可以继续按预期验证 SSV 回调，请勿使公钥的缓存时间超过 24 小时。
> >> 2. Google 预计您的服务器会针对 SSV 回调返回 HTTP 200 OK 成功状态响应代码。如果您的服务器无法访问或未提供预期的响应，Google 将重新尝试发送 SSV 回调，每隔 1 秒发送最多 5 次。
> >> 3. 用回调参数中key_id 取对应公钥，进行签名验证。

### 安装ecdsa包：
```shell
$ pip install ecdsa
```

完整代码如下, python3版本：
```python
# codin=utf8
"""
google admob server side verify
python3 
use ecdsa
"""
import sys
import json
import urllib.parse
import urllib.request
import base64
import hashlib

from ecdsa.keys import VerifyingKey, BadSignatureError
from ecdsa.util import sigdecode_der

# AdMob密钥服务器
VERIFIER_KEYS_URL = 'https://www.gstatic.com/admob/reward/verifier-keys.json'
# request 的模拟浏览器信息
USER_AGENT = '''
Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
'''


def extract_verifier_keys():
    """
    从google的AdMob密钥服务器提取用于验证激励视频广告SSV回调的公钥列表。
    公钥列表以 JSON 格式
    """
    keys_dict = dict()
    response = urllib.request.urlopen(VERIFIER_KEYS_URL)
    if response.status != 200:
        return keys_dict

    json_keys = response.read().decode('utf-8')
    if not json_keys:
        return keys_dict

    data_keys = json.loads(json_keys)
    if not data_keys or 'keys' not in data_keys or not isinstance(data_keys, dict):
        return keys_dict

    for key_ids in data_keys['keys']:
        keys_dict[str(key_ids['keyId'])] = dict(
            pem=key_ids['pem'],
            base64=key_ids['base64']
        )
    return keys_dict


def parse_query_string(query_string: str):
    """
    解析query_string，取出signature， key_id，message
    :param query_string:
    :return:
    """
    query_dict = dict(
        signature='',
        key_id='',
        message=''
    )
    if not query_string or query_string.find('&') == -1:
        return query_dict

    query_string_dict = dict([x.split('=', 1) for x in query_string.split('&')])
    query_dict['signature'] = query_string_dict.get('signature').strip() or ''
    query_dict['message'] = query_string[:query_string.index('signature') - 1].strip() or ''
    query_dict['key_id'] = query_string_dict.get('key_id', '').strip() or ''

    return query_dict


def signature_verifier(ver_message: str, ver_signature: str, public_key_pem: str):
    """
    校验签名
    :param ver_message: 签名字符串
    :param ver_signature:  base64编码的签名
    :param public_key_pem: 公钥
    :return: boolean
    """
    if not all([ver_message, ver_signature, public_key_pem]):
        return False

    public_key = VerifyingKey.from_pem(public_key_pem)
    # 注意这里的 替换 和补位
    ver_signature = base64.urlsafe_b64decode(str(ver_signature) + '===')
    ver_message = bytes(ver_message, encoding="utf8")

    try:
        return public_key.verify(
            ver_signature,
            ver_message,
            hashfunc=hashlib.sha256,
            sigdecode=sigdecode_der,
        )
    except BadSignatureError as e:
        return False


if __name__ == '__main__':
    # Google 回调参数，get传参
    query_string = """abcd"""
    # 解析query_string
    query_dict = parse_query_string(query_string)
    signature = query_dict.get('signature').strip()  # 签名结果字符串
    message = query_dict.get('message').strip()  # 元数据
    key_id = query_dict.get('key_id').strip()  # 签名使用的公钥id 

    # 获取Google AdMob的公钥
    verifier_keys = extract_verifier_keys()
    if not verifier_keys or key_id not in verifier_keys:
        sys.exit('Fetch google admob keys failed!')

    # 根据google回调的key_id 取对应的公钥
    pem_keys = verifier_keys.get(str(key_id), None)
    if not pem_keys:
        sys.exit('Can not found public key by key_id!')

    ret_verifier = signature_verifier(message, signature, pem_keys['pem'])
    print(ret_verifier)
    # True
```
###　执行python脚本:
```shell
$ python depakin.py
True

```
True 校验成功。
