# ruoyi-eggjs-mysql

> Egg plugin for mysql

## 安装

```bash
$ npm i ruoyi-eggjs-mysql --save
```

## 支持的 egg 版本

| egg 3.x | egg 2.x | egg 1.x |
| ------- | ------- | ------- |
| 😁      | 😁      | ❌      |

## 开启插件

```js
// {app_root}/config/plugin.js
exports.mysql = {
  enable: true,
  package: "ruoyi-eggjs-mysql",
};
```

## 配置

```js
// {app_root}/config/config.default.js
config.mysql = {
  default: {
    port: 3306,
    charset: "utf8mb4",
    multipleStatements: true,
    connectionLimit: 100,
  },
  // 单实例
  client: {
    host: "host",
    user: "username",
    password: "password",
    database: "dbname",
  },
  // 多实例
  clients: {
    db1: {
      host: "host",
      user: "username",
      password: "password",
      database: "dbname",
    },
  },
};
```

## 示例

```js
const db1 = app.mysql.get("db1");
// 获取数据库连接池对象
const pool = db1.pool;
// 运行 SQL
await db1.run(sql);
// 单条查询
await db1.select(sql);
// 多条查询
await db1.selects(sql);
// 插入
await db1.insert(sql);
// 更新
await db1.update(sql);
// 删除
await db1.del(sql);
// 简单事务
await db1.transaction([sql1, sql2, sql2]);
```

## License

[MIT](LICENSE)
