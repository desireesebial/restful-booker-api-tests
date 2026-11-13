# Restful-Booker API Tests

Postman collection for testing the Restful-Booker API (`https://restful-booker.herokuapp.com`).

## Collection Structure

1. **Chained Booking Workflow**
   - `POST /auth` - Generates auth token and saves `authToken` to collection variables.
   - `POST /booking` - Creates a booking for John Doe and saves `bookingId` to collection variables.
   - `PUT /booking/:id` - Updates the booking to Jane Doe using `bookingId` and `Cookie: token={{authToken}}`.
   - `GET /booking/:id` - Verifies that the updated fields were saved on the server.
   - `DELETE /booking/:id` - Deletes the test booking using `bookingId` and `Cookie: token={{authToken}}`.

2. **Negative Tests**
   - `POST /auth` with wrong password (`Bad credentials`)
   - `POST /booking` with missing required fields (`500 Internal Server Error`)
   - `PUT /booking/:id` without an auth token (`403 Forbidden`)
   - `GET /booking/:id` for a deleted booking (`404 Not Found`)
   - `DELETE /booking/:id` with an invalid token (`403 Forbidden`)

## How to Run

- **Postman:** Import `Restful-Booker.postman_collection.json` and run the collection using the Collection Runner.
- **CLI (Newman):**
  ```bash
  npx newman run Restful-Booker.postman_collection.json
  ```
