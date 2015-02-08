# 系统

## System

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | string         | Key         |
| ...   | any            | data        |

## System.UserContactType

```js
{
  values: [string valueN]
}
```

### Example:

```js
{
  values: ["QQ", "Mail"]
}
```

# 用户、权限、域

## User

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | int64          |             |
| user  | string         | 用户名       |
| luser | string         | 小写用户名    |
| mail  | string         | Email       |
| lmail | string         | 小写 Email   |
| omail | string         | 当多个账户同 Email 时，mail 字段随机，该字段存储 Email |
| salt  | string         |             |
| hash  | string         |             |
| g     | string         | Gravatar mail |
| gender | int64         | 性别         |
| regat | date           | 注册时间      |
| regip | string         | 注册 IP      |
| loginat | date         | 上次登录时间  |
| loginip | string       | 上次登录 IP   |
| banned | boolean       | 是否已停用    |
| deleted | boolean      | 是否已删除    |

其中，hash 为以下两种格式中的一种(in `\VJ\User\PasswordEncoder`):
- `vj2|username|{hash-hex}`
- `openvj|{bcrypt-hash-hex}`

### Index

- luser
- lmail

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
    "000000000000000000000000": ["MEMBER"]
    "123456781234567812345678": ["OWNER", "MEMBER"]
  }
}
```

### Index

- uid

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
| at    | date           | 创建时间         |

### Note

- 该表仅供管理页面中管理权限时使用
- 对于系统内部角色，`internal = true`，无 `domain`, `owner`, `at`，管理页面中不能删除
- 对于非系统内部角色，`name` 前必须包含前缀 `$$`

以下是系统内部角色：

- EVERYONE（所有人，含未登录用户）
- OWNER（资源所有者）
- DOMAIN_OWNER（域所有者）
- DOMAIN_MEMBER（域成员）

# 会话、登录

## Session

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | string         | session id  |
| expireat | date        | 过期时间      |
| data  | document       | 数据         |

### Index
- expireat ([TTLIndex](http://docs.mongodb.org/manual/tutorial/expire-data/#expire-documents-at-a-certain-clock-time))

## RememberMeToken

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| uid   | int64          | 用户 ID      |
| token | string         | Hash(token) |
| ua    | string         | 记忆时 User-Agent |
| ip    | string         | 记忆时 IP    |
| expireat | date        | 过期时间      |

其中，token 为 `hash(clientSideToken)`
clientSideToken 格式为 `{uid}|{expireTimestamp}|{clientToken}`

### Index

- uid, token
- expireat ([TTLIndex](http://docs.mongodb.org/manual/tutorial/expire-data/#expire-documents-at-a-certain-clock-time))

## LoginLog

| field | type           | description |
| ----- | -------------- | ----------- |
| _id   | mongoid        |             |
| uid   | int64          | 用户 ID      |
| at    | date           | 登录时间      |
| type  | string         | 登录类型      |
| ua    | string         | User-Agent  |
| ip    | string         | IP          |

### Index

- uid, at(desc)