# 系统

## System

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | string         | Key         |
| ...   | any            | data        |

## System.UserCounter

For auto-increasing user collection id.

```js
{
  _id: 'UserCounter',
  count: int
}
```

## System.UserContactType

```js
{
  _id: 'UserContactType',
  values: string[]
}
```

### Example:

```js
{
  _id: 'UserContactType',
  values: ['QQ', 'Mail']
}
```

## System.FilterKeyword

```js
{
  _id: 'UserContactType',
  general: string[],
  name: string[]
}
```

## System.ProblemTemplate

```js
{
  _id: 'ProblemTemplate',
  markdown: string
}
```

# 用户、权限、域

## User

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| uid   | int64          | UserID      |
| user  | string         | 用户名       |
| luser | string         | 小写用户名    |
| mail  | string         | Email       |
| lmail | string         | 小写 Email   |
| omail | string         | 当多个账户同 Email 时，mail 字段随机，该字段存储 Email |
| salt  | string         |             |
| hash  | string         |             |
| g     | string         | Gravatar mail |
| gender | int64         | 性别         |
| regat | mongodate      | 注册时间      |
| regip | string         | 注册 IP      |
| loginat | mongodate    | 上次登录时间  |
| loginip | string       | 上次登录 IP   |
| banned | boolean       | 是否已停用    |

其中，hash 为以下两种格式中的一种(in `\VJ\User\PasswordEncoder`):
- `vj2|username|{hash-hex}`
- `openvj|{bcrypt-hash-hex}`

### Index

- uid(unique)
- luser(unique)
- lmail(uniqie)

## UserRole

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| uid   | int64          |             |
| d.{domain} | string[]  | {domain} 域下的角色 |

### Example

```js
{
  _id: ObjectId(..),
  uid: 1,
  d: {
    "000000000000000000000000": ["DOMAIN_MEMBER"]
    "123456781234567812345678": ["DOMAIN_OWNER", "DOMAIN_MEMBER"]
  }
}
```

### Index

- uid(unique)

## PermissionAllow

| field | type           | description     |
| ----- | -------------- | --------------- |
| _id   | mongoid        |                 |
| domain | mongoid       | 域               |
| val   | string         | 权限名           |
| role  | string         | 角色名           |

### Index

- domain, val, role

## Role

| field | type           | description     |
| ----- | -------------- | --------------- |
| _id   | mongoid        |                 |
| name  | string         | 角色名           |
| internal | boolean     | 是否是系统内部角色 |
| domain | mongoid       | 所属域           |
| owner | int64          | 创建者           |
| at    | mongodate      | 创建时间         |

### Note

- 该表仅供管理页面中管理权限时使用
- 对于系统内部角色，`internal = true`，无 `domain`, `owner`, `at`，管理页面中不能删除
- 对于非系统内部角色，`name` 前必须包含前缀 `$$`

### 系统内部角色

- EVERYONE（所有人，含未登录用户）
- OWNER（资源所有者）
- DOMAIN_OWNER（域所有者）
- DOMAIN_MEMBER（域成员）

### Index

- domain, name(unique)
- internal, name(unique)

## UserInfo

| field | type           | description | which domain  |
| ----- | -------------- | ----------- | ------------- |
| _id   | mongoid        |             | all           |
| uid   | int64          |             | all           |
| domain | mongoid       | 所属域       | all           |
| pref  | document       | 偏好设置      | all           |
| sig   | string         | 签名 HTML    | root          |
| sigraw | string        | 原始签名      | root          |
| contacts | contact[]   | 联系方式      | root          |
| rp    | float          | RP           | all          |
| rp_s  | float          | RP 静态部分   | all           |
| rank  | int64          | 排名         | all           |
| level | int64          | 等级         | all           |

### type:contact

```js
{
  type: string,
  val: string,
  visibility: int
}
```

### Index

- uid,domain(unique)

## Domain

// unstable

| field | type           | description     |
| ----- | -------------- | --------------- |
| _id   | mongoid        |                 |
| name  | string         | 域名称           |
| owner | int64          | 创建者           |
| at    | mongodate      | 创建时间         |
| invalid | boolean      | 是否无效         |

