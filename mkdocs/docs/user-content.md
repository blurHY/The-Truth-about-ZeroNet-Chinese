# 用户数据

你可能已经访问过了 _ZeroMe_ 或 _ZeroTalk_ - 一个社交网络和论坛，都使用 _SQLite_ - 以前说过。

之前创建了一个站长发布内容的站点，记得吧，没有投票功能，今天我们会改进这个东西。

首先创建新站点。

## 包含`content.json`

我们有目录 `data/users`，这个目录会有很多子目录 (目录名为用户的公钥) 内含用户文件.

首先创建 `content.json` 在 `data/users/content.json` 用于存储用户数据 (签名时零网会自动添加必要的对象属性):

    {
        "files": {}
    }

包含入`content.json`:

    ...,
    "ignore": "data/users/.*",
    "includes": {
        "data/users/content.json": {
            "signers": [],
            "signers_required": 1
        }
    },
    ...

第一行的`ignore`叫零网忽略指定目录下的文件。
其余的内容意为`data/users/content.json`也应加载。

签名 `content.json` 和 `data/users/content.json`.

## 创建正确的 `content.json`

用 _ZeroID_. 先思考一下: 节点如何得知哪些节点能给哪些目录签名? _ZeroID_ 和 其他 **证书颁发者** 证明某用户可以签名某文件。

编辑 `data/users/content.json`.

    {
        "ignore": ".*",
        "user_contents": {
            "cert_signers": {
                "zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]
            },
            "permission_rules": {
                ".*": {
                    "files_allowed": "data.json",
                    "max_size": 10000
                }
            },
            "permissions": {
                "ivanq@zeroid.bit": {
                    "max_size": 100000
                }
            }
        }
    }

-   `ignore` 意为这里面的文件不是网站管理员签名的。
-   `user_contents.cert_signers` 是 **证书颁发者** 列表. 这有 `zeroid.bit` 和 `1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz` 证明某用户能签名某目录。
-   `user_contents.permission_rules` 按正则表达式规定哪些用户可以干什么、不能干什么。那个键名 (`.*`) 意为 "将下列规则应用于所有用户".要匹配这个正则表达式的字符串格式为 `{authtype}/{username}@{zite}`. `authtype` 由 **证书颁发者** 设置 (通常是 `web` 或 `bitmsg`). `files_allowed` 是规定用户可以存储哪些的文件的正则表达式。 `max_size` 是文件的最大大小（字节）。
-   `user_contents.permissions` 对单一用户作规定，键名为 `{username}@{zite}`. 值同 `permission_rules`.

这里允许所有人存储 10KB，允许 `ivanq@zeroid.bit` 100KB(这是我).

## 创建 `dbschema.json`

在`dbschema.json`创建一些表:

    +-------------------------------------------------------------------------+
    |                                questions                                |
    +-------------------------+-----------------------------------------------+
    | Row                     | Type                                          |
    +-------------------------+-----------------------------------------------+
    | id                      | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | question                | TEXT                                          |
    +-------------------------+-----------------------------------------------+
    | answers                 | TEXT                                          |
    +-------------------------+-----------------------------------------------+
    | date_added              | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | json_id                 | INTEGER REFERENCES json(json_id)              |
    +-------------------------------------------------------------------------+

    +-------------------------------------------------------------------------+
    |                                 answers                                 |
    +-------------------------+-----------------------------------------------+
    | Row                     | Type                                          |
    +-------------------------+-----------------------------------------------+
    | question_id             | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | answer_id               | INTEGER                                       |
    +-------------------------+-----------------------------------------------+
    | json_id                 | INTEGER REFERENCES json(json_id)              |
    +-------------------------------------------------------------------------+

`dbschema.json`:

```json
{
	"db_name": "votes",
	"db_file": "data/votes.db",
	"version": 2,
	"maps": {
		"users/.*/data.json": {
			"to_table": [
				{
					"node": "questions",
					"table": "questions"
				},
				{
					"node": "answers",
					"table": "answers",
					"key_col": "question_id",
					"val_col": "answer_id"
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
		"questions": {
			"cols": [
				["id", "integer"],
				["question", "text"],
				["answers", "text"],
				["date_added", "integer"],
				["json_id", "integer references json(json_id)"]
			],
			"indexes": ["CREATE UNIQUE INDEX question_id ON questions(id)"],
			"schema_changed": 1
		},
		"answers": {
			"cols": [
				["question_id", "integer"],
				["answer_id", "integer"],
				["json_id", "integer references json(json_id)"]
			],
			"indexes": [
				"CREATE UNIQUE INDEX answer_value ON answers(question_id, json_id)"
			],
			"schema_changed": 1
		}
	}
}
```

只增加了`users/.*/data.json`，意为: 读取`data/users/{something}/data.json`的所有文件并按下列方法处理: `...`.

## 以 ZeroID 授权

之前说过，要用 ZeroID。现在实现它。

先创建 `js/index.js` ，并在其他文件之前引入。

    window.zeroFrame = new ZeroFrame();

创建文件, `js/zeroid.js` 并引入到 `index.html`:

```js
function authAsZeroID(callback) {
	zeroFrame.cmd("siteInfo", [], function(siteInfo) {
		// If logged in, return object with username and public key (address)
		if (siteInfo.cert_user_id) {
			callback({
				user: siteInfo.cert_user_id,
				address: siteInfo.auth_address,
			});

			return;
		}

		// Open authorization window and allow zeroid.bit
		zeroFrame.cmd(
			"certSelect",
			{
				accepted_domains: ["zeroid.bit"],
			},
			function() {
				zeroFrame.cmd("siteInfo", [], function(siteInfo) {
					// If logged in, return object with username and public key (address),
					// else return false
					if (siteInfo.cert_user_id) {
						callback({
							user: siteInfo.cert_user_id,
							address: siteInfo.auth_address,
						});
					} else {
						callback(false);
					}
				});
			}
		);
	});
}
```

尝试在控制台输入 `authAsZeroID(console.log.bind(console))`，就会弹出一个窗口 ，如果你选择 ZeroID, 一个有 `user` 和 `address` 属性的对象就会输出到控制台，如果选 `Unique to this site`, `false` 就会输出到控制台。

后面几章讲如何读写用户数据。
