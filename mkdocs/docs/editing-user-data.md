# 编辑用户数据

试一下编辑用户数据文件，做一个投票系统。

## `data.json` 文件

写一个用于添加问题的函数

首先把 `js/files.js` 引入 `index.html`:

    function readFile(file, callback) {
        zeroFrame.cmd("fileGet", [file, false], callback);
    }

    function writeFile(file, content, callback) {
        zeroFrame.cmd("fileWrite", [file, base64Encode(content)], callback);
    }

    function base64Encode(content) {
        content = encodeURIComponent(content); // Split to bytes in % notation
        content = unescape(content); // Join % notation as bytes (not as chars)
        return btoa(content);
    }

写 `addQuestion()` 函数，保存到 `js/votes.js`:

    function addQuestion(question, answers, callback) {
        ...
    }

如果某人想添加问题，首先得有 ZeroID，并且他授权本站使用。

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }

            ...
        });
    }

打开用户的 JSON：

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }

            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";

                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }

                ...
            });
        });
    }

注意我们尝试使用用户地址打开目录，即用户的公钥，如果用户用 KaffieID，公钥个用户名(`user`)就不一样。

添加问题，保持 JSON，发布 `content.json`:

    function addQuestion(question, answers, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }

            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";

                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }

                var id = content.next_question_id;
                content.questions.push({
                    id: id++,
                    question: question,
                    answers: answers.join("\n"),
                    date_added: Math.floor(Date.now() / 1000)
                });

                content = JSON.stringify(content);

                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, function() {
                        callback(id);
                    });
                });
            });
        });
    }

不需要传递 `privatekey` 参数到 `sitePublish` 命令，因为 `privatekey: null` 意为 "用用户私钥签名".

还要签名 `data/users/{address}/content.json`,而文件可能不存在 (比如用户第一次创建文件)，还好，如果文件不存在，零网会自动创建。

检验下代码，打开开发工具，刷新页面，输入下列文本。

    addQuestion("What's nofish's name?", ["nofish", "Tomas", "Jack"], console.log.bind(console));

授权站点使用 ZeroID，并关注 `data/votes.db` 的变化。

## 答案

尝试自己编写 `addAnswer()` 。

答案:

    function addQuestion(questionId, answerId, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }

            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";

                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }

                content.answers[questionId] = answerId;

                content = JSON.stringify(content);

                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, callback);
                });
            });
        });
    }

注意我们有相似的函数，就写个库吧：

    function editUserData(handler, callback) {
        authAsZeroID(function(user) {
            // User rejected to authorizate
            if(!user) {
                callback(false);
                return;
            }

            readFile("data/users/" + user.address + "/data.json", function(content) {
                content = content || "";

                // Parse JSON
                try {
                    content = JSON.parse(content);
                } catch(e) {
                    content = {
                        questions: [],
                        answers: {},
                        next_question_id: 0
                    };
                }

                handler(content);

                content = JSON.stringify(content);

                writeFile("data/users/" + user.address + "/data.json", content, function() {
                    zeroFrame.cmd("sitePublish", {
                        inner_path: "data/users/" + user.address + "/content.json"
                    }, callback);
                });
            });
        });
    }

    function addQuestion(question, answers, callback) {
        var id;
        editUserData(function(content) {
            id = content.next_question_id;

            content.questions.push({
                id: content.next_question_id++,
                question: question,
                answers: answers.join("\n"),
                date_added: Math.floor(Date.now() / 1000)
            });
        }, function() {
            callback(id);
        });
    }


    function addAnswer(questionId, answerId, callback) {
        editUserData(function(content) {
            content.answers[questionId] = answerId;
        }, callback);
    }

## 读取数据

编写 `getQuestionList()`, `getQuestion()` 和 `getAnswers()` 函数:

    function getQuestionList(sort, callback) {
        if(sort == "popular") {
            zeroFrame.cmd("dbQuery", ["SELECT questions.*, CASE WHEN answers.answer_count IS NULL THEN 0 ELSE answers.answer_count END AS answer_count FROM questions LEFT JOIN (SELECT question_id, COUNT(*) as answer_count FROM answers GROUP BY question_id) AS answers ON (answers.question_id = questions.id) ORDER BY answers.answer_count DESC, questions.date_added DESC LIMIT 0, 10"], callback);
        } else if(sort == "latest") {
            zeroFrame.cmd("dbQuery", ["SELECT * FROM questions ORDER BY date_added DESC LIMIT 0, 10"], callback);
        }
    }

    function getQuestion(id, callback) {
        zeroFrame.cmd("dbQuery", ["SELECT * FROM questions WHERE id = " + id], function(questions) {
            zeroFrame.cmd("siteInfo", [], function(siteInfo) {
                if(siteInfo.cert_user_id) { // User logged in
                    zeroFrame.cmd("dbQuery", ["SELECT answers.*, json.* FROM answers, json WHERE json.directory = \"users/" + siteInfo.auth_address + "\" AND answers.json_id = json.json_id AND answers.question_id = " + id], function(answer) {
                        if(answer.length) {
                            questions[0].answered = answer[0].answer_id;

                            getAnswers(id, function(answers) {
                                questions[0].answers = answers;
                                callback(questions[0]);
                            });
                        } else {
                            questions[0].answered = -1;
                            callback(questions[0]);
                        }
                    });
                } else {
                    questions[0].answered = -1;
                    callback(questions[0]);
                }
            });
        });
    }

    function getAnswers(id, callback) {
        zeroFrame.cmd("dbQuery", ["SELECT answer_id, COUNT(*) as answer_count FROM answers WHERE question_id = " + id + " GROUP BY answer_id"], function(answers) {
            var result = {};
            for(var i = 0; i < answers.length; i++) {
                result[answers[i].answer_id] = answers[i].answer_count;
            }
            callback(result);
        });
    }

核心代码写好了，只差设计网页, [这](downloads/voting.html)是个例子。
