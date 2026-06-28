# Week 1 — Kickoff, Requirements & Setup
**Campus Lost-and-Found & Second-Hand Trading Platform**
**Full-Stack Development with Flask & JavaScript · June 2026**

---

> Keep this handout — it's your reference card for the commands and concepts covered today.

---

## What We're Covering Today

We're starting by understanding the problem before writing a single line of code. You'll learn how a modern web application is structured, write user stories that will guide every feature decision for the next 12 weeks, set up your GitHub repository, configure your development environment, and run your first Flask API route.

---

## The Big Idea

Think of a restaurant. The **dining room** is everything the customer sees and touches — the tables, menus, and plates. The **kitchen** is where all the real work happens, hidden from view. A web application works exactly the same way: the **frontend** (HTML, CSS, JavaScript) is the dining room; the **backend** (Python + Flask) is the kitchen. They communicate through a shared language called **JSON** — like a waiter carrying orders back and forth.

You are going to build both sides over 12 weeks.

---

## 1. Full-Stack Architecture

```
Browser (JS)  →  HTTP Request  →  Flask Server (Python)  →  Query  →  SQLite DB
Renders cards ←  JSON Response ←  Builds JSON from data  ←  Rows  ←  Stores items
```

| Layer | Technology | What it does |
|-------|-----------|--------------|
| Frontend | HTML + CSS + JavaScript | Structure, style, and behaviour in the browser |
| Backend | Python + Flask | API logic, route handling, business rules |
| Database | SQLite | Persistent storage for items, users, points |
| Data format | JSON | How the frontend and backend talk to each other |

---

## 2. Today's Project: Campus Lost-and-Found & Trading Platform

You are building a web application where students can:
- Post items they have found or want to sell
- Browse and search by keyword or category
- Claim an item and contact the poster
- Earn points for posting and helping others

By the end of Week 12 you will have a working app, a public GitHub repository, and a demo video — all suitable for university applications.

---

## 3. Requirements Gathering: User Stories

Before writing any code, good developers ask: **who is this for, and what do they need?**

We capture this as **user stories** — short sentences in a standard format:

> As a **[type of user]**, I want to **[do something]**, so that **[I get some benefit]**.

**Example:**
> As a student who found a wallet on campus, I want to post a description and photo of it, so that the owner can find it and contact me to arrange collection.

Write at least 5 user stories below. Think about different people who might use this app (someone who lost something, someone who found something, a student selling a textbook…).

| # | User Story |
|---|-----------|
| 1 | As a ________________, I want to ________________ so that ________________. |
| 2 | As a ________________, I want to ________________ so that ________________. |
| 3 | As a ________________, I want to ________________ so that ________________. |
| 4 | As a ________________, I want to ________________ so that ________________. |
| 5 | As a ________________, I want to ________________ so that ________________. |

> **Save these in your repo under `docs/requirements.md`** — they will guide every feature decision for the next 12 weeks.

---

## 4. GitHub Repository Setup

Create the GitHub repo **before** setting up your local environment, then clone it down. This is cleaner than initialising Git locally and connecting it later.

### Step 1 — Create the repo on GitHub

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** icon (top right) → **New repository**
3. Fill in:
   - **Repository name:** `campus-lost-and-found`
   - **Description:** Campus lost-and-found and second-hand trading platform
   - **Visibility:** Public ✅
   - **Add a README file:** ✅ (tick this)
   - **Add .gitignore:** choose **Python** from the dropdown ✅
4. Click **Create repository**

### Step 2 — Clone it to your machine

Copy the repo URL from the green **Code** button on GitHub (use HTTPS).

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Clone the repo | `git clone https://github.com/YOUR-USERNAME/campus-lost-and-found.git` | `git clone https://github.com/YOUR-USERNAME/campus-lost-and-found.git` |
| Move into the folder | `cd campus-lost-and-found` | `cd campus-lost-and-found` |

> You now have a local copy of the repo, already connected to GitHub. No `git init` or `git remote add` needed.

### Step 3 — Open in VS Code

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Open folder in VS Code | `code .` | `code .` |

> ⚠️ If `code .` is not recognised on Mac, open VS Code, press `Cmd+Shift+P`, type **Shell Command: Install 'code' command in PATH**, and run it.

---

## 5. Environment Setup

### Step 1 — Check Python is installed

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Check Python version | `python --version` | `python3 --version` |
| Check pip version | `pip --version` | `pip3 --version` |

