# API 文档

## API 基本规范

### 响应体

响应体规格如下。响应体都是 JSON 格式。

- 成功的响应

  ```json5
  {
    "status_code": 200,
    "content": {
      "success": true,
      "code": 0,
      "message": "success", // 自定义
      "data": { // data 的内容自定义
        "result": 1
      }
    }
  }
  ```

- 失败的响应

  ```json5
  {
    "status_code": 404, // 自定义
    "content": {
      "success": false,
      "code": 101, // 根据具体错误修改，见“错误代码”
      "message": "success", // 自定义，不建议用 success
      "data": { // data 的内容自定义
        "result": 1
      }
    }
  }
  ```

- 响应解释

  后端发出的请求是完整的响应体，其中的 status_code 字段会被浏览器处理，前端实际上应该读取的是 content 下的内容。前端应该根据 code 字段给出错误处理。

  以下路由下对于响应体的解释都是针对 data 字段。

  除特别说明外，失败的响应不需要 data 字段。

**为了正确、清晰地处理请求，方便前端处理，不要偷懒使用框架自带的错误处理！**

### 请求体

**注意请求体是 JSON、FORM 还是其它类型！**

所有请求都把 cookie 的加 session 字段作为 token。

### 字段类型说明

- 基本类型如 bool, int, string 等，略。
- datetime 类型，本质为 string 类型，形如 `yyyy-mm-dd hh:mm:ss`，二十四小时制。
- 组合类型 `{}` 和 `[]`：
  - `{<Type1>, <Type2>, ...}` 代表这个类型由 `<Type1>`, `<Type2>`, ... 几个类型组合而成。
  - `[<Type>]` 代表这个类型由任意多个 `<Type>` 类型组合而成。和 JSON 一样不需要在响应时给出 `<Type>` 类型名。
- 可以用字段的名称代表对应的字段类型

### 字段名称规范

- 一级字段，为响应体的 data 字段的下一级字段。
- n 级字段，命名为 `<一级字段名>:<二级字段名>:...:<n 级字段名>`。
- 在组合类型的表示中，如果 `Type` 表示为 `:<Type>` 则对应该字段的下一级字段中名称为 `Type` 的字段，否则必须遵循 n 级字段的命名规则。

## 注册登录

### 根路由

/user

如果已经登录则不应该响应这些请求。

### 用户注册

- 路径 /register
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体 表单数据

|   字段名   |  类型  | 可选 | 解释 | 备注 |
| :--------: | :----: | :--: | :--: | :--: |
|  username  | string |  -   |  -   |  -   |
|  password  | string |  -   |  -   |  -   |
|   email    | string |  -   |  -   |  -   |
| student_id | string |  -   | 学号 |  -   |
| real_name  | string |  -   | 实名 |  -   |

- 成功响应 无

### 用户密码登录

- 路径 /login
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体 表单数据

|  字段名  |  类型  |   可选   | 解释 |              备注              |
| :------: | :----: | :------: | :--: | :----------------------------: |
| username | string | &#10004; |  -   | username 和 email 必须选择一个 |
| password | string |    -     |  -   |               -                |
|  email   | string | &#10004; |  -   | username 和 email 必须选择一个 |

- 成功响应

| 字段名  | 类型 |    解释    |                    备注                    |
| :------ | :--: | :--------: | :----------------------------------------: |
| role    | int  | 用户的身份 | 暂定只有 USER 和 ADMIN 两种，分别为 0 和 1 |
| user_id | uuid |  用户 id   |                     -                      |

## 通知

### 根路由

/notification

|  通知类型  | 对应编号 |
| :--------: | :------: |
|    公告    |    0     |
| 帖子的评论 |    1     |
| 评论的回复 |    2     |
|    打赏    |    3     |

### 获取未读通知

- 路径 /unread
- 方法 GET
- 路径参数 无
- 查询参数

| 字段名 | 类型 |   可选   |        解释        |   备注    |
| :----: | :--: | :------: | :----------------: | :-------: |
| depth  | int  | &#10004; | 最多获取的通知个数 | 默认为 99 |

- 请求体 无
- 成功响应

| 字段名                       |                    类型                    |              解释              |                  备注                  |
| :--------------------------- | :----------------------------------------: | :----------------------------: | :------------------------------------: |
| not_read                     |                    int                     |          未读通知个数          |                   -                    |
| messages                     |                 [:message]                 |               -                |     该字段下最多 depth 个 message      |
| messages:message             | {:id, :type, :content, :url, :notified_at} |               -                |                   -                    |
| messages:message:id          |                    uuid                    |            通知 id             |                   -                    |
| messages:message:type        |                    int                     |            通知类型            |                   -                    |
| messages:message:content     |                   string                   |         通知的内容缩写         | 固定 30 个字符，超出的部分用省略号代替 |
| messages:message:url         |                   string                   | 通知的所在网址，比如帖子的地址 |                   -                    |
| messages:message:notified_at |                  datetime                  |            通知时间            |                   -                    |

