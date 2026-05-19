# Django CRUD — Research Notes

---

## 1. CRUD

**CRUD** stands for **Create, Read, Update, Delete**. It is not a technology — it's a concept for the four main operations performed on data. Almost every feature in every web app maps to one of these four operations. They are the atomic building blocks of interacting with stored data.

**Purpose:** CRUD gives developers a mental model for designing data interactions. Instead of asking "what does this app need to do?", you ask "which CRUD operations does each resource need?" — and the structure of the app follows naturally.

| Operation | What it does | SQL | HTTP Method |
|-----------|-------------|-----|-------------|
| Create | Add new data | INSERT | POST |
| Read | Retrieve data | SELECT | GET |
| Update | Modify existing data | UPDATE | PUT/PATCH |
| Delete | Remove data | DELETE | DELETE |

**Example:** A library management system. Librarians add books (C), search for them (R), update availability when borrowed (U), and remove books that are lost (D). Simple — but that's the entire data layer of the app.

---

## 2. Architecture Patterns

An **architectural pattern** is a high-level, reusable solution to a recurring structural problem in software design. It defines how to organize your code: which components exist, what each one is responsible for, and how they communicate.

The goal is **separation of concerns** — each part of the system has one job, so code is easier to maintain, test, and scale.

---

### MVC — Model View Controller

The classic pattern, used by Ruby on Rails, Spring, and others. Divides an app into three layers:

- **Model** — represents the data and the rules around it. It is a class.
- **View** — the presentation layer. Pure HTML/CSS. Receives data from the Controller and renders it for the user. Doesn't know where the data came from.
- **Controller** — the coordinator between Model and View. A messenger that knows both sides exist and knows how to talk to each one.

Flow: `Request → Controller receives → calls the Model to get/save data → passes the result to the View`

---

### MVT — Model View Template

Django's version of MVC. Same philosophy, different naming:

- **Model** — data and database logic. It is a class.
- **View** — Django's controller. A Python function or class that receives a request, processes it, fetches data from the Model, and decides what to return.
- **Template** — Django's view. An HTML file with special Django syntax that displays the data it receives from the View.

**Key practical difference:** In MVC the developer writes the Controller and manually connects it to the View. In Django's MVT the framework handles the rendering pipeline — Django's URL dispatcher and template engine do the wiring automatically. You write less glue code.

> **Framework** — prebuilt code structure that handles the repetitive parts of building an application. You plug in your logic into the slots it provides. Less flexible, but you skip a lot of boilerplate.
>
> **Rendering pipeline** — chain of steps that turns a user's request into an HTML page they can see.

---

## 3. Django

Django is a high-level web development framework in Python that promotes fast development and clean, pragmatic design. It was created to make building complex, database-oriented web apps easy.

