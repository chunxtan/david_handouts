# Week 2: UI Foundations & Introducing AI Tools

**Date:** 21 June 2026

---

## What we're covering today

* Ensuring our setup from Week 1 is ready to go!
* Request-Response Cycle
* Semantic HTML
* AI Tools (Claude Code, Gemini)
* Building HTML form with Gemini
---

## Request-Response Model

**What happens:**
1. Browser or JavaScript sends a request to a server
2. Server processes the request
3. Server sends back a response
4. This cycle repeats on every page load and every fetch call

**A request has:**
1. URL: which resource is being asked for
2. Method: GET to read data, POST to send new data
3. Body (optional): data being sent, such as form contents

**A response has:**
1. Status code: a number showing success or failure (200 success, 404 not found, 500 server error)
2. Body: data sent back, usually as JSON

![image](1_BwgxZogZeleUD6USCXobNA.webp)

### Readings
* [HTTP Requests](https://medium.com/@cronjit/understanding-request-and-response-in-web-development-a-comprehensive-guide-f357d25d0843)
* [HTTP Request Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)


---

## Concepts

**Wireframing & UX**
A blueprint shows where the kitchen and front door go, not paint colours. A wireframe shows where content sits on a page, not fonts or colours. It's a low fidelity sketch of a screen's layout and content hierarchy, made before any code. Today you sketch Browse, Post Item, and Item Detail.

![image](low-fid-wireframe.png)
![image](figma-prototype.png)

**Semantic HTML**
Semantic tags such as `header`, `main`, `nav`, `footer`, `form`, `label`, `input`, and `button` describe what content *is*, not just how it looks, unlike a generic `div`. The Post Item form uses `form`, `label`, and `input` instead of div soup, so a screen reader, a browser, and a future developer (including future you) can all read the page's structure at a glance.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Campus Lost and Found</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>Campus Lost and Found</h1>
  </header>
  <main>
    <section class="card-row">
      <article class="card">Lost: Blue Water Bottle</article>
      <article class="card">Found: Calculator</article>
      <article class="card">Lost: Umbrella</article>
    </section>
  </main>
  <footer>
    <p>Built by David, Week 2</p>
  </footer>
</body>
</html>
```

**Try it here: https://www.w3schools.com/html/tryit.asp?filename=tryhtml_elements**


**CSS**
File that dicates the style of HTML elements.
```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
}

header, footer {
  background-color: #2c3e50;
  color: white;
  padding: 16px;
  text-align: center;
}

.card-row {
  display: flex;
  gap: 16px;
  padding: 24px;
}

.card {
  background-color: #ecf0f1;
  border: 1px solid #bdc3c7;
  border-radius: 8px;
  padding: 16px;
  flex: 1;
}
```

---

## Commands & code reference

| Action | 🖥 Windows | 🍎 Mac / Linux |
|---|---|---|
| Open the project folder in VS Code | `code .` | `code .` |
| Create the two new files (`index.html`, `style.css`) ⚠️ | `New-Item index.html, style.css` | `touch index.html style.css` |
| Update `app.py` | See code below | Same steps
| Preview your page live | In the terminal, with your virtual environment running, run `app.py` and `Ctrl`+ Click on the link e.g. `http://127.0.0.1:5000` | Same steps |
| Stage your work | `git add .` | `git add .` |
| Commit your work | `git commit -m "Create base form"` | Same command |
| Push to GitHub | `git push` | `git push` |

---

## Worked example

Copy the following code into the relevant files. If these files are not already in your repository, create them.

Your project structure should be somewhat like this:
```
.
├── app.py
├── static
│   ├── main.js
│   └── style.css
├── templates
│   └── index.html
└── venv
```

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Campus Lost and Found</title>
  <link rel="stylesheet" href="/static/style.css">
</head>
<body>
  <header>
    <h1>Campus Lost and Found</h1>
  </header>
  <main>
    <section class="card-row">
      <article class="card">Lost: Blue Water Bottle</article>
      <article class="card">Found: Calculator</article>
      <article class="card">Lost: Umbrella</article>
    </section>
  </main>
  <footer>
    <p>Built by David, Week 2</p>
  </footer>
</body>
</html>
```

**style.css**
```css
body {
  font-family: Arial, sans-serif;
  margin: 0;
}

header, footer {
  background-color: #2c3e50;
  color: white;
  padding: 16px;
  text-align: center;
}

.card-row {
  display: flex;
  gap: 16px;
  padding: 24px;
}

.card {
  background-color: #ecf0f1;
  border: 1px solid #bdc3c7;
  border-radius: 8px;
  padding: 16px;
  flex: 1;
}
```

**app.py**
```python
from flask import Flask, jsonify, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/hello')
def hello():
    return jsonify({'message': 'Hello from Flask!'})

if __name__ == '__main__':
    app.run(debug=True)
```
---

## Exercises

Let's try thse with the help of AI tools. We'll start with Gemini for today as it is the most accessible. But hopefully, we'll get to use Claude, which is the best in the industry at the moment.

1. List the fields the form needs.
2. Write the HTML structure yourself first.
3. Write prompt to review HTML.
4. Write prompt to generate CSS.

---

## Homework

> Complete the "Create Item" form via HTML and CSS files.

---

## Key terms

| Term | Meaning |
|---|---|
| Wireframe | A low fidelity sketch of a screen's layout, made before writing code |
| UX | (User Experience) How easy and intuitive an interface feels to use |
| Semantic HTML | Tags that describe the meaning of content, rather than generic containers |
| Box model | The content, padding, border, and margin that make up every HTML element |
| Flexbox | A CSS layout mode that arranges child elements in a flexible row or column |
| Responsive design | Layout that adapts to different screen widths, fully covered in Week 9 |

---

## Next week preview

Next week: add JavaScript so the form can respond to clicks without reloading the page.
