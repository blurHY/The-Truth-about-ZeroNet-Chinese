# 深入数据库模式

今天讲讲怎么改进 `dbschema.json`.

就用这个 `dbschema.json`:

```json
{
	"db_name": "mydatabase",
	"db_file": "data/mydatabase.db",
	"version": 2,
	"maps": {
		"admin/data.json": {
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
				["directory", "TEXT"],
				["file_name", "TEXT"]
			],
			"indexes": [
				"CREATE UNIQUE INDEX path ON json(directory, file_name)"
			],
			"schema_changed": 1
		},
		"posts": {
			"cols": [
				["id", "integer"],
				["content", "text"],
				["json_id", "integer references json(json_id)"]
			],
			"indexes": ["CREATE UNIQUE INDEX post_id ON posts(id)"],
			"schema_changed": 1
		}
	}
}
```

有一个含`id`和`content`字段的`posts`表，然后从`data/admin/data.json`的`posts`中导入数据。

## `key_col`

想一下哪些数据行是唯一的，比如投票数据——一对的 `id` 和 `author_id` 是唯一的。或者就直接是`json`表中的`id`。 数据是从 JSON 数组加载的，那怎么添加唯一字段呢，JSON 里也没有唯一数据? 哪还有?..

嗯，的确有，记得对象不? 键唯一，所以咱弄个唯一的字段，例如， Post ID。

在 `data/admin/data.json`, 我们把数组改为对象，把元素改为键值。

```json
{
	"posts": {
		"1": {
			"content": "Post 1"
		},
		"2": {
			"content": "Post 2"
		}
	}
}
```

在 `dbschema.json`, 添加 `key_col` 属性， 告诉把对象的键名存储在字段"id"中。

```json
    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts",
                    "key_col": "id"
                }
            ]
        }
    }
```

签名 `content.json`，打开 `data/mydatabase.db`.没什么变化， 但`id` 字段唯一 - 因为 JSON.

嗯, 实际上不是一直都唯一的，如果有两个一样的 `admin/data.json` 文件 (e.g. `data/a/admin/data.json` 和 `data/b/admin/data.json`)，可能有两个相同 ID。但站长不会这样做，因为有用户限制规则。

## `val_col`

只有一个属性的对象不够简洁，改 `data.json`成这样会好点:

```json
{
	"posts": {
		"1": "Post 1",
		"2": "Post 2"
	}
}
```

这样也是可以的，这就是`val_col`的魅力，先改`data/admin/data.json`，再改 `dbschema.json`:

```json
    "maps": {
        "admin/data.json": {
            "to_table": [
                {
                    "node": "posts",
                    "table": "posts",
                    "key_col": "id",
                    "val_col": "content"
                }
            ]
        }
    }
```

添加 `val_col` 属性，告诉零网把对象的属性值存储在 content 字段中。签名`content.json`，再看看有什么变化，你可以再加点文件看看。

这就是问什么我说几乎任何 JSON 结构都是可行的，从简单的`to_table`到更先进的`key_col` 和 `val_col`都行。

## `version`

看下 `json` 表. 目前只有三个字段: `json_id`, `directory` 和 `file_name`.这是因为 `version: 2`。 讲讲`version` 属性:

-   `version: 1`

    1. `json_id` - JSON 文件的 ID
    2. `path` - JSON 文件路径

    **问题**: 无

-   `version: 2`

    1. `json_id` - JSON 文件的 ID
    2. `directory` - JSON 文件的目录
    3. `file_name` - JSON 文件名

    **问题**:
    `directory` 是相对于数据库的路径，所以如果文件路径是`data/admin/data.json`，数据库路径是 `data/mydatabase.db`，则`directory` 就是 `admin`，所以同一目录下就不能有数据库。

-   `version: 3`

    1. `json_id` - JSON 文件的 ID
    2. `site` - JSON 文件所在站点
    3. `directory` - JSON 文件的目录
    4. `file_name` - JSON 文件名

    **问题**:
    同`version: 2`, 但 `site`就是 `directory`斜杠前的部分, 而 `directory` 就是第一个斜杠后的部分。所以在数据库的同一目录或者子目录不能有要导入的 JSON 文件，但子目录的子目录是可以的。
    你可能会问： "为什么是 `site`?"，那跟聚合站有关，待会再谈。
