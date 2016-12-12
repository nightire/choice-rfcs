# I18n service API 设计

## Project

标准 REST 。删除的 API 需要注意检查一下 entry 是否为空。

```
GET /v1/projects
GET /v1/projects/:pid
POST /v1/projects
PUT /v1/projects/:pid
DELETE /v1/projects/:pid
```

## Entry

### 获取所有 entry

只能获取某个 project 下面的 entry 。获取单个 entry 以 key 为标识符。换句话说，project id + entry key 构成了 entry 的唯一标识符。

获取所有 entry ，包含所有语言版本。
```
GET /v1/projects/:pid/entries
```

获取所有 entry ，限定中文和英文。

```
GET /v1/projects/:pid/entries?lang=zh_cn,en_us
```

返回

```json
{
  "data": [
    {
      "key": "common.button.ok",
      "values": {
        "en_us": "OK",
        "zh_cn": "确定",
        "zh_tw": "確定",
      }
    }
  ]
}
```

如果以后需要其他元数据，比如分页信息，可以在顶层加上 `meta` key ，value 限定为 object 。

### 获取一个 entry

```
GET /v1/projects/:pid/entries/:entry_key
GET /v1/projects/:pid/entries/:entry_key?lang=zh_cn
```

### 创建或修改 entry

根据 key 修改 entry，如果没有对应的 entry 就自动创建一个。客户端可选择上传对应语言版本的翻译，但不是必须的。如果不传任何数据，则值不作任何改变，如果触发自动创建，每项语言的值均为 `null` 。

返回的 status code 方面，自动创建 entry 返回 201，否则返回 200 。

这个 API 可用于 UI 自动创建词条的 helper ，也可以用于 i18n service 内部的管理界面。

```
POST /v1/projects/:pid/entries/:entry_key

{
  "en_us": "OK",
  "zh_cn": "确定"
}
```

以下例子中为 `common.button.ok` 创建 entry 。

```
POST /v1/projects/1/entries/common.button.ok
```

返回：201

```json
{
  "key": "common.button.ok",
  "values": {
    "en_us": null,
    "zh_cn": null,
    "zh_tw": null,
  }
}
```

以下例子中为 `common.button.ok` 修改 en_us 和 zh_cn 的翻译文本。

返回：200

```
POST /v1/projects/1/entries/common.button.ok

{
  "en_us": "OK",
  "zh_cn": "确定"
}
```

返回：201

```json
{
  "key": "common.button.ok",
  "values": {
    "en_us": "OK",
    "zh_cn": "确定",
    "zh_tw": null,
  }
}
```

以下例子中把 `common.button.ok` 的 zh_cn 翻译文本设置为 `null`。

返回：200

```
POST /v1/projects/1/entries/common.button.ok

{
  "zh_cn": null
}
```

返回：200

```json
{
  "key": "common.button.ok",
  "values": {
    "en_us": "OK",
    "zh_cn": null,
    "zh_tw": null,
  }
}
```

## Entry output

这个 API 专门给外部 app 使用，获取一个 project 下面的指定语言的所有 entry ，以 nested key 的形式输出，无分页。

```
GET /v1/projects/:pid/entry_output/en_us
```

返回

```json
{
  "common": {
    "button": {
      "ok": "OK",
      "cancel": "Cancel"
    }
  }
}
```