> You need Python 3.8 or higher. If you see `command not found`, download Python from [python.org](https://python.org).

### Step 2 — Create a virtual environment

Run these commands inside your project folder (the one you just cloned).

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Create virtual environment | `python -m venv venv` | `python3 -m venv venv` |
| Activate it | `venv\Scripts\activate` | `source venv/bin/activate` |
| Confirm it's active | You should see `(venv)` in your prompt | You should see `(venv)` in your prompt |
| Deactivate when done | `deactivate` | `deactivate` |

> **Always activate the virtual environment before working on this project.** If you skip this, Flask won't be found.

### Step 3 — Install Flask

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Install Flask | `pip install flask` | `pip3 install flask` |
| Verify installation | `pip show flask` | `pip3 show flask` |
| Save dependencies | `pip freeze > requirements.txt` | `pip3 freeze > requirements.txt` |

> Commit `requirements.txt` to GitHub — it lets anyone else recreate your environment exactly.

---

## 6. Recommended VS Code Extensions

Install these now via the Extensions sidebar (`Ctrl+Shift+X` / `Cmd+Shift+X`). Search by name and click Install.

| Extension | What it does | Why you need it |
|-----------|-------------|-----------------|
| **Python** (Microsoft) | Python language support, linting, IntelliSense | Essential — install first |
| **Thunder Client** | Test API routes without leaving VS Code | Test your Flask routes directly, no browser needed |
| **Prettier** | Auto-formats HTML, CSS, and JavaScript | Keeps your code neat and consistent |

> You can add **SQLite Viewer** and **GitLens** later — we'll use them properly from Week 4 onwards.

---

## 7. Project Folder Structure

Create this structure inside your project folder. Some folders you'll create now; others will be added as the project grows.

```
campus-lost-and-found/
├── app.py                ← your Flask app (start here)
├── requirements.txt      ← dependencies list
├── static/
│   ├── style.css         ← your CSS
│   └── main.js           ← your JavaScript
├── templates/
│   └── index.html        ← your HTML shell (Jinja2 for the initial page only)
└── docs/
    └── requirements.md   ← user stories go here
```

Create the folders and files:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Create folder | `mkdir static templates docs` | `mkdir static templates docs` |
| Create empty file | `type nul > static\style.css` | `touch static/style.css` |
| Create empty file | `type nul > static\main.js` | `touch static/main.js` |
| Create empty file | `type nul > templates\index.html` | `touch templates/index.html` |

---

## 8. Your First Flask App

Create `app.py` in the root of your project and paste this in:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/hello')
def hello():
    return jsonify({'message': 'Hello from Flask!'})

if __name__ == '__main__':
    app.run(debug=True)
```

Run it:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Run the app | `python app.py` | `python3 app.py` |

Open your browser and go to: **`http://localhost:5000/api/hello`**

You should see:
```json
{"message": "Hello from Flask!"}
```

> `debug=True` means Flask restarts automatically when you save changes. Never use this in production.

### Testing with Thunder Client

Thunder Client lets you test API routes without a browser — useful from day one, essential from Week 3 when you start building POST routes.

1. Click the **Thunder Client** icon in the VS Code sidebar (lightning bolt)
2. Click **New Request**
3. Method: **GET** (already selected)
4. URL: `http://localhost:5000/api/hello`
5. Click **Send**

You should see the JSON response in the right-hand panel. ✅

---

## 9. Git & GitHub — Key Commands

Git tracks every change you make. Commit regularly — at least once per session, ideally after each working feature.

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|---------------|
| Check what has changed | `git status` | `git status` |
| Stage all changes | `git add .` | `git add .` |
| Commit with a message | `git commit -m "your message"` | `git commit -m "your message"` |
| Push to GitHub | `git push origin main` | `git push origin main` |
| Pull latest from GitHub | `git pull` | `git pull` |
| See commit history | `git log --oneline` | `git log --oneline` |

> **Write meaningful commit messages:** `Add hello Flask route` not `changes`. Your commit history is visible to university reviewers.

---

## 10. Exercises

**Exercise 1 — Guided:** Follow Steps 4–8 above. Get `{"message": "Hello from Flask!"}` appearing in Thunder Client. Tick it off.

**Exercise 2 — Independent:** Add a second route to `app.py`:
```python
@app.route('/api/status')
def status():
    return jsonify({'status': 'ok', 'project': 'campus-lost-and-found'})
```
Visit `http://localhost:5000/api/status` in Thunder Client and confirm it works. Then commit: `git commit -m "Add status route"`.

**Exercise 3 — Stretch:** Make the `/api/hello` route return your name and today's date as additional fields in the JSON. Check the Python `datetime` module if you need a hint.

---

## 📋 Homework

> **Time estimate: ~1 to 1.5 hours**

1. Complete your 5 user stories and save them to `docs/requirements.md` in your repo.
2. Make sure your Hello Flask app is working: `http://localhost:5000/api/hello` returns JSON.
3. Push everything to your public GitHub repo.
4. Add a `README.md` with:
   - A short paragraph describing the project
   - Your tech stack (list the languages and tools)
   - How to run the app (activate venv, install requirements, run `python app.py`)

---

## Key Terms

| Term | Meaning |
|------|---------|
| **Full-stack** | Building both the frontend (what users see) and backend (logic + data) |
| **API** | Application Programming Interface — a set of routes your backend exposes for others to call |
| **REST API** | A standard way of designing APIs using HTTP verbs (GET, POST, PATCH, DELETE) |
| **JSON** | JavaScript Object Notation — the data format APIs use to send information |
| **Route** | A URL path in Flask that triggers a Python function (e.g. `/api/items`) |
| **Virtual environment** | An isolated Python environment so project dependencies don't conflict with other projects |
| **User story** | A sentence describing a feature from the perspective of the person using it |
| **`requirements.txt`** | A file listing all Python packages your project needs (created by `pip freeze`) |
| **`debug=True`** | Flask mode that auto-reloads on save and shows detailed errors — development only |

---

## Next Week Preview

Week 2 is all about **design before code**: you'll sketch wireframes for the three key screens of your app before writing a single line of HTML, and then start building the structure and style to match.

### 📖 Reading for Next Week

Read at least one of these before the session to give you a head start on thinking like a designer:

- **What is UI Design?** — Figma Resource Library
  [figma.com/resource-library/what-is-ui-design](https://www.figma.com/resource-library/what-is-ui-design/)
  A clear overview of what UI designers actually do and why it matters.

- **A Quick Guide to UI Design Fundamentals** — Blush Design
  [blush.design/blog/post/guide-ui-design](https://blush.design/blog/post/guide-ui-design)
  Covers the difference between UI and UX, layout, colour, and spacing.