### 搜索通知

- 路径 /search
- 方法 GET
- 路径参数 无
- 查询参数

|  字段名  |  类型  |   可选   |                 解释                 |       备注       |
| :------: | :----: | :------: | :----------------------------------: | :--------------: |
| key_word | string | &#10004; |                关键词                |  为空则搜索全部  |
|  status  |  bool  | &#10004; |           已读 1 或未读 0            |  为空则搜索全部  |
|   type   | [int]  | &#10004; | 可选任意多个通知类型，类型见“根路由” |  为空则搜索全部  |
|   page   |  int   | &#10004; |                第几页                | 为空则返回第一页 |
| per_page |  int   | &#10004; |             每页显示几个             | 为空则显示 15 个 |

- 请求体 无
- 成功响应

| 字段名                       |                        类型                         |              解释              |               备注               |
| :--------------------------- | :-------------------------------------------------: | :----------------------------: | :------------------------------: |
| total                        |                         int                         |               -                |                -                 |
| total_page                   |                         int                         |               -                |                -                 |
| page                         |                         int                         |               -                |                -                 |
| per_page                     |                         int                         |               -                |                -                 |
| messages                     |                     [:message]                      |         未读消息的简报         | 该字段下最多 per_page 个 message |
| messages:message             | {:id, :type, :status, :content, :url, :notified_at} |               -                |                -                 |
| messages:message:id          |                        uuid                         |            通知 id             |                -                 |
| messages:message:type        |                         int                         |            通知类型            |                -                 |
| messages:message:status      |                        bool                         |        已读 1 或未读 0         |                -                 |
| messages:message:content     |                       string                        |         通知的内容缩写         |                -                 |
| messages:message:url         |                       string                        | 通知的所在网址，比如帖子的地址 |                -                 |
| messages:message:notified_at |                      datetime                       |            通知时间            |                -                 |

### 获取通知完整信息

- 路径 /{notification_id}
- 方法 GET
- 路径参数

|     字段名      | 类型 | 解释 | 备注 |
| :-------------: | :--: | :--: | :--: |
| notification_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应

| 字段名      |   类型   |              解释              | 备注 |
| :---------- | :------: | :----------------------------: | :--: |
| type        |   int    |            通知类型            |  -   |
| content     |  string  |            通知内容            |  -   |
| url         |  string  | 通知的所在网址，比如帖子的地址 |  -   |
| notified_at | datetime |            通知时间            |  -   |

### 一键确认

- 路径 /read_all
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体 无
- 成功响应 无

## 公告

### 根路由

/billboard

### 获取所有公告

- 路径 /index
- 方法 GET
- 路径参数 无
- 查询参数

|   字段名   | 类型 |   可选   |     解释     |        备注        |
| :--------: | :--: | :------: | :----------: | :----------------: |
|    page    | int  | &#10004; |    第几页    |  为空则返回第一页  |
|  per_page  | int  | &#10004; | 每页显示几个 |  为空则显示 15 个  |
| max_length | int  | &#10004; | 缩写最长长度 | 为空显示 30 个字符 |

- 请求体 无
- 成功响应

| 字段名                       |             类型             |   解释   |           备注           |
| :--------------------------- | :--------------------------: | :------: | :----------------------: |
| total                        |             int              |    -     |            -             |
| total_page                   |             int              |    -     |            -             |
| page                         |             int              |    -     |            -             |
| per_page                     |             int              |    -     |            -             |
| messages                     |          [:message]          |    -     | 最多 per_page 个 message |
| messages:message             | {:id, :content, notified_at} |    -     |            -             |
| messages:message:id          |             uuid             | 公告 id  |            -             |
| messages:message:content     |            string            | 公告缩写 |     最长 max_length      |
| messages:message:notified_at |           datetime           | 公告时间 |            -             |

### 获取对应公告

- 路径 /{id}
- 方法 GET
- 路径参数

| 字段名 | 类型 |  解释   | 备注 |
| :----: | :--: | :-----: | :--: |
|   id   | uuid | 公告 id |  -   |

- 查询参数 无
- 请求体 无
- 成功响应

| 字段名      |   类型   |   解释   | 备注 |
| :---------- | :------: | :------: | :--: |
| content     |  string  | 公告内容 |  -   |
| notified_at | datetime | 公告时间 |  -   |

### 发布公告

**只有管理员可以发布公告**

- 路径 /create
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体

| 字段名  |  类型  | 可选 | 解释 | 备注 |
| :-----: | :----: | :--: | :--: | :--: |
|  title  | string |  -   |  -   |  -   |
| content | string |  -   |  -   |  -   |

- 成功响应 无

## 帖子

### 根路由

/post

### 创建帖子

- 路径 /create
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体

