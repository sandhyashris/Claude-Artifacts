Here is the complete `BACKEND_BEST_PRACTICES.md` file based on the document:

```markdown
# Backend Development: Best Practices and Guidelines

| Field        | Details                  |
|--------------|--------------------------|
| Team         | Cloud Engineering Team |
| Date         | 27/02/2025               |
| Version      | 1.0                      |


---


---

## Coding Best Practices

### 1. Naming Conventions

Naming conventions are essential for writing clear and maintainable code. Proper naming ensures that variables, functions, and classes are self-explanatory and consistent. When naming, always use descriptive names that convey the intent of the variable or function.

**Rules:**
- Stick to widely accepted practices like **camelCase** for variables and functions, and **PascalCase** for classes.
- Avoid using short, non-descriptive names like `x`, `y`, or `temp` unless their meaning is obvious within the context.
- Avoid abbreviations unless widely accepted (e.g., `userId` instead of `uid`).
- Ensure names are self-explanatory (`isUserActive` instead of `flag1`).

Good naming improves collaboration and code readability, especially in large teams.

**Example:**
```js
// Verify the Dynamic Fields and the Body Content - Check if the count and keys are matching
const dynamicFieldsIsValid =
  templateHelper.verifyDynamicFields(schemaValidationData?.body, schemaValidationData?.dynamic_fields)

if (!dynamicFieldsIsValid) {
  return {
    statusCode: 400,
    headers: helper.headers,
    body: JSON.stringify({ message: 'Mismatch between dynamic fields and placeholders. Please ensure they match.' }),
  }
}
```

---

### 2. Code Commenting

Code comments are crucial for maintaining clarity in complex codebases. However, avoid over-commenting or adding redundant explanations. The goal is to clarify **"why" something is being done, not necessarily "how,"** as that should be evident from the code itself.

**Guidelines:**
- Focus comments on tricky logic, algorithm choices, and assumptions made during development.
- Regularly review comments to ensure they remain accurate when code changes.
- Inline documentation and comment standards like [JSDoc](https://jsdoc.app/) (for JavaScript) or [Python docstrings](https://peps.python.org/pep-0257/) are also beneficial in auto-generating documentation from code.

**Note:** You can either write comments based on your own understanding or use the [Mintlify](https://mintlify.com/) or [Amazon Q](https://aws.amazon.com/q/) extension on VS Code.

**Example:**
```js
/* API Documentation Reference */
/* API to receive callback from BTSE for the Crypto Deposit in the wallet address created- From BTSE */
/* BTSE Docs - https://api.btse.co/payment/pay-api#tag/Callbacks/operation/post-payapi-deposit-crypto-address-callback */

// Maximum 6 decimal places are rounded off, to avoid long decimal numbers and avoid partial payments
amount = helper.roundToSixDecimalPlaces(exchangeRateData.rate * existingOrder.order_amount)

/* ********* Explanation for Minimum Payable Amount ******** */
/* Considering a Minimum Payable amount for crypto amount:
   This is to avoid partial payments within the MQR System because of the exchange rate variations
   Suppose for an amount of $100, BTSE gives an amount of 13.45 BTC, and in Coinbase, the exchange rates are 13.40 BTC,
   a payment will be partial just by 0.05 BTC coin
   So the minimum payable amount becomes (13.45 - 0.06725) = 13.38275 BTC */
