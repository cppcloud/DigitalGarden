---
{"dg-publish":true,"tags":["tools"],"permalink":"/Tools/自建的deepl翻译/","dgPassFrontmatter":true}
---


```bash
https://deepl.z23.cc/translate
```

### 使用 post 请求

```bash

curl --location 'https://deepl.z23.cc/translate' \
--header 'Content-Type: application/json' \
--data '{
    "text": "免费，无限量翻译 API",
    "source_lang": "zh",
    "target_lang": "en"
}'
```