| 字段名  |   类型   |   可选   |        解释        |      备注      |
| :-----: | :------: | :------: | :----------------: | :------------: |
|  cost   |   int    | &#10004; |      收费金额      | 默认为 0，免费 |
|  tags   | [string] | &#10004; | tag 的**名称**集合 |    默认为空    |
|  title  |  string  |    -     |         -          |       -        |
| content |  string  |    -     |         -          |       -        |
| bhpan_url | string | - | - | - |

- 成功响应

| 字段名  |  类型  |    解释    | 备注 |
| :------ | :----: | :--------: | :--: |
| post_id |  uuid  |     -      |  -   |
| url     | string | 帖子的地址 |  -   |

### 获取帖子内容

- 路径 /{post_id}
- 方法 GET
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应

| 字段名              |                    类型                    |          解释          |                     备注                     |
| :------------------ | :----------------------------------------: | :--------------------: | :------------------------------------------: |
| post_id             |                    uuid                    |           -            |                      -                       |
| url                 |                   string                   |       帖子的地址       |                      -                       |
| cost                |                    int                     |           -            |                未支付也可以看                |
| tags                |                  [string]                  |   tag 的**名称**集合   |                未支付也可以看                |
| title               |                   string                   |           -            |                未支付也可以看                |
| content             |                   string                   |           -            |         未支付应该返回 null 或者空值         |
| bhpan_url | string | bhpan的地址 |-|
| paid                |                    bool                    |       是否支付过       |         支付过返回 true，否则 false          |
| created_at          |                  datetime                  |           -            |                      -                       |
| favorites           |                    int                     |        收藏个数        |                      -                       |
| likes               |                    int                     |        点赞个数        |                      -                       |
| dislikes            |                    int                     |        点踩个数        |                      -                       |
| ~~sponsors~~        |                    int                     |     投币的用户个数     | 对于付费的帖子，这里还应该计算付费用户的个数 |
| ~~coins~~           |                    int                     |       收到的菜币       |                     同上                     |
| created_by          | {:user_id, :username, :url, :fans, :posts} |       创建者信息       |                      -                       |
| created_by:user_id  |                    uuid                    |           -            |                      -                       |
| created_by:username |                   string                   |           -            |                      -                       |
| ~~created_by:url~~  |                   string                   | 用户个人主页所在的地址 |                      -                       |
| ~~created_by:fans~~ |                    int                     |     用户的粉丝数量     |                      -                       |
| ~~created_by:posts~~ |                    int                     |     发布的帖子数量     |                      -                       |
| like                |                    bool                    |        是否点赞        |                      -                       |
| dislike             |                    bool                    |        是否点踩        |                      -                       |
| favorite            |                    bool                    |        是否收藏        |                      -                       |


### 获取帖子评论

- 路径 /{post_id}/comments
- 方法 GET
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数

|  字段名  | 类型 |   可选   | 解释 |   备注    |
| :------: | :--: | :------: | :--: | :-------: |
|   page   | int  | &#10004; |  -   | 默认为 30 |
| per_page | int  | &#10004; |  -   | 默认为 1  |

- 请求体 无
- 成功响应

| 字段名                               |                             类型                             |          解释          |                             备注                             |
| :----------------------------------- | :----------------------------------------------------------: | :--------------------: | :----------------------------------------------------------: |
| page                                 |                             int                              |           -            |                       未支付返回 null                        |
| per_page                             |                             int                              |           -            |                       未支付返回 null                        |
| total                                |                             int                              |           -            |                       未支付返回 null                        |
| total_page                           |                             int                              |           -            |                       未支付返回 null                        |
| comments                             |                          [:comment]                          |           -            |                       未支付返回 null                        |
| comments:comment                     | {:comment_id, :content, :created_by, :created_at, :likes, :dislikes, :parent_id, :like, :dislike} |           -            |                              -                               |
| comments:comment:comment_id          |                             uuid                             |           -            |                              -                               |
| comments:comment:content             |                            string                            |           -            |                              -                               |
| comments:comment:created_at          |                           datetime                           |           -            |                              -                               |
| ~~comments:comment:likes~~           |                             int                              |           -            |                              -                               |
| ~~comments:comment:dislikes~~        |                             int                              |           -            |                              -                               |
| comments:comment:parent_id           |                             uuid                             |   评论的父级评论 id    | 如果评论是直接对帖子的回复，那么这个字段是 0；如果评论是对某个评论的回复，那么这个字段是被回复的评论的 uuid |
| comments:comment:created_by          |                 {:user_id, :username, :url}                  |      评论的创建者      |                              -                               |
| comments:comment:created_by:user_id  |                             uuid                             |           -            |                              -                               |
| comments:comment:created_by:username |                            string                            |           -            |                              -                               |
| comments:comment:created_by:url      |                            string                            | 用户个人主页所在的地址 |                              -                               |
| comments:comment:like                |                             bool                             |        是否点赞        |                              -                               |
| comments:comment:dislike             |                             bool                             |        是否点踩        |                              -                               |