```

---

### 3. Code Optimization


**Best Practices:**
- Avoid unnecessary computations and database calls inside loops.
- Use helper functions to avoid repetition and improve modularity.
- Optimize database queries, and fetch only minimal data required (avoid using `SELECT *`).
- Use caching mechanisms where applicable (Redis, in-memory caching).
- Consider batch processing for bulk operations.

#### 3.1 Common Helper Functions

- Define reusable functions for date formatting, currency conversion, etc.
- Use utility functions for repetitive validation logic (e.g., email validation, User ID validations, etc.).

#### 3.2 Error Handling

Proper error handling is essential to ensure a smooth user experience while keeping backend errors well-logged for debugging. Follow these best practices when handling errors in backend development:

##### Use Try-Catch Blocks for Error Handling

- Always wrap critical code sections in `try-catch` blocks to handle unexpected failures gracefully.
- For asynchronous operations, use `.catch()` for promises or try-catch inside async functions.

```js
async function getUserProfile(userId) {
  try {
    const user = await db.getUserById(userId);
    if (!user) {
      return {
        statusCode: 404,
        body: JSON.stringify({ message: "User not found" }),
      };
    }
    return {
      statusCode: 200,
      body: JSON.stringify(user),
    };
  } catch (error) {
    console.error("Database error:", error); // Log full error details
    return {
      statusCode: 500,
      body: JSON.stringify({ message: "Something went wrong. Please try again later." }),
    };
  }
}
```

##### Return User-Friendly Error Messages

Provide clear, understandable error messages to clients instead of exposing internal backend errors.

-  `"Database connection to Amazon RDS failed with hostname amazon_rds_host"` — exposes sensitive information.
- `"Something went wrong. Please try again later."` — safe for clients.

For specific business logic failures, provide clear and meaningful error messages:
```json
{ "message": "Payment for this order is already completed." }
```

##### Log Detailed Errors for Debugging

- Do not expose internal stack traces or system-specific messages to the client.
- Always log detailed error information inside the catch block for backend debugging.
- Use `console.error()` for error logging, ensuring logs capture error messages, stack traces, and relevant context.

---

#### 3.3 ESLint Overview

**Reference:** [ESLint with TypeScript & JavaScript](https://eslint.org/)

ESLint is a widely used JavaScript and TypeScript linter that helps enforce coding standards, identify potential errors, and improve code quality. It allows teams to maintain consistency and reduce bugs by flagging problematic patterns.

##### Why Use ESLint?

- Detects and prevents common coding errors.
- Enforces a consistent coding style across the project.
- Helps maintain best practices and cleaner code.
- Supports plugins and rules for better customization.
- It can be integrated with CI/CD pipelines for automated linting.

##### ESLint Configuration – Basic Rules We Follow

**Restricted:**
- `no-console`: Prevents `console.log()` but allows `console.warn()` and `console.error()`.
- `semi: ['error', 'never']`: **No semicolons** allowed at the end of statements.
- `max-len: ['error', { code: 320 }]`: Maximum line length **320 characters** to avoid overly long lines.
- `indent: ['error', 4]`: **4-space indentation** enforced for better readability.

**Allowed:**
- `console.warn()` and `console.error()` are permitted for debugging and error logging.
- CommonJS (`require`) and ES Modules (`import/export`) are supported.
- ECMAScript 2021 features enabled for modern syntax.

##### Proper Use of ESLint Disabling Rules

ESLint helps enforce **code quality and consistency** by identifying known errors and enforcing best practices. However, **disabling ESLint rules for an entire file** is a **bad practice**, as it removes all safeguards and allows potential issues to slip into production.

**❌ Bad Practice: Disabling rules file-wide**
```js
/* eslint-disable no-console */
/* eslint-disable no-continue */
// Console logs used for debugging but not removed (no-console ignored)
console.log("Debugging API response:", apiResponse)
// Using continue inside loops (no-continue ignored)
for (const value of values) {
  if (!value) continue // ⚠️ No lint error, but better alternatives exist
}
```

**Why is this bad?**
- It disables all instances of the rule, making it easy to miss actual problems in the file.
- Developers may forget to re-enable rules, leading to unintended issues being pushed to production.
- Disabling `no-console` globally can result in unnecessary debug logs being committed.

**✅ Good Practice: Disabling ESLint Selectively for Specific Cases**

*Allowing `console.log` for Critical Debugging Only:*
```js
// eslint-disable-next-line no-console
console.log("Critical debug info:", importantData);
```

*Handling `no-await-in-loop` Properly:*

Instead of disabling `no-await-in-loop`, refactor the code to use `Promise.all()` / `Promise.allSettled()` for better performance.

##### Difference Between `Promise.all()` and `Promise.allSettled()`

Both methods handle multiple promises concurrently, but they behave differently when some promises fail:

- **`Promise.all()`** – Fails **immediately** if **any** promise rejects.
- **`Promise.allSettled()`** – Waits for **all** promises to finish, whether they **resolve or reject**, and returns their results.

**Example: Fetching Crypto Exchange Rates for USD**

*Using `Promise.all()` (Fail Fast):*
- If **any** exchange API fails, the entire operation fails.

*When to Use Which:*
- Use `Promise.all()` when **all tasks must succeed** (e.g., fetching data from multiple required sources).
- Use `Promise.allSettled()` when **partial success is acceptable** (e.g., updating multiple records where some may fail).

##### Setting Up ESLint in VS Code

1. Install the ESLint extension in VS Code.
2. Enable `Format on Save` in VS Code settings.
3. Set ESLint as the default formatter.
4. **Format Code Using ESLint**: Press `CTRL + SHIFT + I` to format the document using ESLint. Ensure there are no linting conflicts by checking the Problems tab (`CTRL + SHIFT + M`).

By following these steps, all code formatting will align with our ESLint rules, ensuring consistency across the project.

> ⚠️ **Important Note for New Projects:** If you are starting a new project, make sure to use ESLint's newer configuration format (`eslint.config.js`) instead of the old `.eslintrc` file. The newer format provides better performance, modern syntax support, and improved extensibility.

---

## API Management and Documentation

### 1. Understanding API Requirements

Before developing an API, it is essential to analyze its purpose, reusability, and integration needs. This ensures efficiency, consistency, and scalability.

#### 1.1 Understand the API's Purpose and Reusability

- Determine **why the API is needed** — is it for data retrieval, updates, or real-time communication?
- **Check if an existing API can be reused** instead of creating a new one.
- Evaluate **potential future extensions** to avoid frequent modifications.

#### 1.2 Identify Key Requirements

- **Map API interactions with frontend components**: Identify what data is required and how it will be consumed.
- **Refer to API documentation** to align with expected behavior and response formats.
- **Define data flow and lifecycle**: Understand where data is coming from (DB, cache, third-party API) and how it should be processed.
- **Mock API for External Providers**: Some external APIs only provide a production environment without a sandbox or testing environment. In such cases, backend developers need to create **mock APIs** that simulate the responses of these external services. These mock APIs return **static dummy records** that resemble the real API responses, ensuring frontend and QA teams can test functionality without relying on external production endpoints. This approach improves **development efficiency** and prevents unnecessary API calls to production environments.

#### 1.3 Key Components of an API

APIs consist of multiple components that define how data is requested, sent, and processed. Understanding these parts ensures efficient API usage and integration.

**1. Endpoint**
- The URL that specifies where the API is hosted.
- Example: `https://api.example.com/users`

