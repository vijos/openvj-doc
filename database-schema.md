## User

| field | type           | description |
|--------------------------------------|
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

## RememberMeToken

| field | type           | description |
|--------------------------------------|
| _id   | mongoid        |             |
| uid   | int64          | 用户 ID      |
| token | string         | Hash(token) |
| ua    | string         | 记忆时 User-Agent |
| ip    | string         | 记忆时 IP    |
| expireat | date        | 过期时间      |

其中，token 为 `hash(clientSideToken)`
clientSideToken 格式为 `{uid}|{clientToken}`

### Index

- uid, token
- expireat