### 创建回复

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/comments/create
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体

|  字段名   |  类型  |   可选   |    解释     |   备注   |
| :-------: | :----: | :------: | :---------: | :------: |
|   title   | string |    -     |      -      |    -     |
|  content  | string |    -     |      -      |    -     |
| parent_id |  uuid  | &#10004; | 父级评论 id | 默认为 0 |

- 成功响应 无

### 收藏帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/favour
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 取消收藏帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/not_favour
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 点赞帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/like
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 取消点赞帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/not_like
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 点踩帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/dislike
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 取消点踩帖子

如果付费内容没有购买则不应该成功。

- 路径 /{post_id}/not_dislike
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 搜索帖子

- 路径 /search
- 方法 GET
- 路径参数 无
- 查询参数

|   字段名   |   类型   |   可选   |        解释        |                             备注                             |
| :--------: | :------: | :------: | :----------------: | :----------------------------------------------------------: |
|    page    |   int    | &#10004; |         -          |                           默认为 1                           |
|  per_page  |   int    | &#10004; |         -          |                          默认为 30                           |
|    tags    | [string] | &#10004; | tag 的**名称**集合 |                          为空则忽略                          |
|    pay     |   bool   | &#10004; |      是否付费      |                         默认全部搜索                         |
|  sort_by   |   int    | &#10004; |      排序方式      | （默认）推荐算法 0，点赞 1，最近创建 2，最近评论 3，收藏量 4 |
|  key_word  |  string  |    -     |         -          |                            关键词                            |
| max_length |   int    | &#10004; | 帖子标题的最大长度 |                           默认 30                            |

- 请求体 无
- 成功响应

| 字段名                         |                                             类型                                              |     解释     |         备注          |
| :----------------------------- | :-------------------------------------------------------------------------------------------: | :----------: | :-------------------: |
| page                           |                                              int                                              |      -       |           -           |
| per_page                       |                                              int                                              |      -       |           -           |
| total                          |                                              int                                              |      -       |           -           |
| total_page                     |                                              int                                              |      -       |           -           |
| posts                          |                                            [:post]                                            |      -       |           -           |
| posts:post                     | {:post_id, :post_url, :title, :created_by, :created_at, :likes, :dislikes, :favorites, :cost} |      -       |           -           |
| posts:post:post_id             |                                             uuid                                              |      -       |           -           |
| posts:post:post_url            |                                            string                                             |      -       |           -           |
| posts:post:title               |                                            string                                             |      -       | 长度不超过 max_length |
| posts:post:created_by          |                               {:username, :user_url, :user_id}                                |      -       |           -           |
| posts:post:created_by:username |                                            string                                             |      -       |           -           |
| posts:post:created_by:user_id  |                                             uuid                                              |      -       |           -           |
| posts:post:created_by:user_url |                                            string                                             | 用户主页地址 |           -           |
| posts:post:created_at          |                                           datetime                                            |      -       |           -           |
| posts:post:likes               |                                              int                                              |      -       |           -           |
| posts:post:dislikes            |                                              int                                              |      -       |           -           |
| posts:post:favorites           |                                              int                                              |      -       |           -           |
| posts:post:cost                |                                              int                                              |   资源价格   |       免费为 0        |

### 获取自己分享的帖子

* 路径 own 
* 方法 GET
* 路径参数 无
* 查询参数 无


- 请求体 无
- 成功响应

| 字段名                         |                                             类型                                              |     解释     |         备注          |
| :----------------------------- | :-------------------------------------------------------------------------------------------: | :----------: | :-------------------: |
| posts                          |                                            [:post]                                            |      -       |           -           |
| posts:post                     | {:post_id, :post_url, :title, :created_by, :created_at, :likes, :dislikes, :favorites, :cost} |      -       |           -           |
| posts:post:post_id             |                                             uuid                                              |      -       |           -           |
| posts:post:post_url            |                                            string                                             |      -       |           -           |
| posts:post:title               |                                            string                                             |      -       | 长度不超过 max_length |
| posts:post:created_at          |                                           datetime                                            |      -       |           -           |
| posts:post:likes               |                                              int                                              |      -       |           -           |
| posts:post:dislikes            |                                              int                                              |      -       |           -           |
| posts:post:favorites           |                                              int                                              |      -       |           -           |
| posts:post:cost                |                                              int                                              |   资源价格   |       免费为 0        |

### 修改自己的某个帖子

- 路径 /change

- 方法 POST

- 路径参数 无

- 查询参数 无

- 请求体

    下面几项至少有一个非空，为空的值表示无需改变

    |  字段名   |   类型   |   可选   | 解释 | 备注 |
    | :-------: | :------: | :------: | :--: | :--: |
    |   cost    |   int    | &#10004; |  -   |  -   |
    |   tags    | [string] | &#10004; |  -   |  -   |
    |   title   |  string  | &#10004; |  -   |  -   |
    |  content  |  string  | &#10004; |  -   |  -   |
    | bhpan_url |  string  | &#10004; |  -   |  -   |

