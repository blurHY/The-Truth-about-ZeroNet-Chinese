# 创建数据库

今天咱建个博客站。

## 创建站点

我推荐直接创建新站点，因为更改一个写好的站点的数据库配置比较难调试。我还是会照常给你两种代码格式：一个用 ZeroFrame，另一个用我的库。

## 概览

零网内置 SQLite。当然，你自己也可以创建一个数据库，这不是很难。咱开始创建数据库吧。

## 数据源

零网中只有 _SELECT_ 语句是允许的，数据库是由 JSON 文件导入而成的，所以现在就写点 JSON 文件吧。

### `data.json`

依据传统，用户数据都放在 `data/*/data.json`文件。现在咱写个`data/admin/data.json`吧。创建一个数组`posts`，里面有几行数据。

```json
{
	"posts": [
		{
			"id": 1,
			"title": "Post 1",
			"content": "Post 1"
		},
		{
			"id": 2,
			"title": "Post 2",
			"content": "Post 2"
		}
	]
}
```

## 数据库模式

`dbschema.json` 是个简单的 JSON 文件，下面是个空数据库：

    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {},
        "tables": {}
    }

-   `db_name` 指数据库名，只用于调试。一个站点不能有多个数据库。
-   _SQLite_ 是单个文件，`db_file` 是其路径。
-   `version` 不是 _SQLite_ 版本，是零网的数据库的版本。
-   `maps` 数据库和文件之间的映射关系。
-   `tables` 表结构。

把 `dbschema.json` 复制到站点根目录。

### `json` 表

对于每个注册用户（站长或者是 ZeroID 用户），在`json`表里都有一条记录。零网需要这个表，否则会报错。

现在往文件里加点内容。

`dbschema.json`:

```json
{
	"db_name": "mydatabase",
	"db_file": "data/mydatabase.db",
	"version": 2,
	"maps": {},
	"tables": {
		"json": {
			"cols": [
				["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
				["directory", "TEXT"],
				["file_name", "TEXT"]
			],
			"indexes": [
				"CREATE UNIQUE INDEX path ON json(directory, file_name)"
			],
			"schema_changed": 1
		}
	}
}
```

### 添加表

再添加`posts`表

先设置表名

```json
    {
        "db_name": "mydatabase",
        "db_file": "data/mydatabase.db",
        "version": 2,
        "maps": {},
        "tables": {
            "json": {
                "cols": [
                    ["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
                    ["directory", "TEXT"],
                    ["file_name", "TEXT"]
                ],
                "indexes": ["CREATE UNIQUE INDEX path ON json(directory, file_name)"],
                "schema_changed": 1
            },
            "posts": {
                ...
            }
        }
    }
```

看 `json` 表的格式，每个表都由 `cols`, `indexes` 和 `schema_changed`组成。 `schema_changed` 就是表的版本，改变表后要增加这个值，这样每个节点才会重建数据库。

```json
    "posts": {
        "cols": [
            ...
        ],
        "indexes": [
            ...
        ],
        "schema_changed": 1
    }
```

`cols` 是列（column）的数组，每个列的第一个数组元素是列名 (SQL 关键字, 如`GROUP` 和 `UPDATE`是不允许的). 第二个是 _SQLite_ 列的类型 - `integer`, `float` 或 `text`.

```json
    "posts": {
        "cols": [
            ["id", "integer"],
            ["title", "text"],
            ["content", "text"],
            ["json_id", "integer references json(json_id)"]
        ],
        "indexes": [
            ...
        ],
        "schema_changed": 1
    }
```

注意到 `json_id` 列了吗? 这是 `dbschema.json` 中又一个奇怪的事情， 每个表都要有这个列。

`indexes`数组里列出了索引，`CREATE INDEX`可以放在那里，索引名并不代表什么。

```json
    "posts": {
        "cols": [
            ["id", "integer"],
            ["title", "text"],
            ["content", "text"],
            ["json_id", "integer references json(json_id)"]
        ],
        "indexes": [
            "CREATE UNIQUE INDEX post_id ON posts(id)"
        ],
        "schema_changed": 1
    }
```

现在我们有了一个数据库，签名 `content.json`.然后就会出来一个新文件 `data/mydatabase.db` (依据 `dbschema.json`里 `db_file` 指定的文件名). 这是 _SQLite_ 数据库 ，我推荐用 _SQLiteStudio_ 浏览、编辑数据库。现在你只能看到空的 `json` 和 `post` 表,已经一个特殊的 `keyvalue` 表，这是零网的内部表，不要动它。

> 译者注：_SQLiteStudio_ 太老了，用[DB Browser for SQLite ](http://sqlitebrowser.org/)吧。

### 映射表

#### `to_table`

现在只有空表，空表有啥用呢？用`data/admin/data.json`填充`posts`表吧。

还记得`dbschema.json`的 `maps` 对象吗 ?

```json
    ...
    "version": 2,
    "maps": {},
    "tables": {
        ...
```

改掉。

```json
    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts"
                }
            ]
        }
    }
```

键名 `admin/data.json` 是 **正则表达式**，符合此表达式的文件都会被导入到这个表。目前只有 `to_table`
，这意味着每个符合该正则表达式的 JSON 文件的`posts`对象都会被导入到数据库的`posts`表中。

签名 `content.json`，再查看一下数据库，看到变化了吗? 嗯，`posts`表中的`json_id`对应着`json`表中的`json_id` 。每个 json 文件都在`json`表中有记录，你可能觉得没有什么必要，但以后可能就会用到。

## 数据库的读权限

创建新文件 `js/index.js`，并在`index.html` 中引入它和`js/ZeroFrame.js`。

记得怎么用 ZeroFrame 吧? 调用 `dbQuery`:

    +-------------------------------------------------------------------------+
    |                                 dbQuery                                 |
    +-------------------------------------------------------------------------+
    | Run a query on the sql cache                                            |
    +-------------------------+-----------------------------------------------+
    | Parameter               | Description                                   |
    +-------------------------+-----------------------------------------------+
    | query                   | Sql query command                             |
    +-------------------------+-----------------------------------------------+
    | Return: Result of query as array or object with "error" property        |
    +-------------------------------------------------------------------------+

    var zeroFrame = new ZeroFrame();

    zeroFrame.cmd("dbQuery", ["SELECT * FROM posts"], function(posts) {
        if(posts.error) {
            console.warn(posts.error);
            return;
        }

        console.log(posts);
    });

打开控控制台并刷新，你就能看到:

    Array [ Object, Object ]
            Object {
                content: "Post 1",
                id: 1,
                json_id: 1,
                title: "Post 1"
            }

以后会讲如何动态编辑数据。至于现在，你手动编辑就是，给文件签名，然后刷新，看看有哪些变化。

We'll cover dynamic data changing and design in the following parts. For now, you can directly change `data/admin/data.json`, sign `content.json` and reload zite to see what changes.
