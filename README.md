# ruoyi-eggjs-mysql

> Egg plugin for mysql

基于 [mysql2](https://github.com/sidorares/node-mysql2) 的 Egg.js MySQL 插件，提供简单易用的数据库操作接口和连接池管理。

## 特性

- ✅ 基于 mysql2 连接池，性能优异
- ✅ 支持单实例和多实例配置
- ✅ 提供简洁的 API 封装（select、insert、update、delete）
- ✅ 内置事务支持，自动提交和回滚
- ✅ 开发环境自动打印 SQL 执行时间
- ✅ 错误信息包含执行的 SQL 语句
- ✅ 原生 Promise/Async 支持

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

### 单实例

```js
// {app_root}/config/config.default.js
config.mysql = {
  default: {
    port: 3306,
    charset: "utf8mb4",
    multipleStatements: true,  // 允许执行多条 SQL
    connectionLimit: 100,       // 连接池最大连接数
  },
  client: {
    host: "127.0.0.1",
    user: "root",
    password: "your_password",
    database: "your_database",
  },
};
```

### 多实例

```js
// {app_root}/config/config.default.js
config.mysql = {
  default: {
    port: 3306,
    charset: "utf8mb4",
    multipleStatements: true,
    connectionLimit: 100,
  },
  clients: {
    // 主库
    db1: {
      host: "127.0.0.1",
      user: "root",
      password: "password1",
      database: "database1",
    },
    // 从库
    db2: {
      host: "192.168.1.100",
      user: "root",
      password: "password2",
      database: "database2",
    },
  },
};
```

### 配置参数说明

| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| host | String | - | MySQL 服务器地址 |
| port | Number | 3306 | MySQL 端口 |
| user | String | - | 数据库用户名 |
| password | String | - | 数据库密码 |
| database | String | - | 数据库名称 |
| charset | String | utf8mb4 | 字符集（推荐 utf8mb4 支持 emoji） |
| connectionLimit | Number | 100 | 连接池最大连接数 |
| multipleStatements | Boolean | true | 是否允许一次执行多条 SQL |
| timezone | String | +08:00 | 时区设置 |

更多配置选项请参考 [mysql2 文档](https://github.com/sidorares/node-mysql2#first-query)。

## 使用方法

### 单实例

```js
// 在 controller 或 service 中使用
const { app } = this;

// 单条查询
const user = await app.mysql.select('SELECT * FROM users WHERE id = 1');

// 多条查询
const users = await app.mysql.selects('SELECT * FROM users WHERE age > 18');

// 插入数据（返回新插入行的 ID）
const insertId = await app.mysql.insert("INSERT INTO users (name, age) VALUES ('张三', 25)");

// 更新数据（返回影响的行数）
const affectedRows = await app.mysql.update("UPDATE users SET age = 26 WHERE id = 1");

// 删除数据（返回影响的行数）
const deleted = await app.mysql.del("DELETE FROM users WHERE id = 1");

// 执行任意 SQL
await app.mysql.run("CREATE TABLE IF NOT EXISTS users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(255))");
```

### 多实例

```js
// 获取指定数据库实例
const db1 = app.mysql.get('db1');
const db2 = app.mysql.get('db2');

// 从不同数据库查询
const user = await db1.select('SELECT * FROM users WHERE id = 1');
const order = await db2.select('SELECT * FROM orders WHERE id = 1');
```

## API 说明

### select(sql)

执行单条查询，返回第一行数据。

```js
const user = await app.mysql.select('SELECT * FROM users WHERE id = 1');
// 返回: { id: 1, name: '张三', age: 25 } 或 null
```

### selects(sql)

执行多条查询，返回所有匹配的行。

```js
const users = await app.mysql.selects('SELECT * FROM users WHERE age > 18');
// 返回: [{ id: 1, name: '张三', age: 25 }, { id: 2, name: '李四', age: 30 }]
```

### insert(sql)

执行插入操作，返回新插入行的 `insertId`。

```js
const insertId = await app.mysql.insert("INSERT INTO users (name, age) VALUES ('王五', 28)");
// 返回: 3 (新插入行的自增 ID)
```

### update(sql)

执行更新操作，返回受影响的行数。

```js
const affectedRows = await app.mysql.update("UPDATE users SET age = 26 WHERE id = 1");
// 返回: 1 (受影响的行数)
```

### del(sql)

执行删除操作，返回受影响的行数（实际是 `update` 的别名）。

```js
const deleted = await app.mysql.del("DELETE FROM users WHERE age < 18");
// 返回: 2 (删除的行数)
```

### run(sql)

执行任意 SQL 语句，返回完整的执行结果。

```js
const [results, fields] = await app.mysql.run("SHOW TABLES");
// results: 查询结果数组
// fields: 字段信息数组
```

### transaction(sqls)

执行事务，传入 SQL 数组，全部成功则自动提交，任一失败则自动回滚。

```js
const results = await app.mysql.transaction([
  "INSERT INTO users (name, age) VALUES ('张三', 25)",
  "INSERT INTO users (name, age) VALUES ('李四', 30)",
  "UPDATE accounts SET balance = balance - 100 WHERE user_id = 1",
  "UPDATE accounts SET balance = balance + 100 WHERE user_id = 2",
]);
// 返回: 所有 SQL 的执行结果数组
```

如果事务中任何一条 SQL 执行失败，所有更改会自动回滚：

```js
try {
  await app.mysql.transaction([
    "INSERT INTO users (name, age) VALUES ('张三', 25)",
    "INSERT INTO invalid_table (name) VALUES ('test')", // 这条会失败
  ]);
} catch (error) {
  console.log(error.sqls); // 包含所有执行的 SQL
  // 第一条插入会被自动回滚
}
```

### pool

获取原始的 mysql2 连接池对象，用于高级操作。

```js
const pool = app.mysql.pool;
const [rows, fields] = await pool.query('SELECT * FROM users WHERE id = ?', [1]);
```

## 开发调试

在非生产环境下，插件会自动在控制台打印每条 SQL 的执行时间：

```
SELECT * FROM users WHERE id = 1: 1.234ms
INSERT INTO users (name, age) VALUES ('张三', 25): 2.567ms
```

## 完整示例

### Service 层使用

```js
// app/service/user.js
const { Service } = require('egg');

class UserService extends Service {
  async create(name, age) {
    const insertId = await this.app.mysql.insert(
      `INSERT INTO users (name, age, created_at) VALUES ('${name}', ${age}, NOW())`
    );
    return insertId;
  }

  async findById(id) {
    return await this.app.mysql.select(
      `SELECT * FROM users WHERE id = ${id}`
    );
  }

  async findAll() {
    return await this.app.mysql.selects('SELECT * FROM users ORDER BY id DESC');
  }

  async update(id, data) {
    const affectedRows = await this.app.mysql.update(
      `UPDATE users SET name = '${data.name}', age = ${data.age} WHERE id = ${id}`
    );
    return affectedRows > 0;
  }

  async delete(id) {
    const deleted = await this.app.mysql.del(`DELETE FROM users WHERE id = ${id}`);
    return deleted > 0;
  }

  // 转账示例（事务）
  async transfer(fromUserId, toUserId, amount) {
    return await this.app.mysql.transaction([
      `UPDATE accounts SET balance = balance - ${amount} WHERE user_id = ${fromUserId}`,
      `UPDATE accounts SET balance = balance + ${amount} WHERE user_id = ${toUserId}`,
      `INSERT INTO transactions (from_user, to_user, amount, created_at) 
       VALUES (${fromUserId}, ${toUserId}, ${amount}, NOW())`,
    ]);
  }
}

module.exports = UserService;
```

### 使用参数化查询（推荐）

为了防止 SQL 注入，推荐使用参数化查询：

```js
// 使用 pool 进行参数化查询
const pool = app.mysql.pool;

// 查询
const [users] = await pool.query('SELECT * FROM users WHERE age > ?', [18]);

// 插入
const [result] = await pool.query(
  'INSERT INTO users (name, age) VALUES (?, ?)',
  ['张三', 25]
);
const insertId = result.insertId;

// 更新
const [updateResult] = await pool.query(
  'UPDATE users SET age = ? WHERE id = ?',
  [26, 1]
);
const affectedRows = updateResult.affectedRows;
```

### 多数据库操作

```js
// app/service/sync.js
class SyncService extends Service {
  async syncUserData(userId) {
    const db1 = this.app.mysql.get('db1'); // 主库
    const db2 = this.app.mysql.get('db2'); // 从库

    // 从主库读取用户数据
    const user = await db1.select(`SELECT * FROM users WHERE id = ${userId}`);

    if (user) {
      // 同步到从库
      await db2.insert(
        `INSERT INTO users (id, name, age) VALUES (${user.id}, '${user.name}', ${user.age})
         ON DUPLICATE KEY UPDATE name = '${user.name}', age = ${user.age}`
      );
    }

    return user;
  }
}
```

## 注意事项

1. **SQL 注入防护**：示例中为了简洁使用了字符串拼接，生产环境强烈建议使用参数化查询（通过 `pool.query()` 并传入参数数组）

2. **连接池管理**：插件自动管理连接池，无需手动释放连接（除非使用 `transaction` 或直接操作 `pool`）

3. **字符集设置**：推荐使用 `utf8mb4` 字符集以支持 emoji 等特殊字符

4. **时区问题**：如需设置时区，在配置中添加 `timezone: '+08:00'`

5. **事务使用**：事务会占用一个独立连接直到提交或回滚，注意连接池大小设置

6. **错误处理**：所有方法都会抛出异常，建议使用 try-catch 捕获

```js
try {
  await app.mysql.insert("INSERT INTO users (name) VALUES ('test')");
} catch (error) {
  console.error('执行失败的 SQL:', error.sql);
  console.error('错误信息:', error.message);
}
```

## 性能优化建议

1. **合理设置连接池大小**：根据并发量调整 `connectionLimit`
2. **使用索引**：确保查询字段有适当的索引
3. **避免 SELECT ***：明确指定需要的字段
4. **批量操作**：使用事务进行批量插入/更新
5. **读写分离**：使用多实例配置实现主从分离

## 相关链接

- [mysql2 官方文档](https://github.com/sidorares/node-mysql2)
- [MySQL 官方文档](https://dev.mysql.com/doc/)
- [连接池最佳实践](https://github.com/sidorares/node-mysql2#using-connection-pools)

---

## 关于 ruoyi-eggjs 项目

本插件是 [ruoyi-eggjs](https://github.com/undsky/ruoyi-eggjs) 项目的核心组件之一。

**ruoyi-eggjs** 是一个基于 Egg.js 的企业级后台管理系统，参照若依（RuoYi）架构设计，提供完善的权限管理、用户管理、系统监控等功能，是快速开发企业级应用的最佳选择。

### 主要特性

- 🎯 **完整的权限系统**：基于 RBAC 的权限控制，支持细粒度权限管理
- 🚀 **开箱即用**：集成常用功能模块，快速启动项目开发
- 🔧 **MyBatis 风格**：采用 XML 风格的 SQL 编写，熟悉的开发体验
- 📦 **模块化设计**：松耦合的插件体系，按需使用
- 🛡️ **企业级安全**：XSS 防护、SQL 注入防护、访问控制等
- 📊 **系统监控**：在线用户、登录日志、操作日志、定时任务等

### 项目地址

- GitHub: [https://github.com/undsky/ruoyi-eggjs](https://github.com/undsky/ruoyi-eggjs)
- Gitee: [https://gitee.com/undsky/ruoyi-eggjs](https://gitee.com/undsky/ruoyi-eggjs)

### 相关插件

- [ruoyi-eggjs-cache](https://github.com/undsky/ruoyi-eggjs-cache) - 缓存插件
- [ruoyi-eggjs-mybatis](https://github.com/undsky/ruoyi-eggjs-mybatis) - MyBatis 集成
- [ruoyi-eggjs-mysql](https://github.com/undsky/ruoyi-eggjs-mysql) - MySQL 连接
- [ruoyi-eggjs-ratelimiter](https://github.com/undsky/ruoyi-eggjs-ratelimiter) - 限流插件
- [ruoyi-eggjs-sqlite](https://github.com/undsky/ruoyi-eggjs-sqlite) - SQLite 支持
- [ruoyi-eggjs-handlebars](https://github.com/undsky/ruoyi-eggjs-handlebars) - Handlebars 模板

### 联系方式

- 📮 **Issues**: [提交问题或建议](https://github.com/undsky/ruoyi-eggjs/issues)
- 🌐 **官网**: [https://www.undsky.com](https://www.undsky.com)
- 💬 **讨论**: [GitHub Discussions](https://github.com/undsky/ruoyi-eggjs/discussions)

### 贡献指南

欢迎提交 Issue 和 Pull Request！

如果这个项目对你有帮助，请给我们一个 ⭐️ Star 支持一下！

---

## License

[MIT](LICENSE)
