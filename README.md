# router

Routes for a [Moth](https://github.com/samyyy2423/moth) web server made with `server.moth`: each method and
path to a function of the program's, with the parts of the path the route names given to it. 404 for a path
no route has, 405 for a method a path doesn't take, files from a folder, redirects, and checks that run before
every route, such as a login. In Moth alone.

```text
moth add router
```

```
use "server.moth" as web
use the package "router" as router

to home with request
    return "<h1>Notes</h1>"
end

to show_note with request
    return "<h1>Note " + item "id" of request's params + "</h1>"
end

to add_note with request
    # ... keep the note from request's body ...
    return router's redirect("/")
end

set routes to router's routes()
router's get(routes, "/", home)
router's get(routes, "/notes/:id", show_note)       # /notes/42: params is (dictionary where "id" is "42")
router's post(routes, "/notes", add_note)
router's files(routes, "/static", "public")         # /static/app.css sends public/app.css
web's serve(8080, router's answerer(routes))
```

Build it with `moth build app.moth --allow-libraries --allow-read`, as any server.

## What it has

| | |
|---|---|
| `routes()` | an empty table of routes |
| `get`, `post`, `put`, `patch`, `delete` `(routes, path, f)` | a route: `f(request)` answers that method at that path |
| `route(routes, method, path, f)` | a route for any method, or `"*"` for every one |
| `files(routes, prefix, folder)` | GET under `prefix` sends `folder`'s files; nothing outside the folder can be asked for |
| `before(routes, f)` | `f(request)` runs before each route's function; when it gives anything but `nothing`, that's the answer |
| `not_found(routes, f)` | what answers a path no route has, instead of a 404 |
| `answerer(routes)` | a function `server.moth`'s `serve` takes |
| `answer(routes, request)` | one request answered, as `serve` would: handy in tests |
| `redirect(address)` | a 303 to another page, as after a form is sent |
| `answer_with(status, body)` | any status, with a body |

In a path, `:name` is one part of it and `*name` the rest, slashes and all: `/docs/*page` takes
`/docs/guide/start.md` with `page` as `guide/start.md`. Each comes in the request's `params`, decoded
(`%20` is a space). A path fits with or without a `/` at its end, and its query isn't part of it.

The most particular route a path fits is the one used, whatever order the routes were added in: `/notes/new`
before `/notes/:id`, and that before `/notes/*rest`. A HEAD request is answered by the GET route. A path that
fits routes, but not for its method, is answered 405 with the methods it takes in its `Allow` header.

A page that needs a login:

```
to logged_in_only with request
    if request's path is not "/login" and not ("cookie" is in request's headers)
        return router's redirect("/login")
    end
    return nothing
end

router's before(routes, logged_in_only)
```

## Tests

```text
moth test router.test.moth --allow-read
```

## License

Apache License 2.0, as Moth is: see [LICENSE](LICENSE).