It follows the **MVT** pattern and its core philosophy is **DRY (Don't Repeat Yourself)** — structure code so each piece of logic lives in exactly one place, and everything else just references it.

---

### Project Structure

A Django project is organized into **apps**. Each app is a self-contained module responsible for one feature area — it owns its own models, views, and templates. The project is the container that connects all the apps together.

Inside each app:

**`models.py`**
Defines the data structure. Each class here automatically becomes a table in the database via the ORM (Object Relational Mapper).

**`views.py`**
Functions or classes that receive an HTTP request and return a response. This is where the logic lives: validate a form, query the database, decide what template to render.

**`urls.py`**
The routing table for the app. Maps URL patterns to view functions.

**`templates/`**
HTML files with Django's template language injected. At runtime, Django fills in the dynamic parts and sends plain HTML to the browser. Two syntaxes:
- `{{ variable }}` — outputs a value
- `{% tag %}` — executes logic. The template engine processes all `{% %}` tags on the server before sending HTML to the browser.

**`admin.py`**
Register your models here and Django auto-generates a fully functional admin dashboard where you can create, edit, and delete records. No extra code needed.

**`apps.py`**
Configuration metadata for the app itself — name, label. Rarely touched directly.

**`migrations/`**
Django automatically manages this folder. Every time you change a model it generates a migration file — a set of instructions for how to update the database to match. Like version control for your database schema.

---

### Data Flow: HTML Form → Database

![Raccoon approves this flow](https://i.imgur.com/r3yyYva.png)

1. User fills out the form in the browser
2. Browser sends an **HTTP POST request** to a URL, carrying the form data in the request body
3. `urls.py` matches the URL and calls the corresponding View function
4. The View receives the request object, extracts the POST data, and passes it to a `ModelForm` for validation
5. If the form is valid, the View calls `form.save()`, which creates a Model instance and calls `.save()` on it
6. The Model's `.save()` method triggers Django's ORM to generate and execute a SQL `INSERT` statement against the database
7. The View returns an HTTP redirect or a success template back to the browser

---

### Commands and Tools

| Tool | What it does |
|------|-------------|
| `django-admin startproject` | Sets up the initial project directory with all necessary config files |
| `startapp` | Creates a new app module inside the project with its own `models.py`, `views.py`, etc. |
| `makemigrations` | Reads current Model definitions and generates migration files — detects changes and writes the instructions |
| `migrate` | Executes pending migration files against the actual database. Tables are created or modified |
| `runserver` | Starts a local development server at `http://127.0.0.1:8000/` for testing without deploying |
| `ModelForm` | Class that auto-generates an HTML form from a Model definition, with correct input types and validation |
| `shell` | Opens a Python shell with Django's full environment loaded, useful for testing ORM queries interactively |
| `admin` | Built-in back-office CRUD interface, auto-generated from registered models |

---

### Django Admin

Django ships with a fully functional, auto-generated administration interface at `/admin/`. Register a model in `admin.py` and Django generates a complete CRUD UI: a list view with search and filtering, a detail/edit form, and delete confirmation.

**How it works internally:** The admin reads the Model's field definitions at startup and dynamically constructs forms and views for each registered model. It has its own URL routing, authentication system, and permission controls — you can restrict which admin users can create, edit, or delete which models.

It is a developer and superuser tool for managing application data directly without touching the database. Particularly useful in early stages of a project for testing the data model before building the real frontend.

---

## 4. REST and Django REST Framework

**Django is not REST by default.** In the standard flow, the server receives a request, queries the database, renders a complete HTML page using a template, and sends that HTML to the browser. This is called **server-side rendering** — the server produces the final UI.

**REST (Representational State Transfer)** is an architectural style for building APIs. The server doesn't return HTML — it returns raw structured data (JSON). The client (frontend) is responsible for rendering the UI with that data.

REST principles:

- **Client-server** — frontend and backend are independent, they communicate only through the API
- **Stateless** — every request must carry all the info the server needs. No memory between requests. Improves scalability and reliability
- **Cacheable** — clients can cache responses when appropriate. The server indicates whether a response can be cached and for how long. Reduces server load and improves performance
- **Uniform interface** — resources identified by URLs; clients manipulate resources through representations (JSON, XML); messages are self-descriptive; clients navigate through the app via hyperlinks (HATEOAS)
- **Layered system** — the architecture can have hierarchical layers (load balancers, caches, security layers, gateways)

---

### Django REST Framework (DRF)

A third-party library installed on top of Django. You install it and add it to `INSTALLED_APPS` in `settings.py` — it plugs into Django's existing system and adds REST API capabilities.

Instead of returning HTML, the server returns raw JSON data. The client receives that JSON and renders it however it wants. This is why it's called an **API (Application Programming Interface)** — it's not a website, it's a data service that other applications consume.

| Component | What it does |
|-----------|-------------|
| **Serializer** | Translator. Python object → JSON out / JSON in → validated Python object |
| **ViewSet** | Class that contains all CRUD operations together |
| **Router** | Registers a ViewSet and auto-generates all URLs. No manual `urls.py` entries needed |

---

> **Django** = automatic mode. Makes a lot of decisions for you, handles the settings, gives you a solid structure from the start. Less control, but faster to get something working.
>
> **FastAPI** = manual mode. You decide everything. More flexible and powerful.