## DomainLog

| field | type           | description     | which type  |
| ----- | -------------- | --------------- | ----------- |
| _id   | mongoid        |                 | all         |
| uid   | int64          | 操作用户          | all         |
| at    | mongodate      | 时间             | all         |
| type  | string         | 操作类型          | all         |
| ua    | string         | User-Agent      | all         |
| ip    | string         | IP              | all         |
| target_uid | int64       | 被操作用户        | join      |
| target_domain | mongoid  | 加入的域          | join      |

# 令牌

## Token

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| purpose | string       | 令牌类型      |
| identifier | any       | 唯一标示符    |
| token | string         | 令牌         |
| expireat | mongodate   | 过期时间      |
| data  | document       | 附加数据      |

### For Registration

| field | type           | value       |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| purpose | string       | reg         |
| identifier | any       | email       |
| token | string         | token       |
| expireat | mongodate   | 过期时间      |
| data  | document       | ['email' => email]|

### For RememberMe

| field | type           | value       |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| purpose | string       | rememberme         |
| identifier | any       | uid       |
| token | string         | token       |
| expireat | mongodate   | 过期时间      |
| data  | document       | ['uid' => uid, 'ua' => user-agent, 'ip' => ip-address]|


### Index

- purpose, identifier(unique)
- purpose, token
- expireat ([TTLIndex](http://docs.mongodb.org/manual/tutorial/expire-data/#expire-documents-at-a-certain-clock-time))

# 会话、登录

## Session

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | string         | session id  |
| expireat | mongodate   | 过期时间      |
| data  | document       | 数据         |

### Index

- expireat ([TTLIndex](http://docs.mongodb.org/manual/tutorial/expire-data/#expire-documents-at-a-certain-clock-time))

## LoginLog

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| uid   | int64          | 用户 ID      |
| at    | mongodate      | 登录时间      |
| type  | string         | 登录类型      |
| ua    | string         | User-Agent  |
| ip    | string         | IP          |

### Index

- uid, at(desc)

# 评论与投票

## (abstract) Votable

| field | type           | description   |
| ----- | -------------- | ------------- |
| voting | int64         | 投票值         |
| votes | voteDocument   | 投票          |

### type:voteDocument

```js
{
  uid_1: v_1,
  uid_2: v_2,
  ...
}
```

## Comment : Votable

| field | type           | description   |
| ----- | -------------- | ------------- |
| _id   | mongoid        |               |
| ref   | string         | 关联键         |
| owner | int64          | 创建者         |
| at    | mongodate      | 创建时间       |
| raw   | string         | Markdown 内容 |
| html  | string         | HTML 内容     |
| replies | reply[]      | 子回复         |
| deleted | boolean      | 是否已删除      |
| modifyat | mongodate   | 修改时间 optional |

### type:reply

```js
{
  _id: mongoid,
  owner: int,
  at: mongodate,
  raw: string,
  html: string,
  deleted: boolean,
  modifyat: mongodate //optional
}
```

###type:discussion
```js
{
	_id: mongoid,
	owner: int,
	topicid: mongoid,
	title: string,
	raw: string,
	html: string,
	at: mongodate
}
```

###type:topic
```js
	_id: mongoid,
	name: string
```

### Index

- ref, deleted, _id(asc)

# 题目

## Problem : Votable

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| owner | int64          | 所有者       |
| at    | mongodate      | 创建时间      |
| title | string         | 题目标题      |
| link  | string         | URL         |
| llink | string         | 小写 URL     |
| raw   | string         | 题目内容 Markdown |
| html  | string         | HTML         |
| visible | mongoid[]    | 可见域        |
| tags  | tag[]          | 标签         |
| history | bin          | 历史版本 gzip (json_encode(history[])) |

### type:tag

```js
{
  type: string,
  parts: mixed[],
  name: string
}
```

### type:history

```js
{
  owner: int,
  at: mongodate,
  raw: string
}
```

### Index

- owner, llink(unique)
