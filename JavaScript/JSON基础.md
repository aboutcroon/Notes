JSON.parse() https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse

JSON.stringfy https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify



如果打印的 res 出现了 [object, object]

那么可以使用 JSON.stringfy(res, null, 2)

来使之能打印出来