**2. HTTP Methods**
Defines the type of action being performed:
- `GET` – Retrieve data
- `POST` – Create new data
- `PUT` / `PATCH` – Update existing data
- `DELETE` – Remove data

**3. Query Parameters**
- Used to filter, sort, or modify API responses.
- Example: `GET /users?role=admin&limit=10`

**4. Path Parameters**
- Used to specify a particular resource.
- Example: `GET /users/{user_id}` → `GET /users/12345`

**5. Request Body**
- Contains data sent in `POST`, `PUT`, or `PATCH` requests.
- Typically formatted as JSON.
```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

**6. Headers**
- Provide metadata such as content type and authentication.
```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer your_token_here"
}
```

**7. Authorization (Bearer Token)**
- Secure APIs require authentication using tokens.
- Example: `Authorization: Bearer <access_token>`

**8. API Response**
- The data returned from the server after processing a request.
- Includes:
  - Status Code (`200 OK`, `400 Bad Request`, etc.)
  - Response Body (JSON data or error message)

By understanding these components, developers can efficiently interact with APIs, debug issues, and build robust applications.

#### 1.3. Determine the Correct HTTP Methods and Endpoints

- Select the correct **HTTP method**:
  - `GET` → Fetch data
  - `POST` → Create a resource
  - `PUT` → Replace the entire resource with a new representation (meaning all fields are sent in the request body, even if they are not modified).
  - `PATCH` → Apply partial updates to a resource (meaning that only the fields that need to be changed are sent in the request body).
  - `DELETE` → Remove a resource

- Choose appropriate **resource-based URIs**.

**Proper API Endpoint Design**

When designing RESTful APIs, it's important to follow **clean and structured URL patterns** that align with **HTTP methods**, rather than including action-specific names in the endpoint itself.

**Incorrect API Design (Bad Practice)**
```
POST /users/create    → Creates a new user
GET  /users/list      → Retrieves a list of users
GET  /users/view/{id} → Retrieves details of a specific user
DELETE /users/remove/{id} → Deletes a specific user
```

**Why is this bad?**
- Redundant action-specific names (`create`, `list`, `view`, `remove`) clutter the URL.
- The **HTTP method already conveys the action**, so there's no need to repeat it in the endpoint.
- The URL should represent a **resource**, not an action.

**Correct API Design (Best Practice)**
```
POST   /users        → Creates a new user
GET    /users        → Retrieves a list of users
GET    /users/{id}   → Retrieves details of a specific user
DELETE /users/{id}   → Deletes a specific user
```

**Why is this better?**
- **Cleaner and more readable** URLs.
- The HTTP method defines the operation (`POST` for create, `GET` for fetch, `DELETE` for remove, etc.).

---

### 2. Understanding Suitable HTTP Status Codes

#### 1.1 Success Codes

**200 OK** – The request was successful, and the response contains the expected data.
```json
{ "message": "User details retrieved successfully" }
```

**201 Created** – A new resource has been successfully created.
```json
{ "message": "User account created successfully" }
```

**202 Accepted** – The request has been accepted for processing but is not yet completed. Used in cases like AWS Lambda async tasks or background batch jobs.
> *Example Use Case:* When a user submits a request to process and export large data, the API sends a request to an AWS Batch job. The server accepts the request but informs the client that processing is still ongoing.

#### 1.2 Client Error Codes

**400 Bad Request** – The request is malformed or missing the required parameters.
```json
{ "message": "Invalid request format. 'email' field is required." }
```

**401 Unauthorized** – The request lacks valid authentication credentials.
```json
{ "message": "Unauthorized. Please log in to access this resource." }
```

**403 Forbidden** – The user is authenticated but does not have permission to access the resource.
```json
{ "message": "You do not have permission to perform this action." }
```

**404 Not Found** – The requested resource does not exist.
```json
{ "message": "The requested order ID does not exist." }
```

**409 Conflict** – The request conflicts with the current state of the resource.
```json
{ "message": "Email address is already registered. Please use a different email." }
```

**410 Gone** – The resource existed before but is no longer available.
> *Example Use Case:* A cart is valid for 10 minutes. If a user tries to access it after expiry, the API returns `410 Gone`.
```json
{ "message": "Cart has expired. Please create a new cart." }
```

**424 Failed Dependency** – The request failed because a dependent external service encountered an error.
> *Example Use Case:* A crypto exchange API (BTSE) fails to provide conversion rates, affecting the API that depends on it.
```json
{ "message": "There was an error fetching the latest conversion rates. Please try again later." }
```

**429 Too Many Requests** – The user has sent too many requests in a short period and often returns when AWS API throttling limits are exceeded.
> *Example Use Case:* A client sends too many requests to an AWS API Gateway within a short period, hitting the rate limit.
```json
{ "message": "Too many requests. Please try again after some time." }
```

#### 1.3 Server Error Codes

**500 Internal Server Error** – A generic error when the server encounters an unexpected condition.
```json
{ "message": "An unexpected error occurred. Please try again later." }
```

**502 Bad Gateway** – The server received an invalid response from an upstream service.
> *Example Use Case:* When an API gateway calls a downstream service, but that service is down or returning invalid responses.

**503 Service Unavailable** – The server is temporarily unavailable due to maintenance or high load.
```json
{ "message": "Service is temporarily unavailable. Please try again later." }
```

**504 Gateway Timeout** – The server acting as a gateway or proxy did not receive a response in time.
```json
{ "message": "Gateway Timeout. The request took too long to process." }
```

> **Note:** Status codes highlighted with the 💡 symbol, such as **202, 424, 410**, etc., are often underutilized but play a crucial role in API design. It is essential to understand and practice using them appropriately for their intended use cases.

---

## Database Management

Effective database management is crucial for ensuring the performance, scalability, and security of backend applications. Below are key considerations for managing databases as a backend developer.

### 1. Identifying Suitable Databases

Choosing the right database depends on the application's requirements.

**Relational Databases (SQL)** – Suitable for structured data with relationships.
- **Amazon RDS (PostgreSQL, MySQL, etc.)** – Managed relational database with automatic backups and scaling.
- **Amazon Aurora** – A high-performance relational database with serverless and provisioned options.
- **Read Replicas** – Used to improve read scalability by distributing read requests.

**NoSQL Databases** – Suitable for unstructured or semi-structured data with high scalability needs.
- **DynamoDB** – AWS's managed NoSQL database, ideal for high-traffic, low-latency applications.
- **Best for:** Key-value and document data models, event-driven architectures.

**When to Use SQL vs NoSQL:**
| Criteria | SQL (RDS/Aurora) | NoSQL (DynamoDB) |
|---|---|---|
| Data structure | Structured, relational | Unstructured/semi-structured |
| Scalability | Vertical + Read Replicas | Horizontal auto-scaling |
| Transactions | ACID compliant | Limited (DynamoDB Transactions) |
| Query flexibility | Complex JOINs | Simple key-value or index queries |

### 2. Connection Handling

Efficient connection handling prevents performance bottlenecks and improves scalability.

- **PostgreSQL & MySQL** – Requires connection pooling to avoid excessive open connections.
  - Use connection pooling and **Knex.js** (query builder) instead of raw queries to prevent SQL injection and optimize performance.
  - Avoid opening and closing connections frequently; maintain a pool to efficiently manage connections.

- **DynamoDB** – This does not require connection management as it operates over HTTP requests.

- **Amazon RDS Data API** – Automatically handles connections for SQL-based queries, eliminating the need for connection pooling.

---

## Security

### 1. Authentication of APIs

Every **private API** should have a robust authentication mechanism to prevent unauthorized access.

**Reference:** Cognito Custom Authentication Flow

- **Cognito Authentication** – APIs can validate Cognito user tokens using an **Auth ARN**.
- **Custom Authentication** – Based on application requirements, APIs can implement JWT-based authentication or OAuth.

**Authorization & Role-Based Access Control**
- Ensure that APIs enforce permission checks at the backend, even if the frontend restricts UI actions.
- **Example:** A user may only have *view* access on the front end (edit buttons disabled), but API-level validation should prevent users from performing *update or delete actions* using external tools like Postman or penetration testing scripts.
- Consider using the AWS Service, **Amazon Verified Permissions**.

---

## Testing Code

Testing APIs thoroughly ensures stability and prevents regressions. A structured approach to testing includes:

### 1. Manual API Testing Strategy

**Initial Request Validation**
- Send a request with no parameters and observe the API response.
- Test the endpoint with an incorrect HTTP method (e.g., sending a `GET` request to a `POST` endpoint).

**Negative Testing**
- Provide incomplete or incorrect request bodies.
- Omit required fields and check if the API returns proper validation errors.
- Send invalid data formats (e.g., sending a string instead of an integer).

**Security Testing**
- Attempt unauthorized access to secured endpoints.
- Test privilege escalation scenarios by modifying request parameters.

**Success Path Validation**
- Finally, send a valid request and verify the response is as expected.

By following these testing strategies, developers can identify potential API failures early and ensure the system behaves reliably under various conditions.

### 2. Automated API Testing Strategy

**Reference:** Automated API Testing with Dredd Hooks

To ensure reliable, maintainable, and efficient API tests, automation must avoid static test data and instead use dynamic testing techniques. Dredd, combined with hooks, allows API validation against OpenAPI (Swagger) specifications while handling authentication, dynamic IDs, and test cleanup.

#### Avoiding Static Responses in API Testing

When writing API code, it's crucial to ensure that API tests validate the entire API flow, including database interactions, authentication, and external integrations. However, a bad practice is to return static responses when a specific test input is detected, completely bypassing actual API execution.

**Bad Practice: Static Response for API Tests**
```js
// Lambda Handler
exports.handler = async (event) => {
  //  Lambda API Execution Starts
  const requestBody = JSON.parse(event.body)

  // Bad Practice: Returning a static response for a specific test email
  if (requestBody.email_address === 'name.api-testing@7edge.com') {
    return {
      statusCode: 201,
      headers: headers,
      body: JSON.stringify({
        message: "User created successfully",
        data: {
          //  This User is not created, It's static
          //  That is why, Dynamic API tests cannot be made using this ID
          "id": "560d592e-f3d2-40ee-9fc5-870601bd7004",
          "email_address": "name.api-testing@7edge.com",
          "country_code": "+91",
          "phone_number": "9199199992"
        },
      }),
    }
  }

  // Continue API flow: Add user to Cognito
  const cognitoResponse = await createUserInCognito(requestBody)

  // Store user in the database
  const user = await db.insertUser(requestBody, cognitoResponse.userId)

  //  Send a welcome email
  await sendEmail(requestBody.email_address)
}
```

**⚠️ Why is this a Bad Practice?**
- The API never interacts with the database, Cognito, or other integrations for test cases.
- The same static request data always gets the same static response, meaning the API is not actually tested.
- The API test doesn't validate real execution paths, error handling, or dependencies, making the test useless.
- It leads to **false positives**, as test cases will always pass even if the actual API logic is broken.

**Good Practice: Allow API to Run Normally During Tests**

**Good Practice: Using DEBUG Mode for Controlled Skipping**

Instead of bypassing the API logic completely, a **debug mode** can be implemented to conditionally skip only certain operations, ensuring that most of the API is still executed while preventing unnecessary failures in test environments.

**🛠 Example Uses of DEBUG Mode:**
- This allows testers to bypass OTP verification during automated testing, avoiding issues with real-time OTP delivery.

**Key Takeaways**
- Never hardcode static responses for specific test inputs in production code.
- Always allow the API to execute its full flow during testing.
- Use **DEBUG mode** to conditionally skip only specific non-critical operations (e.g., OTP sending) without bypassing the entire API logic.
- Prefer dynamic test data that reflects real usage scenarios.

---

