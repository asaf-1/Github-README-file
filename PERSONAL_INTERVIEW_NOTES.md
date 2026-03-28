# Personal Interview Notes

This file is for me, not for submission.

## Short Summary

I built a Playwright + TypeScript automation framework for the ParaBank assignment.
It covers one full end-to-end banking flow across UI, API, and `curl`, plus 3 negative tests.
I also made it runnable locally, in GitHub Actions, and in Docker.

## What I Built

Main flow:

1. Register user in UI
2. Log in in UI
3. Get customer id by API
4. Get existing account by API
5. Create a new checking account with `curl`
6. Verify the new account in UI
7. Transfer money in UI
8. Validate balances by API
9. Log out

Negative tests:

1. Invalid login
2. Registration password mismatch
3. Invalid account lookup by API

## Test Flows by Purpose

### Flow 1 of 4: Full Core Banking Flow

This is easier to remember if I divide it into stages:

Stage 1: User creation in UI

1. Open registration page
2. Register a brand-new user
3. Confirm registration success

Stage 2: Login validation in UI

1. Log out
2. Open login page
3. Log in with the same user
4. Confirm account overview is shown

Stage 3: Customer and existing account lookup by API

1. Call API login endpoint
2. Get customer id
3. Call customer accounts endpoint
4. Confirm the customer has one existing checking account
5. Call single account endpoint
6. Confirm account details are correct

Stage 4: New account creation by `curl`

1. Call `createAccount` through `curl`
2. Confirm status code `200`
3. Confirm a new checking account id is returned

Stage 5: New account verification through API and UI

1. Poll the API until both accounts exist
2. Confirm the new account belongs to the same customer
3. Open Accounts Overview in the UI
4. Confirm the new account id appears in the UI
5. Confirm the displayed balance matches the API balance

Stage 6: Transfer money in UI

1. Open Transfer Funds
2. Transfer money from the original account to the new account
3. Confirm transfer success message

Stage 7: Balance validation by API

1. Poll the API until balances update
2. Confirm source account balance decreased
3. Confirm target account balance increased
4. Confirm total money stayed consistent

Stage 8: Logout

1. Log out
2. Confirm user returns to login page

Expected result:

- the full user journey works end to end
- UI and backend stay consistent
- account creation and transfer both succeed
- final balances are correct

### Flow 2 of 4: Invalid Login Negative Test

Step to reproduce:

1. Open the login page
2. Enter an invalid username
3. Enter an invalid password
4. Click log in

Expected result:

- user is not authenticated
- login fails
- error state is shown on the login page

### Flow 3 of 4: Registration Password Mismatch Negative Test

Step to reproduce:

1. Open the registration page
2. Fill valid user details
3. Enter one password in the password field
4. Enter a different password in the repeated password field
5. Submit registration

Expected result:

- registration is blocked
- mismatch validation message is shown
- user account is not created

### Flow 4 of 4: Invalid Account Lookup by API

Step to reproduce:

1. Send an API request for a clearly non-existing account id

Expected result:

- response status is `400`
- response content type is plain text
- response body explains that the account was not found

## Project Structure Explanation

- `tests/e2e`
  Main happy-path business flow
- `tests/negative`
  Negative scenarios
- `src/pages`
  Page objects for UI actions
- `src/api`
  API client wrapper for Swagger-documented endpoints
- `src/utils/curl.ts`
  Runs the required `curl`-based account creation
- `src/assertions`
  Shared domain assertions
- `src/data`
  Dynamic user factory to avoid collisions in public environment
- `src/config`
  Environment config from `.env`

## Why I Designed It This Way

- I separated UI logic, API logic, and command-line logic so the tests stay readable and maintainable.
- I used dynamic users because ParaBank is a shared public site and hardcoded usernames would collide.
- I validated both UI and backend data so the tests check business correctness, not just screen text.
- I compared balances in cents to avoid floating-point issues.

## Swagger Explanation

