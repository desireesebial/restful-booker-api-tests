# restful-booker-api-tests

Postman collection for the [Restful-Booker API](https://restful-booker.herokuapp.com/apidoc/index.html), run in CI with Newman.

## What is in the collection

**Booking lifecycle (chained)** - the requests run in order and pass values to each other:

1. `POST /auth` - get a token, saved as `authToken`
2. `POST /booking` - create a booking, saved as `bookingId`
3. `PUT /booking/:id` - update it, using `authToken` as the cookie
4. `GET /booking/:id` - check the update was saved
5. `DELETE /booking/:id` - delete it, then check `GET` returns 404

**Negative tests** - one per endpoint. Each one can be run on its own:

- N1 wrong password on `/auth`
- N2 missing fields on `POST /booking`
- N3 `PUT` without a token
- N4 `GET` a booking that does not exist
- N5 `DELETE` with a bad token

N3 and N5 create their own booking first, check afterwards that it was not changed or deleted, and then delete it with a fresh token.

## Things this API does that you might not expect

- A failed login returns `200` with `{"reason":"Bad credentials"}` instead of `401`. N1 checks for this.
- Missing fields on `POST /booking` return `500` instead of `400`. N2 checks for the `500`, so if the API gets fixed the test will fail and tell you.
- A successful `DELETE` returns `201`.

## What is checked

Status codes, the `Content-Type` header, JSON schemas (token, create, update, get), and the actual values in the booking. The schemas and the booking dates are set once in the collection pre-request script. Dates are generated each day, so there are no hard-coded years.

Scripts read `baseUrl` and the login details with `pm.variables.get`, so you can switch to another host with an environment or `newman --env-var baseUrl=...`. The login in the collection variables is the public demo login from the API docs.

## Run it

```bash
npm install -g newman@6.2.3 newman-reporter-htmlextra@1.23.1
newman run Restful-Booker.postman_collection.json -r cli,htmlextra
```

## CI

`.github/workflows/newman-api.yml` runs on push, pull request and manual start. It uploads the JUnit file and the HTML report as an artifact, even when tests fail.