- 成功响应

## 任务

### 根路由

/mission

### 创建任务

- 路径 /create
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体

|   字段名   |   类型   |   可选   |        解释        |   备注   |
| :--------: | :------: | :------: | :----------------: | :------: |
| commission |   int    | &#10004; |        报酬        | 默认为 0 |
|    tags    | [string] | &#10004; | tag 的**名称**集合 | 默认为空 |
|   title    |  string  |    -     |         -          |    -     |
|  content   |  string  |    -     |         -          |    -     |

- 成功响应

| 字段名     |  类型  |    解释    | 备注 |
| :--------- | :----: | :--------: | :--: |
| mission_id |  uuid  |     -      |  -   |
| url        | string | 任务的地址 |  -   |

### 获取任务内容

- 路径 /{mission_id}
- 方法 GET
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| mission_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应

| 字段名         |   类型   |        解释        |        备注         |
| :------------- | :------: | :----------------: | :-----------------: |
| mission_id     |   uuid   |         -          |          -          |
| url            |  string  |     帖子的地址     |          -          |
| commission     |   int    |        佣金        |          -          |
| open           |   bool   |      是否有效      |          -          |
| tags           | [string] | tag 的**名称**集合 |          -          |
| title          |  string  |         -          |          -          |
| profile        |  string  |         -          |          -          |
| content        |  string  |         -          |          -          |
| submitted      |   bool   |     是否提交过     |          -          |
| created_at     | datetime |         -          |          -          |
| submit_at      | datetime |    上次提交时间    | 没有提交则返回 null |
| submit_content |  string  |    上次提交内容    | 没有提交则返回 null |

### 提交任务

创建任务的人不能提交。

- 路径 /{mission_id}/submit
- 方法 POST
- 路径参数

|   字段名   | 类型 | 解释 | 备注 |
| :--------: | :--: | :--: | :--: |
| mission_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 表单数据

| 字段名  |  类型  | 可选 |     解释     |          备注          |
| :-----: | :----: | :--: | :----------: | :--------------------: |
| profile | string |  -   |   提交简介   |  未支付时只能看到简介  |
| content | string |  -   | 完整提交内容 | 支付后看到完整提交内容 |

- 成功响应 无

### 获取任务回复

只有发布任务的人可以查看。

- 路径 /{mission_id}/submits
- 方法 GET
- 路径参数

|   字段名   | 类型 | 解释 | 备注 |
| :--------: | :--: | :--: | :--: |
| mission_id | uuid |  -   |  -   |

- 查询参数

|  字段名  | 类型 |   可选   | 解释 |   备注    |
| :------: | :--: | :------: | :--: | :-------: |
|   page   | int  | &#10004; |  -   | 默认为 30 |
| per_page | int  | &#10004; |  -   | 默认为 1  |

- 请求体 表单数据
- 成功响应

| 字段名                   |          类型          |      解释      |                               备注                                |
| :----------------------- | :--------------------: | :------------: | :---------------------------------------------------------------: |
| page                     |          int           |       -        |                                 -                                 |
| per_page                 |          int           |       -        |                                 -                                 |
| total                    |          int           |       -        |                                 -                                 |
| total_page               |          int           |       -        |                                 -                                 |
| submits                  |       [:submit]        |       -        |                                 -                                 |
| submits:submit           | {:submit_id, :profile} |       -        |                                 -                                 |
| submits:submit:submit_id |          uuid          |       -        |                                 -                                 |
| submits:submit:profile   |         string         | 用户的提交简介 | 当用户关闭任务时，选择的提交记录的 profile 内容变成对应的 content |

### 关闭任务

创建任务才能关闭。

- 路径 /{mission_id}/close
- 方法 POST
- 路径参数

|   字段名   | 类型 | 解释 | 备注 |
| :--------: | :--: | :--: | :--: |
| mission_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 表单数据

|  字段名  |  类型  | 可选 |         解释          |   备注   |
| :------: | :----: | :--: | :-------------------: | :------: |
| accepted | [uuid] |  -   | 选中的任务提交记录 id | 至少一个 |

- 成功响应 无

### 搜索任务

- 路径 /search
- 方法 GET
- 路径参数 无
- 查询参数

|   字段名   |   类型   |   可选   |        解释        |                        备注                        |
| :--------: | :------: | :------: | :----------------: | :------------------------------------------------: |
|    page    |   int    | &#10004; |         -          |                     默认为 30                      |
|  per_page  |   int    | &#10004; |         -          |                      默认为 1                      |
|    tags    | [string] | &#10004; | tag 的**名称**集合 |                         -                          |
|   status   |   bool   | &#10004; |      是否有效      |                    默认全部搜索                    |
|  sort_by   |   int    | &#10004; |      排序方式      | （默认）推荐算法 0，最近创建 2，最近提交 3，佣金 4 |
|  key_word  |  string  |    -     |         -          |                         -                          |
| max_length |   int    | &#10004; | 任务标题的最大长度 |                      默认 30                       |

