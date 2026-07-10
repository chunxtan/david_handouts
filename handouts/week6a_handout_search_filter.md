# Week 6 (Part A): Search & Filter

**Date:** 13 July 2026

## What we're covering this session

We are teaching the browse page to answer questions. Right now it dumps every item. By the end you can type a word and pick a category, and the list narrows instantly with no page reload. The real lesson underneath the feature is **API design**.

* Recap of Week 5
* Homework Review
* [API Contracts](#api-contracts)
* [Concept: API contract design](#concept-api-contract-design)
* [Concept: query parameters and request.args](#concept-query-parameters-and-requestargs)
* [Concept: SQL LIKE and wildcards](#concept-sql-like-and-wildcards)
* [Worked example: the search route](#worked-example-the-search-route)
* [Wiring the frontend](#wiring-the-frontend)
* [Testing the contract in Thunder Client](#testing-the-contract-in-thunder-client)
* [Exercises](#exercises)
* [Homework](#homework)
* [Key terms](#key-terms)


## API Contracts

An **API contract** says: here is the URL, here are the inputs you may send, and here is the shape of what comes back, including the awkward cases (nothing matches, you asked for nothing at all).


## Concept: API contract design

**What it actually is.** A short, plain English description of one endpoint: the method, the URL, the query parameters, and the response for the normal case and the edge cases.

**In this project.** Before touching Python, write this out for search:

```
GET /api/items
  Query parameters (all optional):
    search    text to match against title or description
    category  one of: lost, found, trade
  Returns: 200, a JSON array of matching items
  No matches: 200, an empty array []   (not an error)
  No parameters: 200, every item
```

An empty result is a success, not a 404, it just means "no filter." Remember to document these down in comments or in a separate `/docs/api.md` file.

## Concept: query parameters and request.args

The part of a URL after the `?`, written as `key=value` pairs joined by `&`:

```
/api/items?search=bottle&category=lost
```

Flask hands these to you in `request.args`, which behaves like a dictionary. Use `.get()` so a missing parameter returns a default rather than crashing.

```python
search = request.args.get('search', '')     # '' if not supplied
category = request.args.get('category', '')  # '' if not supplied
```

**Common misconception.** `request.args` is for values in the URL (a GET). `request.form` is for values in a submitted form body (a POST). Search reads from the URL, so it is `request.args`.


## Concept: SQL LIKE and wildcards

`LIKE` does pattern matching in SQL. The `%` wildcard means "any run of characters, including none." So `%bottle%` matches "Blue Water Bottle", "bottle opener", and "bottle".

```sql
SELECT * FROM items WHERE title LIKE '%bottle%';
```

In SQLite, `LIKE` is **case insensitive for ASCII letters**, so `bottle` already matches `Bottle` with no extra work.

```python
# The value carries its own wildcards, but it is STILL a bound parameter.
cursor.execute("SELECT * FROM items WHERE title LIKE ?", (f"%{search}%",))
```

## Worked example: the search route

This replaces your current `GET /api/items`. It handles four situations from one function: no filter, search only, category only, and both. Read it top to bottom before running it.

```python
@app.route('/api/items', methods=['GET'])
def get_items():
    search = request.args.get('search', '').strip()
    category = request.args.get('category', '').strip()

    query = "SELECT id, title, description, category, image_url, status FROM items"
    conditions = []
    params = []

    if search:
        conditions.append("(title LIKE ? OR description LIKE ?)")
        params.append(f"%{search}%")
        params.append(f"%{search}%")

    if category:
        conditions.append("category = ?")
        params.append(category)

    if conditions:
        query += " WHERE " + " AND ".join(conditions)

    query += " ORDER BY id DESC"

    connection = get_connection()
    cursor = connection.cursor()
    cursor.execute(query, params)
    rows = cursor.fetchall()
    connection.close()

    return jsonify([dict(row) for row in rows])
```

The shape to notice: we collect `conditions` and `params` in step, then join the conditions with `AND` only if there are any. One search term fills two placeholders because we check both `title` and `description`. Every value sits in `params`, so the query string never contains user input.

Run it and hit the URL directly in the browser to sanity check:

| Action | 🖥 Windows | 🍎 Mac / Linux |
|--------|-----------|----------------|
| Run the app | `python app.py` | `python3 app.py` |

Then visit `http://127.0.0.1:5001/api/items?search=bottle` and confirm you get JSON back.


## Wiring the frontend

Two changes. First, add a search box and a category filter to your landing page. Then make `script.js` send the current values and re-render on every keystroke.

**In `templates/index.html`**, add these controls above the items container:

```html
<input type="text" id="search" placeholder="Search items...">

<select id="category-filter">
    <option value="">All categories</option>
    <option value="lost">Lost</option>
    <option value="found">Found</option>
    <option value="trade">Trade</option>
</select>
```

**In `static/script.js`**, build the query string from the inputs and fetch on each change. `URLSearchParams` assembles the `?search=...&category=...` part for you and skips empty values.

```js
const loadingEl = document.querySelector("#loading");
const containerEl = document.querySelector("#items-container");
const searchInput = document.querySelector("#search");
const categorySelect = document.querySelector("#category-filter");

function renderItems(items) {
    containerEl.innerHTML = "";

    if (items.length === 0) {
        containerEl.innerHTML = "<p>No items match your search.</p>";
        return;
    }

    items.forEach(item => {
        const card = document.createElement("div");
        card.classList.add("item-card");

        const title = document.createElement("h3");
        title.textContent = item.title;

        const category = document.createElement("p");
        category.textContent = item.category;

        card.appendChild(title);
        card.appendChild(category);
        containerEl.appendChild(card);
    });
}

async function loadItems() {
    const params = new URLSearchParams();
    const search = searchInput.value.trim();
    const category = categorySelect.value;
    if (search) params.set("search", search);
    if (category) params.set("category", category);

    loadingEl.style.display = "block";
    try {
        const response = await fetch("/api/items?" + params.toString());
        const items = await response.json();
        renderItems(items);
    } catch (error) {
        containerEl.innerHTML = "<p>Something went wrong loading items.</p>";
    } finally {
        loadingEl.style.display = "none";
    }
}

searchInput.addEventListener("input", loadItems);
categorySelect.addEventListener("change", loadItems);
loadItems();
```

The `input` event fires on every keystroke, so the list narrows as David types. The `change` event fires when the category dropdown changes. No page reload, exactly the pattern from Week 4.


## Testing the contract in Thunder Client

Thunder Client is the REST client built into VS Code (the extension you used in Week 1). Open it from the sidebar, create a new GET request, and test each endpoint.

| Request | You should get |
|---------|----------------|
| `GET /api/items` | Every item |
| `GET /api/items?search=bottle` | Only items with "bottle" in title or description |
| `GET /api/items?search=zzzzz` | `[]`, an empty array, status 200 |
| `GET /api/items?category=lost` | Only items in the lost category |
| `GET /api/items?search=key&category=lost` | Both filters applied together |
| `GET /api/items?search=` | Every item (empty search is not a filter) |


## Exercises

- [ ] Wire up the search box only. Add the `#search` input, add the `searchInput` listener, and confirm the list narrows as you type. Ignore category for now.

- [ ] Add the category dropdown and its `change` listener so search and category work at the same time. Test the combined case in Thunder Client.

- [ ] Searching on every keystroke fires a request per letter. Add a short debounce so the fetch only runs once the user pauses typing (about 300ms). Hint: `setTimeout` and `clearTimeout` around the call inside `loadItems`. What problem does this solve, and what does it cost?


## Homework

> - [ ] Confirm you have at least 3 categories. You already have `lost`, `found`, and `trade`. Consider adding one more that a user story justifies, for example `electronics` or `books`.
> - [ ] Make search and category filter work **simultaneously**. Posting a search word and selecting a category should narrow on both.
> - [ ] Write your API contract for `GET /api/items` in `/docs/api.md` or comments: the method, the URL, the query parameters, the normal response, and the empty result case. Update this file whenever a route changes.
> - [ ] Commit and push with a clear message, for example `feat: add search and category filter to items API`.
>
> **AI Task (Claude Code).** Ask Claude Code to suggest one improvement to your search (for example relevance ranking, or matching across more fields). Read the suggestion, decide whether to implement it. Note: SQLite `LIKE` is already case insensitive for ASCII, so if it suggests that, say why it is already handled.


## Key terms

| Term | Meaning |
|------|---------|
| API contract | A written description of an endpoint: its inputs and its response shape, edge cases included |
| Query parameter | A `key=value` pair after the `?` in a URL, used to refine a GET request |
| `request.args` | Flask's access to query parameters, used like a dictionary with `.get()` |
| `LIKE` | SQL pattern matching operator, used for partial text matches |
| Wildcard (`%`) | In `LIKE`, matches any run of characters including none |
| Parameterised query | A query where user values pass through `?` placeholders, never concatenated into the SQL |


## Next session

Part B: user accounts. Registration, login, and hashed passwords, so every item posted is tagged to the person who posted it.
