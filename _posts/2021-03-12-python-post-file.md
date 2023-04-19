---
layout: post
title: Python使用form表单上传excel文件
tags: python excel
categories: python  
---

写一个测试脚本，测试文件上传并解析的接口逻辑与执行效率，大概的代码如下。 

```python
# coding=utf8

import json
import request

request_URL = "https://api.com/api/uploadExcel"

headers={
	"User-Agent":"Mozilla/5.0(Macintosh;U;IntelMacOSX10_6_8;en-us)AppleWebKit/534.50(KHTML,likeGecko)Version/5.1Safari/534.50",
	"cookie":"testUser",
}

file_path = "file_xx.xlsx"
files = {
	"file":(
		file_path,
		open(file_path, "rb"),
		"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
		)
}

result = requests.post(url, headers=headers, data=dict(a=1), files=files)
print(result.json())

```