Important point:

- The tests do not call the Swagger UI page directly.
- They call the real API endpoints that are documented by the Swagger page.
- So the implementation is Swagger-informed even though the Swagger UI is not part of the test flow.

Examples:

- `login/{username}/{password}`
- `customers/{customerId}/accounts`
- `accounts/{accountId}`
- `createAccount`

## Swagger Calls Used: What and Where

### `login/{username}/{password}`

What it was used for:

- to get the customer object and customer id after the UI login

Where it was used:

- in the core flow after the user logs in through the UI
- implemented in `src/api/bank-api.client.ts` in the `login()` method

Why it mattered:

- it connected the UI-created user to the backend customer id needed for the next API steps

### `customers/{customerId}/accounts`

What it was used for:

- to get the list of accounts for the customer
- first to get the original account
- later to verify the new account appears
- later to validate total balances

Where it was used:

- in the core flow after customer id retrieval
- implemented in `src/api/bank-api.client.ts` in the `getCustomerAccounts()` method

Why it mattered:

- it was the main endpoint for confirming how many accounts the customer has and whether the new one was created

### `accounts/{accountId}`

What it was used for:

- to get a single account's exact details
- to verify the original account
- to verify updated balances after transfer
- also in the negative API test with an invalid id

Where it was used:

- in the core flow and in the negative API test
- implemented in `src/api/bank-api.client.ts` in `getAccount()` and `getAccountResponse()`

Why it mattered:

- it gave exact account-level validation instead of only relying on the UI

### `createAccount`

What it was used for:

- to create the new checking account

Where it was used:

- in the core flow
- implemented in `src/utils/curl.ts` through `createCheckingAccountViaCurl()`

Why it mattered:

- this was an explicit assignment requirement and proved direct backend interaction using `curl`

## Why `curl` Was Used

The assignment explicitly asked to create a new checking account using a `curl` command.
Because of that, I did not replace it with a normal Playwright API request.
I implemented a helper that actually executes `curl` and captures the command, body, and status code.

## Docker Explanation

The Docker setup includes:

- Node runtime
- npm dependencies
- Playwright Chromium
- required Linux browser dependencies with `--with-deps`

I also verified it by running the full suite inside Docker, not just writing the Docker files.

## CI Explanation

I added GitHub Actions to:

1. checkout code
2. install dependencies
3. install Chromium
4. run typecheck and tests
5. upload artifacts

## Reporting and Debugging

The framework produces:

- list reporter
- HTML report
- JSON report
- JUnit XML report
- screenshots on failure
- video on failure
- trace on first retry

This is good enough for assignment quality and CI debugging.

## Important Glossary

- `Playwright`
  A browser automation tool. We use it to run UI tests, API checks, and automate browser actions.

- `TypeScript`
  A language built on top of JavaScript that adds types. It helps keep the code safer and easier to maintain.

- `UI`
  The user interface that the user sees and interacts with. In this assignment, that is the ParaBank website.

- `API`
  A way for systems to communicate directly without going through the UI. Here we used it to validate real backend data.

- `Swagger`
  The documentation interface for the API. It shows which endpoints exist, which parameters they expect, and what responses they return.

- `OpenAPI`
  The format/specification that describes the API structure. Swagger is usually the way this spec is presented and explored.

- `curl`
  A command-line tool that sends HTTP requests. In this assignment we used it to create a new account because that was an explicit requirement.

- `HTTP request`
  A request sent to a server. For example, `GET` to retrieve data or `POST` to create/send data.

- `HTTP response`
  The server's answer, including status code, headers, and body.

- `Status code`
  A code that shows whether the request succeeded or failed. For example, `200` means success and `400` means a request error.

- `Endpoint`
  A specific API address that performs a specific operation, such as getting an account or creating an account.

- `GET`
  A request type mainly used to retrieve information.

- `POST`
  A request type usually used to create an action or send data to the server.