- 请求体 无
- 成功响应

| 字段名                |                         类型                          | 解释 |         备注          |
| :-------------------- | :---------------------------------------------------: | :--: | :-------------------: |
| page                  |                          int                          |  -   |           -           |
| per_page              |                          int                          |  -   |           -           |
| total                 |                          int                          |  -   |           -           |
| total_page            |                          int                          |  -   |           -           |
| posts                 |                        [:post]                        |  -   |           -           |
| posts:mission         | {:mission_id, :url, :title, :created_at, :commission} |  -   |           -           |
| posts:post:mission_id |                         uuid                          |  -   |           -           |
| posts:post:url        |                        string                         |  -   |           -           |
| posts:post:title      |                        string                         |  -   | 长度不超过 max_length |
| posts:post:created_at |                       datetime                        |  -   |           -           |
| posts:post:commission |                          int                          | 佣金 |           -           |

## 用户个人信息

### 根路由

/user/{user_id}

- 路径参数
| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| user_id | uuid |  -   |  -   |

### 简介

- 路径 /profile
- 方法 GET
- 路径参数 无
- 查询参数 无
- 请求体 无
- 成功响应

| 字段名     |   类型    |      解释      |                    备注                    |
| :--------- | :-------: | :------------: | :----------------------------------------: |
| role       |    int    |    用户身份    | 暂定只有 USER 和 ADMIN 两种，分别为 0 和 1 |
| created_at | datetime  |    注册时间    |                     -                      |
| avatar     |  string   | 头像对应的邮箱 |            暂定，具体看前端实现            |
| signature  |  string   |    个性签名    |                     -                      |
| username   |  string   |       -        |                     -                      |
| email      |  string   |       -        |                     -                      |
| likes      |    int    |    收到的赞    |                     -                      |
| fans       |    int    |    粉丝数量    |                     -                      |
| follows    |    int    |    关注数量    |                     -                      |
| favorites  |    int    |    收藏数量    |                     -                      |
| posts      |    int    |    发帖数量    |                     -                      |
| replies    |    int    |    回复数量    |                     -                      |
| capital    | int/long? |    菜币数量    |                     -                      |
| avatarurl | avatarurl | 头像url | 若图像未设置，应该返回null |


### 关注的人

- 路径 /follows
- 方法 GET
- 路径参数 无
- 查询参数

|  字段名  | 类型 |   可选   |     解释     |       备注       |
| :------: | :--: | :------: | :----------: | :--------------: |
|   page   | int  | &#10004; |    第几页    | 为空则返回第一页 |
| per_page | int  | &#10004; | 每页显示几个 | 为空则显示 15 个 |

- 请求体 无
- 成功响应

| 字段名     |  类型  | 解释 |          备注           |
| :--------- | :----: | :--: | :---------------------: |
| total      |  int   | 总数 |            -            |
| total_page |  int   |  -   |            -            |
| page       |  int   |  -   |            -            |
| per_page   |  int   |  -   |            -            |
| users      | [uuid] |  -   | uuid 不超过 per_page 个 |

### 粉丝

- 路径 /fans
- 方法 GET
- 路径参数 无
- 查询参数

|  字段名  | 类型 |   可选   |     解释     |       备注       |
| :------: | :--: | :------: | :----------: | :--------------: |
|   page   | int  | &#10004; |    第几页    | 为空则返回第一页 |
| per_page | int  | &#10004; | 每页显示几个 | 为空则显示 15 个 |

- 请求体 无
- 成功响应

| 字段名     |  类型  | 解释 |          备注           |
| :--------- | :----: | :--: | :---------------------: |
| total      |  int   | 总数 |            -            |
| total_page |  int   |  -   |            -            |
| page       |  int   |  -   |            -            |
| per_page   |  int   |  -   |            -            |
| users      | [uuid] |  -   | uuid 不超过 per_page 个 |

### 收藏

- 路径 /favorites
- 方法 GET
- 路径参数 无
- 查询参数

|   字段名   | 类型 |   可选   |      解释      |       备注       |
| :--------: | :--: | :------: | :------------: | :--------------: |
|    page    | int  | &#10004; |     第几页     | 为空则返回第一页 |
|  per_page  | int  | &#10004; |  每页显示几个  | 为空则显示 15 个 |
| max_length | int  | &#10004; | 简介的最大长度 | 缺省为 30 个字符 |

- 请求体 无
- 成功响应

