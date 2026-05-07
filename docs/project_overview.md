# Project Overview: Notes Automation Framework

## 1. What this project is

This repository is a hybrid automation framework for a Notes application. It includes:

- UI automation using Selenium WebDriver
- API automation using `requests`
- Hybrid end-to-end tests combining UI and API
- Test data generation and configuration management
- Reporting support via Allure

The goal of the framework is to validate user authentication, note creation, retrieval, update, deletion, and data consistency between UI and API.

---

## 2. Project structure explained from scratch

### Root files

- `README.md`
  - User-facing guide for setup and running tests.
- `requirements.txt`
  - Python libraries required by the project.
- `pytest.ini`
  - Pytest configuration and marker registration.
- `Jenkinsfile`
  - CI/CD pipeline definition for Jenkins.
- `conftest.py`
  - Global Pytest hooks used by all tests.

### Main folders

- `api/`
  - `api_client.py` : HTTP client for API requests.
  - `endpoints.py` : Named API endpoint paths.

- `config/`
  - `config.yaml` : Framework settings, server URLs, browser settings, and test user defaults.
  - `environment.py` : Environment helper to load environment variables if used.

- `docs/`
  - Documentation files, including this overview and test planning files.

- `fixtures/`
  - `driver_fixture.py` : Browser driver creation for Selenium.
  - `api_fixture.py` : API test setup and cleanup helpers.
  - `conftest.py` : Pytest fixtures for browser sessions, API sessions, and hybrid flows.

- `pages/`
  - `base_page.py` : Common Selenium page operations, waits, self-healing, screenshot support.
  - `login_page.py` : Page Object Model for login actions.
  - `register_page.py` : Page Object Model for registration.
  - `notes_page.py` : Page Object Model for notes dashboard actions.

- `tests/`
  - `test_login.py` : UI login tests.
  - `test_notes_api.py` : Notes API tests.
  - `test_notes_ui.py` : Notes UI tests.
  - `test_e2e.py` : Hybrid end-to-end consistency tests.

- `utils/`
  - `config_reader.py` : YAML loader and config getters.
  - `helpers.py` : Data generation, screenshots, utility methods.
  - `logger.py` : Structured logging for tests.

- `reports/`
  - Generated test artifacts: screenshots, Allure results, logs.

---

## 3. What the tests cover

### Total test cases

This project contains the following test case sets:

- `tests/test_login.py` : 8 UI login-related test cases
- `tests/test_notes_ui.py` : 6 UI note management test cases
- `tests/test_notes_api.py` : 11 API test cases
- `tests/test_e2e.py` : 4 hybrid UI+API end-to-end test cases

**Total: 29 test cases**

### Test coverage by area

- Authentication / login
- Notes creation and display in UI
- Notes CRUD operations via API
- Negative flows for invalid login and unauthorized API access
- Performance checks for API response times and UI page load times
- Hybrid validation to confirm UI and API layers remain consistent

---

## 4. How the test framework works

### Pytest

This project uses `pytest` as the test runner. Each file in `tests/` defines a class with test functions. When you run `pytest`, it discovers and executes these tests.

### Fixtures

Fixtures are reusable setup/teardown pieces that provide browser drivers, API clients, and authenticated users.

Key fixtures:

- `driver`
  - Creates a Selenium browser session for every UI test.
  - Uses `create_driver()` from `fixtures/driver_fixture.py`.
  - Captures screenshots on failure.

- `api_client`
  - Creates an unauthenticated API client.

- `authenticated_api`
  - Registers a test user via API, logs in, and provides an authenticated `APIClient`.
  - Cleans up the user after the test.

- `logged_in_ui`
  - Creates a user with API, then logs in through the UI.
  - Returns `driver`, `login_page`, `notes_page`, and test user details.

- `e2e_setup`
  - Builds a hybrid session with both browser and API client.
  - Used for tests that must verify UI and API together.

### Page Object Model (POM)

The framework uses POM to separate test logic from page interaction details.

- `BasePage` contains shared browser actions, waits, and utilities.
- `LoginPage` contains locators and actions for the login page.
- `NotesPage` contains locators and actions for note creation, deletion, and listing.
- `RegisterPage` contains locators and actions for registration.

This makes tests readable and easier to maintain.

---

## 5. Why specific locators are used

Locators are the way Selenium finds elements on a webpage.

### Primary locator strategy: `data-testid`

Most locators use `data-testid` attributes, such as:

- `input[data-testid='login-email']`
- `button[data-testid='login-submit']`
- `[data-testid='note-card']`

**Why this is used:**

- `data-testid` is usually stable and designed for automated tests.
- It does not depend on visible text, CSS classes, or page layout.
- It avoids brittle selectors that break when the UI changes.

### Secondary locator strategy: CSS selectors

Examples:

- `button[data-testid='note-submit']`
- `input[data-testid='note-title']`

**Why CSS selectors:**

- Fast and easy for Selenium to evaluate.
- Works well when the element has stable attributes.
- More readable than complex XPath in many cases.

### Fallback locator strategy: XPath

Fallback locators are used when the primary `data-testid` or CSS selector fails:

- `//input[@type='email' or @name='email' or @placeholder='Email address']`
- `//button[@type='submit' or contains(text(), 'Login')]`

**Why XPath fallback:**

- XPath can express more complex element relationships.
- It can match by `type`, `name`, placeholder text, or button text.
- Useful when the page structure changes slightly.

### Why not use other locators?

- `By.ID` and `By.NAME` are only used when a stable identifier exists.
- Absolute XPath is avoided because it breaks if any part of the page layout changes.
- Locator strategies based only on text can fail if the label text changes or is localized.
- `By.CLASS_NAME` is avoided as a primary locator when CSS classes are dynamic or used for styling only.

