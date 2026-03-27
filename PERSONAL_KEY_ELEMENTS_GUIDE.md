# Key Elements Guide

This file is a simple memory guide for the project.
Each item answers only two questions:
- What is it?
- Why do we use it?

## Playwright

**What:** A tool that automates the browser.
**Why:** It lets us simulate real user actions like clicking, typing, logging in, and transferring money.

## TypeScript

**What:** JavaScript with types.
**Why:** It helps catch mistakes earlier and makes the code easier to maintain.

## UI Test

**What:** A test that works through the website screens like a real user.
**Why:** It proves the visible user flow works end to end.

## API Test

**What:** A test or validation that talks directly to the backend service.
**Why:** It is faster and more reliable for checking data like account details and balances.

## curl

**What:** A command-line tool that sends HTTP requests.
**Why:** The assignment required creating the new checking account using a curl command, so we used it to call the backend directly.

## Swagger

**What:** API documentation that lists the available endpoints and request/response structure.
**Why:** It tells us which backend calls exist and how to use them correctly.

## End-to-End Flow

**What:** One full business scenario from beginning to end.
**Why:** It proves the core banking journey works across UI and backend together.

## Negative Tests

**What:** Tests for invalid or wrong behavior.
**Why:** They prove the system handles errors correctly and does not allow bad actions.

## Assertion

**What:** A check inside the test.
**Why:** It verifies that the result is correct, for example status code, UI text, account type, or balance.

## Page Object Model (POM)

**What:** A way to organize UI code by page.
**Why:** It keeps the tests cleaner, more reusable, and easier to maintain.

## Page Objects

**What:** Classes like `LoginPage`, `RegisterPage`, and `TransferFundsPage`.
**Why:** They store UI actions in one place instead of repeating selectors in every test.

## API Client

**What:** A class that wraps backend requests.
**Why:** It keeps API calls centralized and makes the tests easier to read.

## Reusable Assertions

**What:** Shared validation helpers for accounts and customers.
**Why:** They reduce duplication and keep validations consistent.

## Test Data Factory

**What:** A helper that creates new user data for tests.
**Why:** It avoids hardcoding the same user values and helps generate fresh data.

## Environment Configuration

**What:** Central configuration for URLs, timeouts, headless mode, and numeric values.
**Why:** It avoids hardcoding values in many files and makes the project easier to move between machines or environments.

## `playwright.config.ts`

**What:** The main Playwright configuration file.
**Why:** It controls timeout values, reporters, browser project, retries, screenshots, videos, and trace behavior.

## Reporting

**What:** The test output formats like list, HTML, JSON, and JUnit.
**Why:** They help humans and CI systems understand the test results.

## Debug Artifacts

**What:** Screenshots, videos, and traces captured on failure.
**Why:** They help investigate why a test failed.

## Bootstrap Script

**What:** A PowerShell helper script that installs dependencies, installs Chromium, and runs the suite.
**Why:** It gives a simple one-command starting point for a new machine or a reviewer.

## Docker

**What:** A containerized environment for running the project.
**Why:** It helps the project run the same way on different machines.

## `docker-compose.yml`

**What:** A file that defines how to run the Docker service for this project.
**Why:** It makes container execution easy with one command.

## CI

**What:** Continuous Integration, meaning automated runs in a pipeline.
**Why:** It helps verify the tests automatically after code changes.

## GitHub Actions

**What:** GitHub's built-in CI system.
**Why:** It can run the framework automatically from the repository.

## Jenkins

**What:** Another CI tool used a lot in companies.
**Why:** The project is structured so it can also be run there later.

## Dockerfile

**What:** The file that defines the Docker image.
**Why:** It installs Node, dependencies, Playwright, and the browser needed to run the tests in a container.

## Headless Mode

**What:** Running the browser without showing the UI on screen.
**Why:** It is useful for CI and automated runs.

## Trace / Screenshot / Video

**What:** Failure evidence collected by Playwright.
**Why:** They make debugging much easier than only reading a text error.

## Stability Hardening

**What:** Changes made to reduce flaky failures in a public shared environment.
**Why:** Public demo systems can be inconsistent, so stable automation needs retries, polling, and better waits.

## Security Verification Handling

**What:** A helper that waits for the public site's security check to finish.
**Why:** Without it, tests can fail because the site temporarily blocks normal page interaction.

## Single Worker Execution

**What:** Running one test worker instead of several in parallel.
**Why:** It reduces load and flakiness against a shared public environment.

## Why This Framework Structure Is Good

**What:** The project is split into tests, pages, API, config, assertions, and utilities.
**Why:** Each part has one responsibility, which makes the code easier to understand, update, and scale.

## Simple Interview Summary

You can say this:

"I built a Playwright TypeScript automation framework for ParaBank. I used page objects for UI actions, an API client for backend validation, and curl for the required account creation call. I added configuration management, reporting, Docker support, CI readiness, and stability improvements for a public shared environment."

## General Purpose of Key Elements

This section answers a slightly different question:
- not only why we used it in this project
- but also why people use it in general

## Why `curl` in General

**General purpose:** `curl` is used to send HTTP requests directly from the command line.
**Why people use it:** It is quick for testing APIs, debugging endpoints, scripting, and CI jobs without needing a browser or UI.
**Why it mattered here:** The assignment explicitly asked to create the new account with a `curl` command, so it showed direct backend interaction.

## Why Swagger in General

**General purpose:** Swagger documents the API and shows available endpoints, parameters, and response shapes.
**Why people use it:** It helps developers, testers, and automation engineers understand how to call the backend correctly.
**Why it mattered here:** It guided which ParaBank API endpoints were used in the tests.

## Why Playwright in General

**General purpose:** Playwright is used for browser automation and end-to-end testing.
**Why people use it:** It is good for validating real user flows across modern browsers with strong debugging support.
**Why it mattered here:** It handled registration, login, UI validation, transfer flow, and logout.

## Why API Validation in General

**General purpose:** API validation checks backend behavior directly.
**Why people use it:** It is usually faster, more stable, and more precise than checking everything through the UI.
**Why it mattered here:** It was the best way to validate customer id, account data, and balances.

## Why Page Object Model in General

**General purpose:** POM organizes UI code by page or feature.
**Why people use it:** It reduces duplication and makes tests easier to maintain when the UI changes.
**Why it mattered here:** It kept selectors and UI actions out of the test body so the test stayed readable.

## Why Docker in General

**General purpose:** Docker runs the project inside a consistent container.
**Why people use it:** It reduces machine-specific differences and makes setup easier across different computers.
**Why it mattered here:** It made the framework portable and easier for a reviewer or CI pipeline to run.

## Why CI in General

**General purpose:** CI runs checks automatically after code changes.
**Why people use it:** It gives quick feedback and helps catch issues early.
**Why it mattered here:** The framework is ready to run automatically in GitHub Actions and can be moved to Jenkins later.

## Why a Bootstrap Script in General

**General purpose:** A bootstrap script is a quick setup helper for a new machine.
**Why people use it:** It reduces onboarding friction and avoids manual setup mistakes.
**Why it mattered here:** It let a reviewer install dependencies, install Playwright, and run the project with one script.
