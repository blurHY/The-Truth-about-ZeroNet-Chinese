## 编辑聚合站点

前面一章，我们建了一个聚合者站点和被聚合站（hub），用于搭建 PostHere。

## 创建用户数据的目录

在被聚合站点（hub）中创建文件 `data/users/content.json` 

```json
{
    "ignore": ".*",
    "user_contents": {
        "cert_signers": {
            "zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]
        },
        "permission_rules": {
            ".*": {
                "files_allowed": "data.json",
                "max_size": 50000
            }
        },
        "permissions": {}
    }
}
```

-   `ignore` - 不把用户添加到 `files` 属性.。
-   `user_contents` - 由用户签名的数据。
-   `user_contents.cert_signers` - 允许的证书颁发者。
-   `user_contents.permission_rules` - 对符合某种条件的用户的规则。
-   `user_contents.permissions` - 针对单个用户的规则。

复制完后把这个文件引入至 `content.json`:

```json
"ignore": "data/users/.*",
"includes": {
    "data/users/content.json": {
        "signers": [],
        "signers_required": 1
    }
},
```

{==签名 content.json 和 data/users/content.json==}

## 数据库模式

我们使用 SQLite，把下面这段复制到主站点的 `dbschema.json` 

```json
{
    "db_name": "merger",
    "db_file": "merged-PostHere/merger.db",
    "version": 3,
    "maps": {
        ".+/data/users/.+/content.json": {
            "to_json_table": ["cert_user_id"]
        },
        ".+/data/users/.+/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts"
                }
            ]
        }
    },
    "tables": {
        "json": {
            "cols": [
                ["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
                ["site", "TEXT"],
                ["directory", "TEXT"],
                ["file_name", "TEXT"],
                ["cert_user_id", "TEXT"]
            ],
            "indexes": ["CREATE UNIQUE INDEX path ON json(directory, site, file_name)"],
            "schema_changed": 2
        },
        "posts": {
            "cols": [
                ["id", "integer"],
                ["title", "text"],
                ["content", "text"],
                ["date_added", "integer"],
                ["json_id", "integer references json(json_id)"]
            ],
            "indexes": [
                "CREATE UNIQUE INDEX post_id ON posts(id)"
            ],
            "schema_changed": 2
        }
    }
}
```

-   `version: 3` 意为 `json` 表也有 `site` 字段。对于聚合者站点， `site` 就是被聚合站点的地址，`merged-PostHere` 也可以用。
-   `.+/data/users/.+/content.json` 对象中的 `to_json_table` ： `json` 也有 `cert_user_id` 字段。 `to_json_table` 意为： “使用当前文件的 `cert_user_id` 属性作为 `json` 表中的当前文件对应的一行的`cert_user_id`字段的值”. `cert_user_id` 就是用户名。

### 获取用户名

示例：

    +-------------------------------------------------------------------------+
    |                                  json                                   |
    +---------+---------+-------------------+--------------+------------------+
    | json_id | site    | directory         | file_name    | cert_user_id     |
    +---------+---------+-------------------+--------------+------------------+
    | 1       | 1Red... | data/users/1Cv... | data.json    | NULL             |
    +---------+---------+-------------------+--------------+------------------+
    | 2       | 1Red... | data/users/1CV... | content.json | ivanq@zeroid.bit |
    +---------+---------+-------------------+--------------+------------------+
    +-------------------------------------------------------------------------+
    |                                  posts                                  |
    +---------+-------------+--------------------------+------------+---------+
    | id      | title       | content                  | date_added | json_id |
    +---------+-------------+--------------------------+------------+---------+
    | 1       | Hello world | My first post in this... | 1234567890 | 1       |
    +---------+-------------+--------------------------+------------+---------+

为了获得文章作者名，对于每篇文章，都要先获取相应的`json_id`，再于`json`表按此id中取得`directory`字段，然后再在`json`表中找一条同一`directory`且`file_name` = `content.json`的数据，最终取得`cert_user_id`。

数据库查询语句：

```sql
SELECT posts.*, json2.cert_user_id as username FROM posts, json, json AS json2 WHERE json.directory = json2.directory AND json2.file_name = "content.json" AND posts.json_id = json.json_id
```

当然还有另一种方法：

```json
    ".+/data/users/.+/content.json": {
        "to_json_table": ["cert_user_id"],
        "file_name": "data.json"
    },
```

`file_name` 相当于 `JOIN`。对于每个用户的 `content.json`, 零网会找一条 `site` 和 `directory` 属性相同且 `file_name` = `data.json` 的数据，并把`cert_user_id` 添加到这行数据。结构就变成了：

    +-------------------------------------------------------------------------+
    |                                  json                                   |
    +---------+---------+-------------------+--------------+------------------+
    | json_id | site    | directory         | file_name    | cert_user_id     |
    +---------+---------+-------------------+--------------+------------------+
    | 1       | 1Red... | data/users/1Cv... | data.json    | ivanq@zeroid.bit |
    +---------+---------+-------------------+--------------+------------------+
    +-------------------------------------------------------------------------+
    |                                  posts                                  |
    +---------+-------------+--------------------------+------------+---------+
    | id      | title       | content                  | date_added | json_id |
    +---------+-------------+--------------------------+------------+---------+
    | 1       | Hello world | My first post in this... | 1234567890 | 1       |
    +---------+-------------+--------------------------+------------+---------+

可以简化查询语句了：

```sql
SELECT posts.*, json.cert_user_id AS username FROM posts, json WHERE posts.json_id = json.json_id
```

别忘了，零网在聚合者站点中使用虚拟目录，所以可以这样改文件 `merged-PostHere/{hub}/data/users/{address}/data.json`.

## 增删子站

用API `mergerSiteList` 列出已聚合的子站。

    +-------------------------------------------------------------------------+
    |                             mergerSiteList                              |
    +-------------------------------------------------------------------------+
    | Return merged sites.                                                    |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | query_site_info         | If True, then gives back detailed site info   |
    |                         | for merged sites                              |
    +-------------------------+-----------------------------------------------+
    | Return: List of merger sites as object                                  |
    +-------------------------------------------------------------------------+

试着运行 `zeroFrame.cmd("mergerSiteList", [false], console.log.bind(console));`，可以在控制台看到 `Object { 1RedXn7jxM23y4WsR7ByWzhjFaCcBJwVQ: "PostHere" }` 。

以及其他常用命令:

    +-------------------------------------------------------------------------+
    |                              mergerSiteAdd                              |
    +-------------------------------------------------------------------------+
    | Start downloading new merger site (requires confirmation if called      |
    | twice in 10 seconds)                                                    |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | addresses               | Site address or list of site addresses        |
    +-------------------------+-----------------------------------------------+
    | Return: Always "ok" (even before site is added)                         |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    |                            mergerSiteDelete                             |
    +-------------------------------------------------------------------------+
    | Stop seeding and delete a merged site                                   |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | address                 | Site address                                  |
    +-------------------------+-----------------------------------------------+
    | Return: "ok" or object with "error" property                            |
    +-------------------------------------------------------------------------+

## 例子

已完成的站点： [这](downloads/posthere.html) 和 [这](downloads/postherehub.html).