### Self-healing locators

The `BasePage` includes a self-healing method. If a locator fails, it tries alternative locators automatically.

This is useful because:

- Web applications can change over time.
- Small UI updates should not break every test.
- It makes the framework resilient and easier to maintain.

---

## 6. Detailed explanation of key files

### `pages/base_page.py`

Contains the shared browser logic for all pages:

- `find_element()` / `find_elements()`
- `click()` with retry and JS click fallback
- `type_text()` with clear before typing
- `wait_for_element_visible()` / `wait_for_element_invisible()`
- `navigate_to()` and `refresh_page()`
- `get_page_load_time()` for performance checks
- Self-healing locator fallback logic

### `pages/login_page.py`

Handles login page flows:

- Navigating to login page
- Entering email and password
- Clicking login
- Verifying login success
- Detecting login page elements and error messages

### `pages/notes_page.py`

Handles the notes dashboard:

- Creating a note with title, description, and category
- Reading all note titles from the dashboard
- Verifying note presence
- Deleting a note by title
- Clicking logout
- Refreshing page and measuring page load time

### `api/api_client.py`

Handles API interaction:

- Register user
- Login user and store auth token
- Create, read, update, delete notes
- Health check and profile retrieval
- Attaching API responses to Allure reports

### `api/endpoints.py`

Defines endpoint paths in one place so tests and client code do not hard-code URLs.

### `fixtures/driver_fixture.py`

Creates browser drivers using `webdriver-manager`:

- Chrome (default)
- Edge
- Firefox support is present but not fully used in tests

It also configures:

- headless mode
- window size
- implicit waits
- page load timeout

### `fixtures/api_fixture.py`

Helps create a reusable API client, register a test user, authenticate, and clean up after tests.

### `fixtures/conftest.py`

Defines Pytest fixtures used by tests:

- `driver`
- `api_client`
- `authenticated_api`
- `logged_in_ui`
- `e2e_setup`

Also contains the hook to capture screenshot on failure.

### `utils/config_reader.py`

Loads `config/config.yaml` once and provides helper functions to access configuration sections.

### `utils/helpers.py`

Provides utility functions for:

- Random email generation
- Note title/description generation
- Screenshot capture
- Timing measurement
- Placeholder AI support for test data and failure analysis

---

## 7. How the tests are organized by feature

### `tests/test_login.py`

Covers login functionality:

- `test_valid_login`
- `test_login_redirects_to_notes`
- `test_invalid_email`
- `test_invalid_password`
- `test_empty_email`
- `test_empty_password`
- `test_both_fields_empty`
- `test_login_page_elements_displayed`

### `tests/test_notes_ui.py`

Covers note management in the UI:

- `test_create_note_home_category`
- `test_create_note_work_category`
- `test_create_note_personal_category`
- `test_note_appears_instantly_after_creation`
- `test_multiple_notes_creation`
- `test_delete_note_via_ui`

### `tests/test_notes_api.py`

Covers Notes REST API endpoints:

- `test_api_health_check`
- `test_get_all_notes`
- `test_get_notes_returns_list`
- `test_create_note_api`
- `test_get_note_by_id`
- `test_delete_note_api`
- `test_update_note_api`
- `test_get_notes_response_time`
- `test_create_note_response_time`
- `test_get_notes_without_auth`
- `test_delete_nonexistent_note`

### `tests/test_e2e.py`

Covers hybrid UI + API consistency:

- `test_ui_create_api_verify`
- `test_api_delete_ui_verify`
- `test_full_crud_hybrid`
- `test_ui_page_load_performance`

---

## 8. How to run tests from scratch

### 1. Set up Python environment

```powershell
cd C:\Users\rakes\OneDrive\Desktop\QA_Final_Rakhi\QA_Final
python -m venv myenv
myenv\Scripts\activate
pip install -r requirements.txt
```

### 2. Run a full test suite

```powershell
pytest tests/ -v
```

### 3. Run only UI tests

```powershell
pytest tests/test_login.py tests/test_notes_ui.py -v
```

### 4. Run only API tests

```powershell
pytest tests/test_notes_api.py -v
```

### 5. Run hybrid E2E tests

```powershell
pytest tests/test_e2e.py -v
```

### 6. Run with Allure reporting

```powershell
pytest tests/ -v --alluredir=reports/allure-results
allure serve reports/allure-results
```

> If your browser is not visible, set `headless: true` in `config/config.yaml`.

---

## 9. Why this framework is built this way

### Clear separation of concerns

- Test logic lives in `tests/`
- Page interaction lives in `pages/`
- API logic lives in `api/`
- Setup and cleanup lives in `fixtures/`
- Configuration lives in `config/`
- Utility helpers live in `utils/`

This separation makes the project easier to understand and maintain.

### Reliable locators

The framework uses stable locators first, then fallback locators only when necessary. This reduces test breakage when the UI changes.

### Reusable test setup

Fixtures create users and browser sessions automatically so each test can start from a clean state.

### Hybrid verification

The hybrid tests prove that the UI and API are not only working individually, but also staying in sync. This is a strong validation strategy for modern web apps.

---

## 10. Final summary

This project is a complete test automation framework for a Notes application. It includes:

- 29 test cases
- UI, API, and hybrid end-to-end coverage
- Structured Page Object Model implementation
- Resilient locator strategy using `data-testid`, CSS, and XPath fallback
- Reusable fixtures for browser and API setup
- Configurable behavior through `config/config.yaml`

If you want, I can also create a separate `quick_start.md` that explains each step like a beginner tutorial from zero knowledge.