| 字段名                              |                          类型                          |        解释        |        备注        |
| :---------------------------------- | :----------------------------------------------------: | :----------------: | :----------------: |
| total                               |                          int                           |        总数        |         -          |
| total_page                          |                          int                           |         -          |         -          |
| page                                |                          int                           |         -          |         -          |
| per_page                            |                          int                           |         -          |         -          |
| favorites                           |                       [favorite]                       |         -          |         -          |
| favorites:favorite                  | {:title, :content, :post_by, :created_at, :updated_at} |         -          | 不超过 per_page 个 |
| favorites:favorite:title            |                         string                         |         -          | 不超过 max_length  |
| favorites:favorite:post_by          |                   {:username, :url}                    |         -          |         -          |
| favorites:favorite:post_by:username |                         string                         | 帖子创建者的用户名 |         -          |
| favorites:favorite:post_by:url      |                         string                         |   用户的个人主页   |         -          |
| favorites:favorite:created_at       |                        datetime                        |         -          |         -          |
| favorites:favorite:updated_at       |                        datetime                        |         -          |         -          |

### 修改信息

**注意鉴权，只有用户本人和管理员可以操作！**

- 路径 /modify
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体 表单数据

|  字段名   |  类型  |   可选   | 解释 |    备注    |
| :-------: | :----: | :------: | :--: | :--------: |
| password  | string | &#10004; |  -   | 为空不修改 |
|   email   | string | &#10004; |  -   | 为空不修改 |
| signature | string | &#10004; |  -   | 为空不修改 |

- 成功响应 无

## 标签

### 根路由

/tags

### 创建标签

- 路径 /create
- 方法 POST
- 路径参数 无
- 查询参数 无
- 请求体 表单数据

| 字段名 |  类型  | 可选 |   解释   | 备注 |
| :----: | :----: | :--: | :------: | :--: |
|  name  | string |  -   | 标签名称 |  -   |

- 成功响应 无

### 查询标签

- 路径 /search
- 方法 GET
- 路径参数 无
- 查询参数

|  字段名  |  类型  |   可选   |   解释   |    备注    |
| :------: | :----: | :------: | :------: | :--------: |
| key_word | string | &#10004; | 标签名称 | 为空返回全 |

- 请求体 无
- 成功响应

| 字段名 |   类型   |         解释         | 备注 |
| :----- | :------: | :------------------: | :--: |
| tags   | [string] | 标签列表，内容为名字 |  -   |

## 补充（普通用户）

之前的api文档缺少一些必须的api，为了方便起见都统一写在了此处。

如果觉得路径不合理可以去改。

### 查询是否关注某人

* 路径 /query_follow

- 方法 GET
- 路径参数 无
- 查询参数

| 字段名  | 类型 | 可选 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: | :--: |
| user_id | uuid |  -   |  -   |  -   |

- 请求体 无
- 成功响应 
| 字段名   |  类型   |                       解释                        | 备注 |
  | :------- | :-----: | :-----------------------------------------------: | :--: |
  | followed | boolean | 如果关注了查询的这个用户,则返回true,否则返回false |  -   |

### 关注某人

- 路径 follow
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| user_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 取消关注某人

- 路径 not_follow
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| user_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无
- 成功响应 无

### 发布帖子的评论

wxf写的什么勾八，啥都缺

给一个分享贴评论，这里先不实现多级评论了

- 路径 /comment

- 方法 POST

- 路径参数 

  | 字段名  | 类型 | 解释 | 备注 |
  | :-----: | :--: | :--: | :--: |
  | post_id | uuid |  -   |  -   |

- 查询参数 无

- 请求体 表单数据

    |  字段名   |  类型  |   可选   | 解释 |    备注    |
    | :-------: | :----: | :------: | :--: | :--------: |
    | text  | string | - | 评论内容 | - |

- 成功响应 无

### 上传头像

* 路径 /updata_avatar

* 方法 post

* 路径参数 无

* 查询参数 无

* 请求体 表单数据

    |  字段名   |  类型  |   可选   | 解释 |    备注    |
    | :-------: | :----: | :------: | :--: | :--------: |
    | avatar | File | - | 新头像 | - |
    
* 成功响应

    |   字段名   | 类型 | 解释 | 备注 |
    | :--------: | :--: | :--: | :--: |
    | avatar_url | uuid |  -   |  -   |

## 补充（管理员）
### 封禁某个账号

只有管理员才能封禁某个账号。

- 路径 /block
- 方法 POST
- 路径参数

    | 字段名  | 类型 | 解释 | 备注 |
    | :-----: | :--: | :--: | :--: |
    | user_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无

- 成功响应 无

### 解封某个账号

只有管理员才能封禁某个账号。

- 路径 /unblock

- 方法 POST

- 路径参数

  | 字段名  | 类型 | 解释 | 备注 |
  | :-----: | :--: | :--: | :--: |
  | user_id | uuid |  -   |  -   |

- 查询参数 无

- 请求体 无

- 成功响应 无

### 删除某个帖子

