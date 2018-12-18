This tutorial has common tips and tricks for things that aren't readily apparent. Some of these will probably be expanded into full tutorials.

## Onpopstate won't work. What's going on?
Since zeronet displays all zites in an iframe, the onpopstate is actually using the iframe's address. Therefore, you cannot use onpopstate. But don't worry, there's a way around this. It turns out that the ZeroFrame API has a way of hooking into onpopstate. The wrapper passes the onpopstate event into the inner iframe and you can catch it using `onRequest`. Here's how I'm doing it on this zite (and in my ZeroFrame Router):

```javascript
onRequest (cmd, message) {
    if (cmd == "wrapperPopState") {
        if (message.params.state) {
            if (!message.params.state.url) {
                message.params.state.url = message.params.href.replace(/.*\?/, "");
            }

            // Don't worry too much about this, it's just a function
            // in my ZeroFrame Router to navigate to a route.
            this.navigate(message.params.state.url.replace(/^\//, ''));
        }
    }
}
```

## Can A Zite Owner Modify A User's data.json File?
Yes they can. All you have to do is modify the file, then manually put the path to the `content.json` file in the sidebar beside the `Sign` button and click `Sign` and `Publish`. Usually the path will be something like: `data/users/[user's id key folder]/content.json`.

## What Browsers Do I Need To Support For My Zite?
You only need to support browser versions that support Web Socket's because that is a requirement for ZeroNet. This means you *do not* have to support any version of IE *before* version 10. Here's a link showing browser support for Web Sockets: [https://caniuse.com/#search=websockets](https://caniuse.com/#search=websockets).

## AUTOINCREMENT Issue in Database
There is an interesting thing that happens with `AUTOINCREMENT`. Everytime you rebuild the database for your zite, any columns that use `AUTOINCREMENT` will not be reset back to 1, but will instead start at the next integer after the integer of the last row, which was deleted during the rebuild along with the rows before it. I'm unsure if this is intended or not. If you are manually updating your json file, a way around this is to just hardcode the integer's, incrementing them yourself. This is currently what I do for the tutorial id's on this zite.

## Reloading List When Database Changes
If you would like to reload a list, whose items were gotten from a database, whenever the database changes (for example, reloading comments when someone posts a new comment), all you need to do is add this to your `onRequest` function:

```javascript
if (message.params.event[0] == "file_done") {
    loadComments();
}
```

The `file done` message is sent whenever a json file was changed, the new version has been downloaded, and the database has reloaded/updated.

## Using A Data.json File That's Not User-specific
You can use a `data.json` file that only *you* can sign and isn't stored in a users folder. This is what I use to store the tutorials list for this zite. First, you need to make sure it isn't ignored in your `content.json` file. Next, create the file in the data folder. Below, this will be called `tutorials.json` since it stores tutorials. Then, in your `dbschema.json` file, add this:

In `maps` section:

```json
"tutorials.json": {
    "to_table": [
        {
            "node": "tutorials",
            "table": "tutorials"
        }
    ]
}
```

This tells ZeroNet that in the `tutorials.json` file, you have an array called `tutorials` whose data should be mapped into the `tutorials` table in the sqlite database.

In `tables` section:

```json
"tutorials": {
    "cols": [
        ["id", "INTEGER PRIMARY KEY"],
        ["slug", "TEXT NOT NULL"],
        ["file", "TEXT NOT NULL"],
        ["title", "TEXT NOT NULL"],
        ["description", "TEXT"],
        ["author", "TEXT NOT NULL"],
        ["tags", "TEXT"],
        ["json_id", "INTEGER REFERENCES json (json_id)"]
    ],
    "indexes": ["CREATE UNIQUE INDEX tutorial_key ON tutorials(id, json_id)", "CREATE UNIQUE INDEX tutorials_slug ON tutorials(slug)", "CREATE INDEX tutorial_file ON tutorials(slug)", "CREATE INDEX tutorial_title ON tutorials(title)"],
    "schema_changed": 12
}
```

This tells ZeroNet all of the columns and indexes of the `tutorials` table. The columns should directly relate to what's in your `data.json` file.

**Make sure you add `["json_id", "INTEGER REFERENCES json (json_id)"]` otherwise none of this will work.**

Whenever you change a table in your `dbschema.json` file, *make sure you update the `schema_changed` field* so ZeroNet knows to reload that table. Now all you need to do is press `Reload` and `Rebuild` on the sidebar on your zite.
