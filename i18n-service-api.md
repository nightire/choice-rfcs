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
  ],
  "meta": {
  }
}
```

`meta` 用来存储以后需要的元数据，比如分页信息。

### 获取一个 entry

```
GET /v1/projects/:pid/entries/:entry_key
GET /v1/projects/:pid/entries/:entry_key?lang=zh_cn
```

### 创建新的 entry

根据 key 创建 entry，如果 entry 已存在就返回 entry 信息，没有就创建一个。默认返回 entry 的所有语言版本。这个 API 也可以用于写 UI 时自动创建词条。

```
POST /v1/projects/:pid/entries/:entry_key
```

以下例子为 `common.button.ok` 创建 entry 。

```
POST /v1/projects/1/entries/common.button.ok
```

返回：201

```json
{
  "key": "common.button.ok",
  "values": {
    "en_us": "OK",
    "zh_cn": "确定",
    "zh_tw": "確定",
  }
}
```

### 修改 entry

修改 entry 的翻译文本，可以修改一个或多个，没有指定的语言版本不受影响。始终返回 entry 的所有语言版本。

```
PUT /v1/projects/:pid/entries/:entry_key
```

以下例子为修改 entry 的 en_us 翻译文本。

```
PUT /v1/projects/1/entries/common.button.ok

{
  "en_us": "OK"
}
```

## Entry output

这个 API 专门给外部 app 使用，获取一个 project 下面的指定语言的所有 entry ，以 key value 形式输出，无分页。

```
GET /v1/projects/:pid/entry_output/en_us
```

返回

```json
{
  "common.button.ok": "OK",
  "common.button.cancel": "Cancel",
}
```
