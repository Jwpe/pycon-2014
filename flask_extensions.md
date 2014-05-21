# Designing Flask Extensions

## Topics

- Intro
- Case study
- Beyond the basics
- Questions

## Flask Intro

"The idea of Flask is to build a good found"ation for all applications. Everything else is up to you or extensions."
- Armin Ronacher

### The app object

- `app` is the heart of Flask functionality
- ways we can change the `app` object:
    - Middleware
    - Hooking into the request path

- `app.config` is a place to store config vars - sort of a dictionary

## Case study

- `init_app` separated from `_init_` is a common Flask idiom. This allows extension to be instantiated with any app.

- `app.add_template_test` allows you to add template functions to Jinja

- Flask has a special registry for extensions: `app.extensions`

## Beyond the basics

### Good extensions to copy

- Flask-Debug-Toolbar
- Flask-SeaSurf - csrf, request processing, cookies
- Flask-Admin - admin page port
- Flask-Classy - adds class-based views for API building