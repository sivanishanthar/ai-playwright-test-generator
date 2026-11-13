# AI Playwright Test Generator

A standalone Node.js ES-module command-line project, separate from `ai-task-assistant`.

## Step 1: project foundation

The CLI accepts a natural-language test requirement, validates it, and prints it back. OpenAI, dotenv, and Playwright dependencies are installed for the next step. The capture CLI does not call OpenAI. The separate generation script described below calls OpenAI and saves a test for review. Neither script launches a browser.

## Setup

Use Node.js 22 or later and npm.

```bash
cd ai-playwright-test-generator
npm ci
```

Optional preparation for future AI integration:

```bash
cp .env.example .env
```

Add your own `OPENAI_API_KEY` to `.env` for generation. No key is needed for the capture CLI or unit tests. Keep credentials out of source files; `.env` files are ignored and `.env.example` contains no secrets.

## Run

Interactive prompt:

```bash
npm start
```

Or supply the requirement directly:

```bash
npm start -- "Verify that a user can log in with valid credentials"
```

Blank requirements exit with status 1.

## Test

```bash
npm test
```

Tests use Node's built-in test runner and cover validation, argument input, prompted input, and CLI failure behavior. They run locally without API requests or browser downloads.

## Dependencies and next step

- `openai`: official OpenAI JavaScript SDK ([documentation](https://developers.openai.com/api/docs/libraries)).
- `dotenv`: loads local environment configuration.
- `playwright` and `@playwright/test` (development): browser automation and the test runner for generated tests.

The separate generation script below uses a login requirement; the capture CLI remains unchanged. Browser binaries are not needed for this setup; install them with `npx playwright install chromium` when browser execution is added. Review generated code before running it.

No Git repository, commit, remote, or push is created by this setup.

## Generate and safely save a test

Set OPENAI_API_KEY locally in your environment or .env, then run:

```bash
node src/testAgent.js
# Or choose another unused filename:
node src/testAgent.js login-alternative.spec.js
```

The generation script calls OpenAI and saves to automatedTest/generated-tests/login.spec.js under the current working directory. Run it from this project directory. The Step 1 npm start CLI still captures requirements only.

Saving accepts only simple names ending in .spec.js, creates private files, refuses overwrites and existing symlinks, and rejects a symlinked output directory. Keep the local project directory under your control while saving. Generated code is never executed automatically or printed by this script. Review it before running: model instructions reduce invented details but do not validate generated code.

Generated tests should read BASE_URL (the full entry URL), TEST_USERNAME and TEST_PASSWORD from the runtime environment. Unknown labels, selectors and success checks must have TODOs and configuration variables; incomplete tests should skip before interacting with the page. Supply those details before running with Playwright. Never commit local credentials or secret-bearing generated files.

Unit tests use mocked AI responses and temporary directories; npm test makes no API calls and launches no browsers.

## Generate from your own requirement

```bash
npm start -- --generate --output login.spec.js "Verify successful login"
# Prompt for the requirement:
npm start -- --generate
```

Generation explicitly calls OpenAI and requires your local OPENAI_API_KEY. Without --generate, the CLI keeps its original capture-only behavior. --output accepts an unused simple .spec.js filename; omitted filenames use a unique ID. Output stays in automatedTest/generated-tests under the current project directory. Generation mode does not print your requirement, generated source, or API error details. Review the saved file and configure its TODOs before executing it.

## Run the verified ParaBank login test

The reviewed file automatedTest/smoke_test/auth/login.spec.js uses the live ParaBank login input names and button, with the overview account table as its success check.

Set BASE_URL to https://parabank.parasoft.com/parabank/index.htm in your local .env, plus TEST_USERNAME and TEST_PASSWORD for an existing ParaBank test account. Add credentials locally; do not share them in chat or commit them. Missing configuration causes the test to skip.

```bash
npx playwright install chromium
npx playwright test automatedTest/smoke_test/auth/login.spec.js --grep SM-02
# To watch the browser:
npx playwright test automatedTest/smoke_test/auth/login.spec.js --grep SM-02 --headed
```

The Playwright configuration loads local environment settings and keeps traces, screenshots and video disabled. Other generated files still require individual review before execution. No account registration or login has been performed as part of this setup.

## Syntax validation and invalid login

Every save now checks JavaScript syntax as an ES module with Node's --check, without executing source or resolving its imports. Malformed output is rejected before creating the output directory or file. Diagnostics never print generated source. This checks syntax only: review application behavior and selectors before running any generated test.

For negative login coverage, configure TEST_INVALID_USERNAME and TEST_INVALID_PASSWORD locally with intentionally invalid credentials for your ParaBank environment. They are separate from the successful-login credentials and have no fallback. Missing configuration skips the test.

```bash
npx playwright test automatedTest/smoke_test/auth/login.spec.js --grep SM-03 --headed
```

The test requires the explicit authentication error, keeps the login form visible, and verifies no account table appears. Unit tests do not call OpenAI or ParaBank.

## ParaBank test plan and data setup

See [the test plan](docs/TEST_PLAN.md) for prioritized smoke, regression, end-to-end, API and performance coverage, and [test data setup](docs/TEST_DATA.md) for preparation and ownership checks.

Use [config/parabank.env.example](config/parabank.env.example) as a reference to add missing keys to your existing local .env. Do not overwrite existing credentials.

```bash
npm run check:test-data -- login
# Configuration groundwork for future transfer/ownership tests:
npm run check:test-data -- transfer
```

These commands report only variable names and generic problems. They do not contact ParaBank, register accounts, validate credentials or move funds. State-changing tests and performance execution remain future work as listed in the plan.

## Read-only smoke suite

```bash
npm run check:test-data -- login
npm run test:smoke -- --headed
```

Five reviewed tests cover home/login controls, successful login, invalid login, overview/detail consistency and logout/protected overview access. The valid account needs at least one existing bank account; no accounts or transactions are created. Tests run serially with isolated browser contexts. Missing configuration skips the relevant test; every required smoke case must pass without skips. Unreviewed generated drafts are excluded from this command.

## Test folders

- `automatedTest/unit_test/`: Node.js unit tests (`npm test`).
- `automatedTest/smoke_test/`: reviewed ParaBank browser smoke tests (`npm run test:smoke`).
- `automatedTest/generated-tests/`: generated drafts awaiting review; excluded from default Playwright discovery.

The successful-login smoke file is now `automatedTest/smoke_test/auth/login.spec.js`. Default Playwright commands discover reviewed smoke, regression, E2E and API folders; generated drafts and performance scripts are excluded. Generation continues saving drafts into `automatedTest/generated-tests/`; review and promote a draft to the appropriate suite before running it.

## Test organization

See [test organization](docs/TEST_ORGANIZATION.md) for the folder layout, suite commands and tagging rules. Smoke tests are grouped by feature and are also included in regression via tags, without duplicate copies. `automatedTest/support/` holds shared browser helpers. Reserved regression, E2E, API and performance folders are explicitly marked as planned.

```bash
npm run test:unit
npm run test:smoke -- --headed
npm run test:regression -- --list
```

The regression selection contains five smoke cases plus four authentication-validation/recovery cases (nine total); other feature coverage remains planned.

## Authentication regression

```bash
npm run test:regression -- --headed
```

Nine cases: five smoke plus empty login fields (both, username only, password only) and valid-login recovery after an invalid attempt. Each rejected submission must show the expected error and expose no authenticated account table or logout link.
