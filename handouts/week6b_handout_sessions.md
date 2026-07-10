# Week 6 (Part B): User Accounts

**Date:** 15 July 2026

## What we're covering this session

We are giving the app a memory of user identities. A user can register, log in, and stay logged in as they move around the site. Posting an item now requires an account, and every item is stamped with the person who posted it. Identity and security live on the server, never in the browser, and are never delegated wholesale to AI.

* Recap of Part A
* Homework Review
* [The big idea: the cloakroom ticket](#the-big-idea-the-cloakroom-ticket)
* [Concept: sessions and cookies](#concept-sessions-and-cookies)
* [Concept: hashing passwords](#concept-hashing-passwords)
* [Do we need JWT?](#do-we-need-jwt)
* [Setup: the users table and the secret key](#setup-the-users-table-and-the-secret-key)
* [Worked example: register, login, logout](#worked-example-register-login-logout)
* [Concept: protected routes](#concept-protected-routes)
* [Tagging items to their owner](#tagging-items-to-their-owner)
* [Wiring the frontend](#wiring-the-frontend)
* [Exercises](#exercises)
* [Homework](#homework)
* [Key terms](#key-terms)


## The big idea: the cloakroom ticket

Picture a cloakroom at a theatre. You hand over your coat and get a small numbered ticket. The ticket is nothing on its own, just a number. Your coat stays safe behind the counter. Every time you want something, you show the ticket and the attendant looks up what belongs to you.

Login works the same way. When you log in, the server keeps your identity behind the counter (the **session**) and hands your browser a small ticket (the **cookie**). On every later request the browser shows the ticket automatically, and the server looks up who you are. The valuable part stays on the server. The browser only ever holds the stub.


## Concept: sessions and cookies

**What they actually are.** A **cookie** is a small piece of data the server asks the browser to store and send back on every request to that site. A **session** is data the server keeps that is tied to a particular visitor. Flask's `session` object is the bridge: you put values into it in Python, and Flask stores them in a cookie that is **signed** so the browser cannot tamper with it.

**In this project.** On successful login we write the user's id into the session:

```python
session["user_id"] = user["id"]
```

From then on, any route can read `session.get("user_id")` to know who is asking, with no need to send a username on every request. Logging out is just clearing the session.

**Common misconception.** The cookie does not contain the password, and it does not contain trustable identity you can edit. Because Flask signs it, changing `user_id` in the browser breaks the signature and Flask rejects it. The signing depends on a secret key, which is why the app needs one (below).


## Concept: hashing passwords

**Analogy.** A hash is a fingerprint. You can take a fingerprint from a person easily, but you cannot rebuild the person from the fingerprint. To check identity you take a fresh fingerprint and compare. Passwords work the same way: we store the fingerprint of the password, never the password.

**What it actually is.** A hash function turns any input into a fixed scrambled string, and it only runs one way. We store the hash. When someone logs in, we hash what they typed and compare hashes. We never need, or keep, the original.

**In this project.** Werkzeug ships with Flask and gives you exactly two functions:

```python
from werkzeug.security import generate_password_hash, check_password_hash

password_hash = generate_password_hash("hunter2")     # store this
check_password_hash(password_hash, "hunter2")         # True
check_password_hash(password_hash, "wrong")           # False
```

`generate_password_hash` also adds a random **salt** for you, so two users with the same password still get different hashes. You do not manage any of that by hand.

**Common misconception.** Hashing is not encryption. Encryption is reversible with a key. A password hash is meant to be a dead end. If your database leaks, the attacker gets fingerprints, not passwords.


## Do we need JWT?

Short answer for this project: no. You will hear about **JWT** (JSON Web Tokens), but we will not be using them in your project.

JWT solves a distributed problem: many separate services that each need to trust a token without a shared session store. Your app is a single Flask server talking to its own frontend on the same origin. A signed session cookie is the simpler, correct tool here.

## Setup: the users table and the secret key

**No installs.** Werkzeug arrives with Flask. Confirm it imports:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Confirm Werkzeug is available | `python -c "import werkzeug; print(werkzeug.__version__)"` | `python3 -c "import werkzeug; print(werkzeug.__version__)"` |

**The users table.** Update `init_db.py` to create `users` and to add a `posted_by` column to `items`. Create `users` first, because `items.posted_by` points at it.

```python
import sqlite3

DB_NAME = "lostfound.db"

def init_db():
    connection = sqlite3.connect(DB_NAME)
    cursor = connection.cursor()

    cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL UNIQUE,
        password_hash TEXT NOT NULL,
        created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
    """)

    cursor.execute("""
    CREATE TABLE IF NOT EXISTS items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT,
        category TEXT NOT NULL,
        image_url TEXT,
        status TEXT NOT NULL DEFAULT 'available',
        posted_by INTEGER REFERENCES users(id)
    );
    """)

    connection.commit()
    connection.close()
    print("Database initialised.")

if __name__ == "__main__":
    init_db()
```

`username TEXT NOT NULL UNIQUE` means two people cannot share a username, and the database enforces it, not just your Python. `posted_by INTEGER REFERENCES users(id)` is the one to many link you sketched in Week 5: one user, many items.

Because the data is still disposable seed data, rebuild from scratch:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Wipe and rebuild | `python reset_db.py` | `python3 reset_db.py` |

> ⚠️ When real users exist you do not reset. You add `posted_by` with the `ALTER TABLE` migration you previewed in Week 5. While the data is fake, reset freely.

Seeded items have no owner, so their `posted_by` is `NULL`. That is fine, the browse page will show them as posted by an unknown user.

**The secret key.** Flask needs a secret to sign the session cookie. Add this near the top of `app.py`:

```python
app.secret_key = "dev-secret-change-me"
```

> ⚠️ A hardcoded secret is fine for local development only. Later on, you will move it into an environment variable with `python-dotenv` so it never lands in Git.


## Worked example: register, login, logout

Add these imports at the top of `app.py`:

```python
from functools import wraps
from flask import Flask, jsonify, render_template, request, session, redirect
from werkzeug.security import generate_password_hash, check_password_hash
from db import get_connection
```

**Register.** Reject blank input, reject a taken username, store the hash, then log the new user straight in.

```python
@app.route('/api/register', methods=['POST'])
def register():
    username = request.form.get('username', '').strip()
    password = request.form.get('password', '')

    if not username or not password:
        return jsonify({"error": "Username and password are required."}), 400

    connection = get_connection()
    cursor = connection.cursor()

    existing = cursor.execute(
        "SELECT id FROM users WHERE username = ?", (username,)
    ).fetchone()
    if existing:
        connection.close()
        return jsonify({"error": "That username is already taken."}), 409

    cursor.execute(
        "INSERT INTO users (username, password_hash) VALUES (?, ?)",
        (username, generate_password_hash(password)),
    )
    connection.commit()
    user_id = cursor.lastrowid
    connection.close()

    session["user_id"] = user_id
    session["username"] = username
    return jsonify({"status": "ok", "username": username})
```

**Login.** Look the user up, compare hashes, and give the same error whether the username is wrong or the password is wrong, so you never reveal which usernames exist.

```python
@app.route('/api/login', methods=['POST'])
def login():
    username = request.form.get('username', '').strip()
    password = request.form.get('password', '')

    connection = get_connection()
    cursor = connection.cursor()
    user = cursor.execute(
        "SELECT id, username, password_hash FROM users WHERE username = ?",
        (username,),
    ).fetchone()
    connection.close()

    if user is None or not check_password_hash(user["password_hash"], password):
        return jsonify({"error": "Invalid username or password."}), 401

    session["user_id"] = user["id"]
    session["username"] = user["username"]
    return jsonify({"status": "ok", "username": user["username"]})
```

**Logout and "who am I".** Logout clears the session. The `me` route lets the frontend ask whether anyone is logged in, which the nav bar uses.

```python
@app.route('/api/logout', methods=['POST'])
def logout():
    session.clear()
    return jsonify({"status": "ok"})

@app.route('/api/me', methods=['GET'])
def me():
    if "user_id" in session:
        return jsonify({"logged_in": True, "username": session.get("username")})
    return jsonify({"logged_in": False})
```

**The HTML shells.** Two page routes serve the forms. These follow your convention: pages have no `api/` prefix, the JSON endpoints do.

```python
@app.route('/register', methods=['GET'])
def register_page():
    return render_template('register.html')

@app.route('/login', methods=['GET'])
def login_page():
    return render_template('login.html')
```


## Concept: protected routes

**Analogy.** `@login_required` is a bouncer on the door. Before the route runs, the bouncer checks for a valid ticket (a `user_id` in the session). No ticket, no entry.

**What it actually is.** A decorator that wraps a route and runs a check first. If the check fails, the real route never runs.

```python
def login_required(view):
    @wraps(view)
    def wrapped(*args, **kwargs):
        if "user_id" not in session:
            return jsonify({"error": "You must be logged in to do that."}), 401
        return view(*args, **kwargs)
    return wrapped
```

**Why the check must be on the server.** The frontend can hide the post button from logged out users, but that is decoration. Anyone can skip the browser entirely and call the API directly. Prove it in the lesson: with the server running, try to post without logging in.

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Call the protected route directly | `curl -X POST http://127.0.0.1:5001/api/items` | `curl -X POST http://127.0.0.1:5001/api/items` |

You get the 401, not because the frontend stopped you, but because the server did. A check that lives only in the browser is not a sufficiently secure check at all.

**Common misconception.** "The button is hidden, so it is safe." Hiding UI is user experience, not security. Security is the server refusing.


## Tagging items to their owner

Now protect the item POST and stamp each new item with the logged in user. This keeps reading `request.form`, exactly as your route does today. The only additions are the decorator and the `posted_by` value pulled from the session.

> **Note on your POST route.** If your item POST is still at `/items`, rename it to `/api/items` to sit alongside your GET and match your `api/` convention. Flask allows the same path for GET and POST as long as the function names differ.

```python
@app.route('/api/items', methods=['POST'])
@login_required
def create_item():
    title = request.form.get('title')
    description = request.form.get('description')
    category = request.form.get('category')
    image_url = request.form.get('image_url')

    if not title or not category:
        return jsonify({"error": "Title and category are required."}), 400

    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute(
        """
        INSERT INTO items (title, description, category, image_url, posted_by)
        VALUES (?, ?, ?, ?, ?)
        """,
        (title, description, category, image_url, session["user_id"]),
    )
    connection.commit()
    new_id = cursor.lastrowid
    connection.close()

    return jsonify({"status": "ok", "id": new_id})
```

The owner comes from `session["user_id"]`, never from the form. A user cannot claim to be someone else, because the browser never gets to state who the owner is. The server already knows from the session.

**Show the poster on each card.** Update `GET /api/items` to join the users table so each item carries its poster's username. A `LEFT JOIN` keeps items whose `posted_by` is `NULL` (your seeded ones) in the results. This is the final version of the route and it includes the Part A search logic.

```python
@app.route('/api/items', methods=['GET'])
def get_items():
    search = request.args.get('search', '').strip()
    category = request.args.get('category', '').strip()

    query = """
        SELECT items.id, items.title, items.description, items.category,
               items.image_url, items.status,
               users.username AS posted_by_username
        FROM items
        LEFT JOIN users ON items.posted_by = users.id
    """
    conditions = []
    params = []

    if search:
        conditions.append("(items.title LIKE ? OR items.description LIKE ?)")
        params.append(f"%{search}%")
        params.append(f"%{search}%")

    if category:
        conditions.append("items.category = ?")
        params.append(category)

    if conditions:
        query += " WHERE " + " AND ".join(conditions)

    query += " ORDER BY items.id DESC"

    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute(query, params)
    rows = cursor.fetchall()
    connection.close()

    return jsonify([dict(row) for row in rows])
```


## Wiring the frontend

**Register and login pages.** Both are the same shape: a form, a message line for errors, and a shared script. Create `templates/register.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Register</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <h1>Create an account</h1>
    <form id="register-form">
        <div>
            <label for="username">Username</label><br>
            <input type="text" id="username" name="username" required>
        </div>
        <div>
            <label for="password">Password</label><br>
            <input type="password" id="password" name="password" required>
        </div>
        <button type="submit">Register</button>
    </form>
    <p id="message"></p>
    <p>Already have an account? <a href="/login">Log in</a></p>

    <script src="{{ url_for('static', filename='auth.js') }}"></script>
</body>
</html>
```

Create `templates/login.html` the same way, but change the title and heading to Log in, the form id to `login-form`, and the footer link to point at `/register`.

**The shared auth script.** Create `static/auth.js`. It submits whichever form is on the page via `fetch`, so there is no full page reload, and shows the server's error message inline. On success it sends the user to the home page.

```js
const messageEl = document.querySelector("#message");

async function submitAuth(endpoint, form) {
    const response = await fetch(endpoint, {
        method: "POST",
        body: new FormData(form),
    });
    const data = await response.json();

    if (response.ok) {
        window.location.href = "/";
    } else {
        messageEl.textContent = data.error || "Something went wrong.";
    }
}

const registerForm = document.querySelector("#register-form");
if (registerForm) {
    registerForm.addEventListener("submit", (event) => {
        event.preventDefault();
        submitAuth("/api/register", registerForm);
    });
}

const loginForm = document.querySelector("#login-form");
if (loginForm) {
    loginForm.addEventListener("submit", (event) => {
        event.preventDefault();
        submitAuth("/api/login", loginForm);
    });
}
```

Sending `new FormData(form)` is why the server keeps reading `request.form`. We changed how the form is sent (via `fetch`, no reload), not what the server reads.

**The post item form.** Your Post Item form currently submits natively, which reloads the page and cannot handle a login requirement gracefully. Switch it to `fetch` so a logged out user is redirected to login instead of seeing raw JSON. Add a message line and a script to `templates/list_item_form.html`:

```html
<p id="message"></p>
<script src="{{ url_for('static', filename='post_item.js') }}"></script>
```

Create `static/post_item.js`:

```js
const form = document.querySelector("#post-item-form");
const messageEl = document.querySelector("#message");

form.addEventListener("submit", async (event) => {
    event.preventDefault();

    const response = await fetch("/api/items", {
        method: "POST",
        body: new FormData(form),
    });

    if (response.status === 401) {
        window.location.href = "/login";
        return;
    }

    const data = await response.json();
    if (response.ok) {
        window.location.href = "/";
    } else {
        messageEl.textContent = data.error || "Could not post item.";
    }
});
```

> **If you would rather keep the native form submit** for now, you can. The one change needed is in the decorator: instead of returning a 401, redirect a logged out visitor with `return redirect("/login")`. The trade off is that a genuine API caller then receives an HTML page instead of a clean error, which is why `fetch` plus 401 is the tidier fit for an API.

**Show who is logged in, and who posted each item.** In `templates/index.html` add a nav container above everything:

```html
<nav id="nav"></nav>
```

Then in `static/script.js`, add a nav renderer and show the poster on each card. Call `renderNav()` alongside `loadItems()`.

```js
async function renderNav() {
    const navEl = document.querySelector("#nav");
    const response = await fetch("/api/me");
    const data = await response.json();

    if (data.logged_in) {
        navEl.innerHTML = "Logged in as <strong></strong> · <a href='#' id='logout'>Log out</a>";
        navEl.querySelector("strong").textContent = data.username;
        document.querySelector("#logout").addEventListener("click", async (event) => {
            event.preventDefault();
            await fetch("/api/logout", { method: "POST" });
            window.location.reload();
        });
    } else {
        navEl.innerHTML = "<a href='/login'>Log in</a> · <a href='/register'>Register</a>";
    }
}
renderNav();
```

Inside `renderItems`, add the poster line to each card:

```js
const poster = document.createElement("p");
poster.textContent = item.posted_by_username
    ? "Posted by " + item.posted_by_username
    : "Posted by an unknown user";
card.appendChild(poster);
```

> Note the username goes in with `textContent`, not by pasting it into HTML. Inserting user supplied text straight into `innerHTML` is an XSS risk, which is exactly the attack Week 10 demonstrates and fixes.


## Exercises

> **Guided.** Build register end to end. Create `register.html` and `auth.js`, register a user, and confirm in DB Browser that a row appears in `users` with a long `password_hash`, not the plain password.
>
> **Independent.** Build login and the nav bar. Log in, see your username appear, post an item, and confirm the card shows you as the poster. Log out and confirm the nav flips back.
>
> **Stretch.** Prove the server side check. With `curl`, POST to `/api/items` while logged out and confirm the 401. Then log in through the browser, and in `ai-log.md` explain in your own words why hiding the post button in the frontend would not have been enough.


## Homework

> - [ ] Link items to users: store the poster's id when posting, and show the poster's username on every card in the browse view.
> - [ ] Confirm passwords are stored as hashes. Open `users` in DB Browser and check the `password_hash` column. If you can read the password, something is wrong.
> - [ ] Update `/docs/api.md` with the new routes: `POST /api/register`, `POST /api/login`, `POST /api/logout`, and `GET /api/me`. Document inputs, success response, and error responses.
> - [ ] Commit and push with a clear message, for example `feat: add user accounts, hashed passwords, and item ownership`.
>
> **AI Task (Claude Code).** Ask Claude Code: "What are three security risks in the current login implementation?" Write each risk in `ai-log.md` in your own words, and note which ones you already handle and which ones Week 10 will address.


## Key terms

| Term | Meaning |
|------|---------|
| Session | Server side data tied to one visitor, holding who they are between requests |
| Cookie | A small piece of data the browser stores and sends back on each request |
| `secret_key` | The secret Flask uses to sign the session cookie so it cannot be tampered with |
| Hash | A one way scramble of a value, stored instead of the value itself |
| Salt | Random data added before hashing so identical passwords produce different hashes |
| `password_hash` | The stored fingerprint of a password, produced by `generate_password_hash` |
| Decorator | A function that wraps a route to run logic before it, such as an auth check |
| `@login_required` | Our decorator that blocks a route unless a `user_id` is in the session |
| 401 | The HTTP status for "you are not authenticated" |
| Foreign key | A column pointing at another table's row, here `items.posted_by` to `users.id` |