只有管理员才可以删除，帖子发布者也不可以（主要是因为懒得实现前端界面了）。

- 路径 /deletePost
- 方法 POST
- 路径参数

| 字段名  | 类型 | 解释 | 备注 |
| :-----: | :--: | :--: | :--: |
| post_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无

- 成功响应 无

### 删除某个任务

只有管理员才可以删除，任务发布者也不可以（主要是因为懒得实现前端界面了）。

删除某个任务后，菜币没收（因为任务肯定是因为违规才会被删除，没收提问者的菜币很合理）。

- 路径 /deleteMission
- 方法 POST
- 路径参数

|   字段名   | 类型 | 解释 | 备注 |
| :--------: | :--: | :--: | :--: |
| mission_id | uuid |  -   |  -   |

- 查询参数 无
- 请求体 无

- 成功响应 无



## 错误代码

|              名称               | 值  |             含义              | 备注 |
| :-----------------------------: | :-: | :---------------------------: | :--: |
|             SUCCESS             |  0  |             成功              |  -   |
|              ERROR              | 100 |         未定义的错误          |  -   |
|         INTERNAL_ERROR          | 101 | 内部错误被 catch 之后的返回值 |  -   |
|          FORMAT_ERROR           | 102 |         请求格式错误          |  -   |
|            NET_ERROR            | 103 |           网络错误            |  -   |
|            NOT_FOUND            | 104 |       找不到对应的资源        |  -   |
|           USER.ERROR            | 200 |       和用户有关的错误        |  -   |
|   USER.REGISTER.USERNAME_USED   | 201 |          用户名重复           |  -   |
|   USER.REGISTER.WEAK_PASSWORD   | 202 |           密码太弱            |  -   |
|    USER.REGISTER.EMAIL_USED     | 203 |           邮箱重复            |  -   |
|     USER.REGISTER.NOT_MATCH     | 204 |       学号和实名不匹配        |  -   |
| USER.REGISTER.STUDENT_NOT_FOUND | 205 |        找不到学生信息         |  -   |
|       USER.REGISTER.ERROR       | 206 |           其他错误            |  -   |
|  USER.LOGIN.ACCOUNT_NOT_FOUND   | 207 |       登陆时找不到账号        |  -   |
|    USER.LOGIN.WRONG_PASSWORD    | 208 |           密码错误            |  -   |
|       USER.LOGIN.BLOCKED        | 209 |          用户被封禁           |  -   |
|        USER.LOGIN.ERROE         | 210 |           其他错误            |  -   |
|         USER.NOT_FOUND          | 211 |     找不到对应的用户信息      |  -   |
|         USER.NOT_LOGGIN         | 212 |          用户未登录           |  -   |
|       USER.ALREADY_LOGGIN       | 213 |         用户重复登录          |  -   |
|       USER.NOT_PRIVILEGED       | 214 |         用户权限不足          |  -   |
|       NOTIFICATION.ERROR        | 300 |       和通知有关的错误        |  -   |
|     NOTIFICATION.NOT_FOUND      | 301 |          找不到通知           |  -   |
|         BILLBOARD.ERROR         | 400 |       和公告有关的错误        |  -   |
|     BILLBOARD.CREATE_ERROR      | 401 |         公告创建失败          |  -   |
|       BILLBOARD.NOT_FOUND       | 402 |          找不到公告           |  -   |
|           POST.ERROR            | 500 |       和帖子有关的错误        |  -   |
|        POST.CREATE_ERROR        | 501 |         帖子创建失败          |  -   |
|         POST.NOT_FOUND          | 502 |          找不到帖子           |  -   |
|          POST.NOT_PAID          | 503 |          帖子未支付           |  -   |
|    POST.COMMENT.CREATE_ERROR    | 504 |           评论失败            |  -   |
|     POST.COMMENT.NOT_FOUND      | 505 |          找不到评论           |  -   |
|          MISSION.ERROR          | 500 |       和任务有关的错误        |  -   |
|      MISSION.CREATE_ERROR       | 501 |         任务创建失败          |  -   |
|        MISSION.NOT_FOUND        | 502 |          找不到任务           |  -   |
|      MISSION.NOT_SUBMITTED      | 503 |          任务未提交           |  -   |
|   MISSION.SUBMIT.CREATE_ERROR   | 504 |         任务提交失败          |  -   |
|    MISSION.SUBMIT.NOT_FOUND     | 505 |      找不到任务提交记录       |  -   |
|      MISSION.SUBMIT.ERROR       | 506 |           其他错误            |  -   |
|      MISSION.CLOSE_FAILED       | 507 |         任务关闭失败          |  -   |
|       MISSION.NOT_CLOSED        | 508 |          任务未关闭           |  -   |
|            PAY.ERROR            | 600 |       支付菜币相关错误        |  -   |
|       PAY.NO_ENOUGH_MONEY       | 601 |           菜币不够            |  -   |