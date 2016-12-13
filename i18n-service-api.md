# I18n service API 设计


## Project

Project 是 entries 的容器。每个 entry 必须属于一个 project 。project 有两种：

- Global project ，系统默认创建的 project ，仅有一个，用于存放以 `global` 为顶层命名空间的 entries ，比如 `global.a.b.c` 。
- 用户创建的 project ，每个应用对应一个 project ，entries 各自独立，但都通过 global folder 共享 global project 下面的 entries 。

Project 的主要属性有 `id` 和 `name` 。`id` 是 project 的唯一标识符，`name` 是用于显示的名字。

注解：之前设计中有考虑把 `name` 作为用户设定的 project 唯一标识符。我觉得这种方式不如自增长 id 或者 uuid 。原因是用户设定的标识符容易被重用。比如某个叫 `editor` 的 project 被删除了，然后又新建了一个同名 project 。两个 project 其实并不相同，但拿 `name` 作为唯一标识符可能导致一些验证方面的漏洞，比如在采用 OAuth 2.0 这种只认资源权限不认用户或组织的校验方式中，以上情况容易让人突然可以访问其他用户的 project 。

### 获取所有 projects

```
GET /v1/projects
```

返回 200 ，response body 为 project 列表。

```json
{
  "data": [
    {
      "id": 1,
      "name": "Q"
    },
    {
      "id": 2,
      "name": "Editor"
    }
  ]
}
```

### 获取单个 project

`pid` 是 project id ，设计时可以为自增长的 number 或者 uuid 。

```
GET /v1/projects/:pid
```

返回 200 ，response body 为 project 。

### 获取 global project

简化 global project 的获取。如果 global project 的 id 为 5 ，以下等同于 `GET /v1/projects/5` 。

```
GET /v1/projects/global
```

返回 200 ，response body 为 project 。

### 创建 project

```
POST /v1/projects

{
  "name": "Design"
}
```

返回 201 ，response body 为 project 。

### 更新 project

```
PUT /v1/projects/:pid

{
  "name": "Design"
}
```

返回 200 ，response body 为 project 。

### 删除 project

```
DELETE /v1/projects/:pid
```

返回 204 ，没有 response body 。


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

### 创建或更新 entry

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

## Folder

Folder 从属于一个 project ，用于为 entries 分组，每个 folder 用 `filter` 属性指定一种 entries 的筛选规则。目前有三种：

1. `global` ，筛选 global entries 。
1. `all` ，筛选 project 下面的所有 entries ，不包括 global entries 。
1. `selection` ，根据 folder 存储的 entry keys 筛选指定的 entries 。

global 和 all 两个 folders 均为自动创建，无法修改和删除。global project 包含一个 global folder ，且不能创建任何其他 folders 。其他 project 创建时会自动生成 global 和 all 两个 folders ，用户可以创建 selection folders 。

### 获取一个 project 下面的 folders

```
GET /v1/projects/:pid/folders
```

返回 200

```json
{
  "data": [
    {
      "id": 1,
      "name": "Global",
      "filter": "global"
    },
    {
      "id": 2,
      "name": "All",
      "filter": "all"
    },
    {
      "id": 3,
      "name": "Dashboard",
      "filter": "selection"
    },
  ]
}
```

### 获取 folder 下面的 entries

```
GET /v1/folders/:fid/entries
```

返回 200 ，response body 为 entry 列表。

### 创建 folder

创建的 folder 的 filter 均为 `selection` ，并且 `name` 不能跟同一 project 下的其他 folder 重名。

```
POST /v1/folders

{
  "project_id": 1,
  "name": "Common",
  "entry_keys": [
    "common.button.ok",
    "common.button.cancel"
  ]
}
```

返回 201 ，response body 为 folder 。

### 更新 folder

可以更新 folder 的 `name` 和 `entry_keys` ，所有指定的属性均为替换而不是增量累加，没有指定的属性保持不变。

```
PUT /v1/folders/:fid

{
  "name": "Common",
  "entry_keys": [
    "common.button.ok"
  ]
}
```

返回 200 ，response body 为 folder 。

### 删除 folder

删除 folder 。只能删除 selection folder 。

```
DELETE /v1/folders/:fid
```

返回 204 ，没有 response body 。


## 错误的返回值

所有 API 发生错误时都遵循一下规范。

错误的状态码有以下几种：

- 当授权信息不正确时（比如 token 不正确，过期），返回 401 。
- 当授权正确但没有权限访问某资源时（比如访问其他组织的 project），返回 403 。
- 当指定的资源不存在时，返回 404 。
- 当 API 请求遇到错误时（比如参数不合法，校验错误），返回 422 。

错误的 response body 有两种。

当需要返回单条错误信息时，用以下格式。大多数情况都是这种格式。

```json
{
  "error": {
    "message": "Some error messages"
  }
}
```

当需要返回多条错误信息时，用以下格式。这种情况多发生在 422 校验错误。下面是创建 folder API 校验错误的情况。

```json
{
  "errors": {
    "name": "Name has already been taken",
    "entry_keys": "entries can't be blank"
  }
}
```


## 问题

- API 规范方面，要不要考虑 JSON API ？这样前端也许可以直接用 Ember Data 。后端或许要调研一下合适的方案。
- 当 response body 需要返回 folder 资源的情况下，folder 是否需要包含 entry_keys ？
- 如何更新 folder 的指定 entries ？比如添加或者删除某几个 entries 。是否需要这些功能？
- 考虑到 i18n 管理界面可能会有 "在某个 folder 下创建 entry" 的情况，是否需要让创建 entry 的 API 可以传 `folder_id` ？
