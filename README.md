# Restful-Booker API Automation Suite (Postman + Newman)

Automated stateful API test suite for [Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html) using Postman and Newman in GitHub Actions CI.

## Architecture & Highlights

1. **Chained stateful lifecycle (`Chained Booking Workflow`)**
   - `1. Generate Auth Token (POST /auth)` extracts `{{authToken}}`.
   - `2. Create Booking (POST /booking)` validates the response schema and extracts `{{bookingId}}`.
   - `3. Update Booking (PUT /booking/:id)` uses `{{bookingId}}` and `Cookie: token={{authToken}}`.
   - `4. Verify Updated Booking (GET /booking/:id)` asserts the changes were persisted.
   - `5. Delete Booking & Verify Removal (DELETE /booking/:id)` expects `201 Created`, then a follow-up `GET` must return `404 Not Found`.
2. **Self-contained negative suite (`Negative Tests`)** - every request can run on its own, in any order:
   - The `404` check uses a constant `{{nonExistentBookingId}}`.
   - The unauthorised `PUT` and `DELETE` requests each create their own booking in a pre-request script (`{{negPutTargetId}}`, `{{negDeleteTargetId}}`), prove with a follow-up `GET` that it was not changed or deleted, then delete it using a freshly generated token.
   - Documented API quirks: `200 OK` with `{"reason":"Bad credentials"}` on a failed login (`POST /auth`) and `500 Internal Server Error` on missing required fields (`POST /booking`). Both are asserted exactly and are candidates for defect reports.
3. **Assertions**
   - Status codes, `Content-Type`, and JSON Schema on the auth, create, update and get responses. Schemas are defined once in the collection pre-request script and include a `YYYY-MM-DD` pattern for dates.
   - Specific payload data points (names, price, deposit flag, dates, additional needs).
4. **Dynamic data and variables**
   - Dates are generated at run time (regenerated once per calendar day) in the collection pre-request script.
   - Scripts read `{{baseUrl}}` and credentials through `pm.variables.get`, so a Postman environment or `newman --env-var baseUrl=...` can point the whole suite, including helper requests, at another host.

## Run locally

```bash
npm install -g newman@6.2.3 newman-reporter-htmlextra@1.23.1
newman run Restful-Booker.postman_collection.json -r cli,htmlextra
```

The credentials in the collection variables are the public demo credentials from the Restful-Booker documentation.

## CI

`.github/workflows/newman-api.yml` runs on push, pull request and manual dispatch, then uploads the JUnit XML and HTML report as an artifact (7 days), even when tests fail.
