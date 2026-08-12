# Week 5: Introduction to Databases

**Date:** 8 & 10 July 2026

## What we're covering this week

* Recap + Complete Week 4 Handout
* [Why Databases?](#why-databases)
* [Relational Databases](#relational-databases)
* [Schema](#schema)
* [SQL and SQLite](#sql-and-sqlite)
* [The four operations (CRUD)](#the-four-operations-crud)
* [Exercise: design your database](#exercise-design-your-database)
* [Creating a table](#creating-a-table)
* [The main SQL commands](#the-main-sql-commands)
* [Talking to SQLite from Python](#talking-to-sqlite-from-python)
* [Seed data](#seed-data)
* [Exercise: fetch real data into your cards](#exercise-fetch-real-data-into-your-cards)


## Why Databases?

You could store your items in a text file or a JSON file. But it faces limitations when it comes down to having a live app used by multiple users at the same time.

Concretely, a database gives you:
* **Fast lookups.** Find one item by its id without scanning everything.
* **Structure.** Every item has the same fields, enforced.
* **Safe concurrent access.** Two writes at once do not corrupt your data.
* **Querying.** Ask questions like "all trade items in the electronics category" in one line.
* **Persistence.** Data survives a server restart or crash.


## Relational Databases

### What "relational" means
The name comes from the mathematical idea of a **relation**, which is just a formal word for a table: a set of rows, where every row has the same columns.

So a relational database is one that organises data into **tables** (relations), each made of:

* **Columns** (also called fields or attributes): the kinds of information you store, such as `title` or `category`.
* **Rows** (also called records): one entry, such as one specific lost water bottle.

Think of a single table as a spreadsheet. Column headers across the top, one item per row underneath.

| id | title | category | status |
|----|-------|----------|--------|
| 1 | Blue Water Bottle | lost | available |
| 2 | Casio Calculator | found | available |

### Relationships

Once you have more than one table, you can express how their rows connect. There are three shapes:

**One to one.** One row in table A matches exactly one row in table B. Example: a user and their single profile settings row.

**One to many.** One row in table A matches many rows in table B. This is the common one. Example: **one user posts many items**. Each item belongs to exactly one user, but a user can post any number of items.

**Many to many.** Rows on both sides can match many rows on the other. Example: items and tags, where an item can have many tags and a tag can apply to many items. These need a third linking table.

* Reference: https://cdn.cs50.net/sql/2023/x/lectures/1/lecture1.pdf

> **In your app, what are some relationships that exist?**


## Schema

A **schema** is the blueprint of your database. It is the full description of what tables exist, what columns each table has, what type each column holds, and what rules apply.

A schema for one table answers four questions per column:

1. What is the column called? (`title`)
2. What type of data does it hold? (`TEXT`)
3. Can it be empty? (`NOT NULL` means no)
4. Does it have a default? (`DEFAULT 'available'`)


## SQL and SQLite

**SQL** (Structured Query Language) is the language you use to talk to a relational database. You write SQL statements like `SELECT * FROM items` to ask questions and make changes. Almost every relational database understands SQL, so this skill transfers directly to bigger systems later.

**SQLite** is a specific database engine. Its selling point is that the entire database is a single file on disk (for example `lostfound.db`), with no separate server to install or run. That makes it perfect for learning and for a project like this. The same SQL you write here works, with tiny differences, on the big engines used in production.

Python ships with a built-in `sqlite3` module. You do not `pip install` anything to use SQLite from Python. Let's just confirm it works.

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Confirm the sqlite3 module works | `python -c "import sqlite3; print(sqlite3.sqlite_version)"` | `python3 -c "import sqlite3; print(sqlite3.sqlite_version)"` |

If that prints a version number (for example `3.45.1`), you are ready.

### Install DB Browser for SQLite (recommended)

To _see_ your data with your own eyes rather than only through code, install **DB Browser for SQLite**, a free visual tool. This is the "open the filing cabinet and look inside" step.

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Install the GUI | Download the installer from [sqlitebrowser.org](https://sqlitebrowser.org/dl/) and run it | Download the `.dmg` from [sqlitebrowser.org](https://sqlitebrowser.org/dl/), **or** run `brew install --cask db-browser-for-sqlite` |


## The four operations (CRUD)

Almost everything you ever do to stored data is one of four operations. Together they spell **CRUD**, and each maps directly to an action in your app.

| Operation | SQL keyword | In your app |
|-----------|-------------|-------------|
| **C**reate | `INSERT` | A user posts a found item |
| **R**ead | `SELECT` | The browse page lists items |
| **U**pdate | `UPDATE` | An item is marked as claimed |
| **D**elete | `DELETE` | A user removes their post |

If you can confidently do these four things to your `items` table, you understand the core of databases. Everything after that (searching, joins, points) is a variation on CRUD.


## Exercise: design your database

> **Exercise (do this on paper before any code)**
>
>
> 1. **List the entities.** What are the main "things" your app stores? Think beyond items. (Hint: look back at your user stories. Users? Points?)
> 2. **For your `items` entity, list the columns.** For each one, decide its type (`TEXT`, `INTEGER`) and whether it can be empty.
> 3. **Choose a primary key.** How will you uniquely identify one specific item, even if two items have the same title?
> 4. **Sketch the relationships.** Draw a line between entities that connect. Is each line one to one, one to many, or many to many? (Hint: how many items can one user post?)
>
> Keep this sketch. You will build the `items` table from it today, and the rest across the coming weeks.
>
> **Guiding constraint:** only add a column if a user story actually needs it. If nothing needs it yet, leave it out.


## Creating a table

You describe a table to the database with a `CREATE TABLE` statement. Here is the one for `items`, which we will build from.

```sql
CREATE TABLE IF NOT EXISTS items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    category TEXT NOT NULL,
    image_url TEXT,
    status TEXT NOT NULL DEFAULT 'available'
);
```

Reading it line by line:

* `id INTEGER PRIMARY KEY AUTOINCREMENT` gives every row a unique number that SQLite fills in for you automatically. This is how you fetch one exact item later.
* `TEXT` stores strings. `INTEGER` stores whole numbers.
* `NOT NULL` means the column must have a value. You cannot post an item with no title.
* `DEFAULT 'available'` means if you do not provide a status, it starts as `available`.
* `IF NOT EXISTS` stops the statement erroring if the table is already there.


### Create the database file

Make a small script that runs the statement above once. Create `init_db.py` in your project root:

```python
import sqlite3

# --- SCHEMA ------------------------------------------------------
# items: every lost, found, or trade listing on the platform
#   id          unique row id, auto-assigned
#   title       short name of the item (required)
#   description longer detail about the item
#   category    lost / found / trade (required)
#   image_url   optional link to a picture
#   status      available / claimed, defaults to available
# -----------------------------------------------------------------

connection = sqlite3.connect("lostfound.db")
cursor = connection.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT,
    category TEXT NOT NULL,
    image_url TEXT,
    status TEXT NOT NULL DEFAULT 'available'
);
""")

connection.commit()
connection.close()
print("Database initialised.")
```

Run it once:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Create the database | `python init_db.py` | `python3 init_db.py` |

You should now see a new file `lostfound.db` appear in your project. Open it in DB Browser to confirm the `items` table exists (it will be empty for now).

---

## The main SQL commands

These are the statements you will use constantly. Read them as sentences.

**Insert a row (Create):**

```sql
INSERT INTO items (title, description, category, image_url)
VALUES ('Blue Water Bottle', 'Metal flask, dent on base', 'lost', 'https://placehold.co/300x200');
```

**Read rows (Read):**

```sql
SELECT * FROM items;                     -- every column, every row
SELECT title, category FROM items;       -- only these two columns
SELECT * FROM items WHERE category = 'lost';   -- only matching rows
SELECT * FROM items ORDER BY id DESC;    -- newest first
```

`WHERE` filters rows. `ORDER BY` sorts them. You will lean on `WHERE` heavily next session when you build search.

**Change a row (Update):**

```sql
UPDATE items SET status = 'claimed' WHERE id = 1;
```

**Remove a row (Delete):**

```sql
DELETE FROM items WHERE id = 1;
```

> ⚠️ **The most important habit of the week:** notice that `UPDATE` and `DELETE` without a `WHERE` clause hit **every row**. `DELETE FROM items;` empties the whole table. Always pair them with a `WHERE`.

---

## Talking to SQLite from Python

Your Flask app never types raw SQL by hand. It runs SQL through the `sqlite3` module. The pattern is always the same five steps.

```python
import sqlite3

# 1. open the file
connection = sqlite3.connect("lostfound.db")

# 2. get a cursor
cursor = connection.cursor()

# 3. run a statement
cursor.execute("SELECT * FROM items")

# 4. read the results
rows = cursor.fetchall()

# 5. close up
connection.close()
```

Two things to internalise:

* **`commit` saves writes.** After an `INSERT`, `UPDATE`, or `DELETE`, call `connection.commit()` or the change is silently thrown away. This is the classic beginner trap where "my data keeps disappearing".
* **`fetchall` vs `fetchone`.** `fetchall()` returns a list of all matching rows. `fetchone()` returns just the next one, handy when you know there is only one (fetching a single item by id).

### Making rows behave like dictionaries

By default a row comes back as a plain tuple like `(1, 'Blue Water Bottle', ...)`, which is awkward to turn into JSON. Set a `row_factory` once and each row acts like a dictionary instead.

Create a small helper, `db.py`:

```python
import sqlite3

DB_NAME = "lostfound.db"

def get_connection():
    connection = sqlite3.connect(DB_NAME)
    connection.row_factory = sqlite3.Row   # rows behave like dicts
    return connection
```

Now `row["title"]` works, and `dict(row)` converts a row straight into something `jsonify` understands.

### Passing values safely with `?`

When a value comes from a user, never glue it into the SQL string yourself. Use a `?` placeholder and pass the value separately. SQLite fills it in safely.

```python
# Correct: the value goes in through the ? placeholder
cursor.execute("SELECT * FROM items WHERE category = ?", ("lost",))

# Never do this: building the string by hand is unsafe
# cursor.execute("SELECT * FROM items WHERE category = '" + user_input + "'")
```

Build this habit from your very first query. It keeps your code clean now, and in a later session you will learn the name for the attack it prevents (SQL injection).

---

## Seed data

An empty database is hard to build a frontend against. **Seed data** is a set of realistic sample rows you load in so you have something to display while developing. It replaces the hardcoded `ITEMS` list that currently sits in your `app.py`.

> **Try this prompt in Claude Code:**
>
> > "Write a Python script called `seed.py` that imports `get_connection` from my `db.py`. It should insert 6 realistic campus lost-and-found items into the `items` table using `executemany` and a parameterised query. Each item needs a title, description, category (one of lost, found, trade), and an image_url using https://placehold.co placeholders. Commit and close the connection, then print how many rows were inserted."
>
> Read the output line by line before running it. Can you explain what `executemany` does differently from `execute`?

For reference, a correct `seed.py` looks like this:

```python
from db import get_connection

ITEMS = [
    ("Blue Water Bottle", "Metal flask, small dent on the base.", "lost", "https://placehold.co/300x200"),
    ("Black Umbrella", "Left near the Main Hall entrance.", "found", "https://placehold.co/300x200"),
    ("Casio Calculator", "Scientific calculator, name inked on back.", "found", "https://placehold.co/300x200"),
    ("Textbook: Physics AS", "Good condition, willing to trade.", "trade", "https://placehold.co/300x200"),
    ("Set of Keys", "Three keys on a red lanyard.", "lost", "https://placehold.co/300x200"),
    ("Wired Earphones", "Found in the library, second floor.", "found", "https://placehold.co/300x200"),
]

def seed():
    connection = get_connection()
    cursor = connection.cursor()
    cursor.executemany(
        "INSERT INTO items (title, description, category, image_url) VALUES (?, ?, ?, ?)",
        ITEMS,
    )
    connection.commit()
    connection.close()
    print(f"Inserted {len(ITEMS)} items.")

if __name__ == "__main__":
    seed()
```

Run it once:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Load the seed data | `python seed.py` | `python3 seed.py` |

Open `lostfound.db` in DB Browser again. Your six rows should be sitting in the `items` table.

### Point the GET route at the database

Now replace the hardcoded list in `app.py` with a real query. This is the moment your frontend starts showing genuinely stored data.

```python
from flask import Flask, jsonify, render_template
from db import get_connection

app = Flask(__name__)

@app.route('/items', methods=['GET'])
def get_items():
    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute("SELECT id, title, description, category, image_url, status FROM items")
    rows = cursor.fetchall()
    connection.close()

    items = [dict(row) for row in rows]
    return jsonify(items)
```

Run the app and visit `http://127.0.0.1:5001/items`. You should see your seeded items returned as JSON, straight from the database.


## Resetting the database

Once you have posted a few test items, your `lostfound.db` fills up with junk rows. During development, you often want an empty database which is called a reset.

The bluntest reset deletes the database file and rebuilds it. Because everything in it right now is disposable seed data, this is completely safe. You would never do this to real user data, which is exactly what migrations (next section) are for.


**The manual way (so you see what happens):**

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| ⚠️ Delete the database file | `del lostfound.db` | `rm lostfound.db` |
| Rebuild the empty schema | `python init_db.py` | `python3 init_db.py` |
| Reload the seed data | `python seed.py` | `python3 seed.py` |

**The one command way.** Doing three steps by hand every time is tedious and easy to get wrong. Wrap it in a script. This needs one small change first: make `init_db.py` and `seed.py` expose a function instead of running everything at import time.

Update `init_db.py` so the work lives in a function:

```python
import sqlite3

DB_NAME = "lostfound.db"

def init_db():
    connection = sqlite3.connect(DB_NAME)
    cursor = connection.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT,
        category TEXT NOT NULL,
        image_url TEXT,
        status TEXT NOT NULL DEFAULT 'available'
    );
    """)
    connection.commit()
    connection.close()
    print("Database initialised.")

if __name__ == "__main__":
    init_db()
```

Update `seed.py` the same way:

```python
from db import get_connection

ITEMS = [
    ("Blue Water Bottle", "Metal flask, small dent on the base.", "lost", "https://placehold.co/300x200"),
    ("Black Umbrella", "Left near the Main Hall entrance.", "found", "https://placehold.co/300x200"),
    ("Casio Calculator", "Scientific calculator, name inked on back.", "found", "https://placehold.co/300x200"),
    ("Textbook: Physics AS", "Good condition, willing to trade.", "trade", "https://placehold.co/300x200"),
    ("Set of Keys", "Three keys on a red lanyard.", "lost", "https://placehold.co/300x200"),
    ("Wired Earphones", "Found in the library, second floor.", "found", "https://placehold.co/300x200"),
]

def seed():
    connection = get_connection()
    cursor = connection.cursor()
    cursor.executemany(
        "INSERT INTO items (title, description, category, image_url) VALUES (?, ?, ?, ?)",
        ITEMS,
    )
    connection.commit()
    connection.close()
    print(f"Inserted {len(ITEMS)} items.")

if __name__ == "__main__":
    seed()
```

The `if __name__ == "__main__":` line means the script still runs the same way when you call it directly, but the function can now be imported and reused.

Now create `reset_db.py` in your project root:

```python
import os
from init_db import init_db
from seed import seed

DB_NAME = "lostfound.db"

if os.path.exists(DB_NAME):
    os.remove(DB_NAME)
    print(f"Removed old {DB_NAME}.")

init_db()
seed()
print("Reset complete.")
```

One command, same result on either OS (Python does the file deletion for you, so no `del` or `rm`):

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Wipe and rebuild the database | `python reset_db.py` | `python3 reset_db.py` |

> ⚠️ A reset **destroys everything** in the database. That is fine now because the data is fake seed data you can regenerate. The moment real users exist, do NOT use reset. Use migrations.


## Schema changes & migrations

### When the schema needs to change

Your database design is not finished. Over the coming weeks you will add a `users` table, link items to whoever posted them, add points, add an audit log. Each of those is a **change to the schema of a database that already holds data**.

**The big idea.** Think of migrations as *git commits for your database structure*. Each one is a small, ordered, recorded change that moves the schema from one version to the next. Resetting is knocking the house down and rebuilding it; a migration is renovating a room while you still live there, without throwing out your furniture.

**What a migration actually is.** An ordered, versioned instruction that alters the schema in place and keeps existing rows intact. Instead of rewriting `CREATE TABLE`, you issue a change such as:

```sql
ALTER TABLE items ADD COLUMN posted_by INTEGER;
```

Run once, this adds a `posted_by` column to every existing row (filled with `NULL`) without deleting a thing. `ALTER TABLE ... ADD COLUMN` is the migration primitive you will use most.

**Two tools, two situations:**

| Situation | Tool | Keeps data? |
|-----------|------|-------------|
| Early development, data is disposable seed data | `reset_db.py` | No |
| The table holds data you cannot lose | `ALTER TABLE` migration | Yes |

**File Management** Keep a `migrations/` folder of numbered SQL files that run in strict order, so any machine can replay the exact same schema history:

```
migrations/
  001_create_items.sql
  002_add_posted_by.sql
  003_create_users.sql
```

Tools like Alembic or Flask-Migrate automate this for large apps. You do not need them here. For this course: **while the data is fake, reset freely; once it is real, migrate.**


## Exercise: fetch real data into your cards

Your landing page `script.js` still renders the old placeholder fields (`item.name`, `item.location`). Those columns do not exist any more. Time to update it.

> **Exercise (graduated)**
>
> **Guided:** the fetch and loop in `script.js` already work. Change the card rendering so it reads the real columns your API now returns: `item.title`, `item.description`, and `item.category`.
>
> **Independent:** add the item image. Create an `img` element, set its `src` to `item.image_url`, and append it to the card. (Hint: `const img = document.createElement("img"); img.src = item.image_url;`)
>
> **Stretch:** handle a broken or missing image gracefully so a dead link does not leave an ugly gap. (Hint: look up the image element's `onerror` handler, and decide what to show instead.)
>
> Success looks like: refresh the page and see all six seeded items rendered as cards with their real title, description, category, and picture.


## Routing rename checklist

In the future part of the app, we want to have `GET /items/:id` to return item JSON. But this exercise wants `/items/<id>` to be a browsable page. The same path can't be both a JSON endpoint and an HTML shell. So let's do a few changes:

- [ ] Week 3 form: `action="/items"` becomes `action="/api/items"`.
- [ ] Week 4/5 landing fetch in `script.js`: `fetch("/items")` becomes `fetch("/api/items")`.
- [ ] Existing GET list route: rename `@app.route('/items', ...)` to `@app.route('/api/items', ...)`.



## Homework

- [X] Finalise your database design sketch from the in-session exercise. List every entity, and for the `items` table note each column, its type, and whether it can be empty.
- [ ] Make sure `init_db.py`, `db.py`, and `seed.py` all run cleanly and produce a populated `lostfound.db`, according to your ER diagram.
- [ ] Add a schema comment at the top of `init_db.py` explaining each column of each table and why it exists. Tie each column back to user stories where relevant.
- [ ] Make the Post Item form write a real row to your DB.
- [ ] Complete the card rendering exercise so the landing page shows your seeded items (title, description, category, image) with no full-page reload.

> Replace your `style.css` file with the following content:
> ```
> body {
>    font-family: sans-serif;
>    margin: 2rem;
> }
>
> .item-card {
>    border: 1px solid #ccc;
>    border-radius: 6px;
>    padding: 1rem;
>    margin-bottom: 0.75rem;
> }
> ```

- [ ] Click a card to open that item's page.
> Each card on the landing page should link to `/items/<id>`, a page that shows just that one item.
>
> **Guided:** in `script.js`, wrap each card in a link so the whole card is clickable. Set its `href` to the item's id:
> ```js
> const card = document.createElement("a");
> card.href = `/items/${item.id}`;
> card.classList.add("item-card");
> ```
>
> **Independent:** build the detail page.
>
> Success looks like: click any card, land on `/items/3`, see that item's full detail, and get a clean message if the id does not exist.
- [ ] Commit and push everything with a clear message, for example `feat: add sqlite database, seed data, and db-backed items route`.