- `JSON`
  A common data format used between systems. Most of the API responses in this project are JSON.

- `Page Object Model`
  A way to organize test automation so each UI page has its own file with clear actions. This improves readability and maintainability.

- `Assertion`
  A check that verifies the actual result matches the expected result.

- `Happy Path`
  The main valid scenario where everything works correctly.

- `Negative Test`
  A test that checks how the system behaves when something invalid happens, such as logging in with wrong credentials.

- `Dynamic test data`
  Test data that is created at runtime, such as a unique username for each run. This helps avoid collisions.

- `Backend`
  The server side and business logic. Not what the user sees, but what happens behind the scenes.

- `Frontend`
  What the user actually sees and interacts with in the browser.

- `Chromium`
  The browser used by the tests right now. It is the engine behind Chrome and similar browsers.

- `Headless`
  A mode where the browser runs without opening a visible window. Useful for CI and automated runs.

- `Reporter`
  A mechanism that generates test results, such as an HTML report, JSON, or JUnit XML.

- `Trace`
  A detailed recording of the test run that helps understand what happened when a test fails.

- `Screenshot on failure`
  An automatic screenshot taken when a test fails, to help understand the UI state.

- `Video on failure`
  A video of the test run in failure cases, to help inspect what happened.

- `Environment variables`
  Values like `BASE_URL` or `HEADLESS` that control behavior without changing code.

- `.env`
  A file where local environment variables can be defined.

- `Docker`
  A way to package the project with everything it needs so it can run consistently on other machines.

- `Container`
  An isolated runtime environment where the application or tests run.

- `Dockerfile`
  The file that defines how the container is built.

- `docker-compose`
  A tool that makes it easy to run containers with a simple command.

- `CI`
  Short for Continuous Integration. A system that runs checks automatically after code changes.

- `GitHub Actions`
  GitHub's CI system. We used it here to run the project in the cloud.

- `Jenkins`
  Another common CI system. This project could be connected to it in the future.

- `Git`
  A version control system used to track code changes.

- `GitHub`
  A hosting and collaboration platform for Git repositories, including code sharing and CI integration.

- `Repository`
  The project folder as managed in Git and GitHub.

- `README`
  The main documentation file that explains how to run the project, the design decisions, and the assumptions.

- `package.json`
  The file that defines the dependencies and scripts of a Node project.

- `package-lock.json`
  The file that locks dependency versions to keep installations consistent.

- `npm`
  The tool used to manage dependencies and run scripts in a Node project.

## Tradeoffs I Can Mention

- I used Chromium only for stability and simplicity.
- I did not add schema-validation libraries to keep dependencies lean.
- I did not reset the public database because this is a shared public environment.
- I used one full happy-path test because the assignment is centered on one business journey.

## Assumptions I Can Mention

- The public ParaBank environment is reachable
- Registration is allowed
- `curl` exists on the machine/container
- Opening a new account transfers `100.00` from the source account
- The create-account response is not always the final balance truth, so I verify with follow-up API calls

## If They Ask "How Would You Scale It?"

I would say:

- add more domain-specific API services
- add reusable components for common UI fragments
- add more tags like smoke/regression/api
- add multi-browser coverage
- add environment matrix in CI
- add schema/contract validation against the OpenAPI spec
- add authenticated fixtures to speed up non-registration tests

## If They Ask "How Do You Run It?"

Local:

```powershell
npm.cmd install
npx.cmd playwright install chromium
npm.cmd test
```

Docker:

```powershell
docker compose up --build playwright
```

## If They Ask "What Was Most Important?"

My answer:

The most important thing was making the test reliable in a public shared environment.
That is why I used dynamic user creation, strong assertions, API cross-checks, and Docker verification.

## If They Ask "What Would You Improve Next?"

My answer:

- add contract validation against the Swagger spec
- add more negative and edge-case scenarios
- add multi-browser coverage
- add reusable fixtures for faster runtime
- add richer reporting dashboards if the suite grows
