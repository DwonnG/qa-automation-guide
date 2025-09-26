# QA Automation Quick Reference Guide

## **Table of Contents**
- [API Testing Essentials](#api-testing-essentials)
- [UI Testing Essentials](#ui-testing-essentials)  
- [Test Plan Structure](#test-plan-structure)
- [4 Testing Pillars Framework](#4-testing-pillars-framework)
- [Test Data & Fixtures](#test-data--fixtures)
- [Common Patterns](#common-patterns)
- [Debugging Test Failures](#debugging-test-failures)

---

## **API Testing Essentials**

### **API Test Project Structure**

<details open>
<summary>API project structure</summary>

```
api_tests/
├── conftest.py                 # Shared fixtures and pytest configuration
├── api/                        # API client and base classes
│   ├── __init__.py
│   ├── base_client.py         # Base API client with common methods
│   ├── auth_client.py         # Authentication API methods
│   ├── user_client.py         # User management API methods
│   └── product_client.py      # Product API methods
├── tests/                      # Test files organized by API domain
│   ├── __init__.py
│   ├── test_authentication.py # Auth endpoint tests
│   ├── test_users.py          # User CRUD tests
│   ├── test_products.py       # Product API tests
│   └── test_integration.py    # Cross-API integration tests
├── utils/                      # Helper utilities
│   ├── __init__.py
│   ├── test_data.py           # Test data generators and factories
│   ├── assertions.py          # Custom assertion helpers
│   └── helpers.py             # Common utility functions
├── config/                     # Configuration files
│   ├── __init__.py
│   ├── environments.py        # Environment-specific settings
│   └── endpoints.py           # API endpoint definitions
├── schemas/                    # JSON schema validation files
│   ├── user_schema.json       # User response schemas
│   └── product_schema.json    # Product response schemas
├── reports/                    # Test execution reports
└── requirements.txt           # Dependencies
```

</details>

### **Example API Client Structure**

<details open>
<summary>API client code examples</summary>

```python
# api/base_client.py
import requests
import logging

class BaseAPIClient:
    def __init__(self, base_url, headers=None):
        self.base_url = base_url
        self.session = requests.Session()
        if headers:
            self.session.headers.update(headers)
        self.logger = logging.getLogger(__name__)
    
    def get(self, endpoint, **kwargs):
        url = f"{self.base_url}{endpoint}"
        response = self.session.get(url, **kwargs)
        self.logger.info(f"GET {url} - Status: {response.status_code}")
        return response
    
    def post(self, endpoint, **kwargs):
        url = f"{self.base_url}{endpoint}"
        response = self.session.post(url, **kwargs)
        self.logger.info(f"POST {url} - Status: {response.status_code}")
        return response

# api/user_client.py
from .base_client import BaseAPIClient

class UserAPIClient(BaseAPIClient):
    def create_user(self, user_data):
        return self.post("/api/users", json=user_data)
    
    def get_user(self, user_id):
        return self.get(f"/api/users/{user_id}")
    
    def update_user(self, user_id, user_data):
        return self.put(f"/api/users/{user_id}", json=user_data)
    
    def delete_user(self, user_id):
        return self.delete(f"/api/users/{user_id}")
```

</details>

### **Basic Request Patterns**

<details open>
<summary>Basic API request patterns</summary>

```python
import requests, pytest

# CRUD Operations
response = requests.get(f"{base_url}/api/users/{user_id}")
response = requests.post(f"{base_url}/api/users", json=payload)
response = requests.put(f"{base_url}/api/users/{user_id}", json=payload)
response = requests.delete(f"{base_url}/api/users/{user_id}")

# Common Assertions
assert response.status_code == 200
assert response.json()["field"] == expected_value
assert response.elapsed.total_seconds() < 2.0
```

</details>

### **Test Structure**

<details open>
API test structure example</summary>

```python
class TestAPI:
    @pytest.fixture(autouse=True)
    def setup(self):
        self.base_url = "https://api.example.com"
        self.headers = {"Authorization": "Bearer token"}
    
    def test_create_user_success(self):
        payload = {"name": "Test", "email": "test@example.com"}
        response = requests.post(f"{self.base_url}/users", json=payload, headers=self.headers)
        assert response.status_code == 201
```

</details>

## **UI Testing Essentials**

### **UI Test Project Structure**

<details open>
UI project structure</summary>

```
tests/
├── conftest.py                 # Shared fixtures and configuration
├── pages/                      # Page Object Models
│   ├── __init__.py
│   ├── base_page.py           # Base page class with common methods
│   ├── login_page.py          # Login page specific methods
│   ├── dashboard_page.py      # Dashboard page methods
│   └── user_management_page.py
├── tests/                      # Test files organized by feature
│   ├── __init__.py
│   ├── test_authentication.py # Login/logout tests
│   ├── test_user_management.py # User CRUD tests
│   └── test_navigation.py     # UI navigation tests
├── utils/                      # Helper utilities
│   ├── __init__.py
│   ├── test_data.py           # Test data generators
│   ├── browser_factory.py     # Browser setup and configuration
│   └── helpers.py             # Common helper functions
├── config/                     # Configuration files
│   ├── __init__.py
│   ├── settings.py            # Environment settings
│   └── locators.py            # Element locators (optional)
├── reports/                    # Test execution reports
├── screenshots/                # Screenshots for failed tests
└── requirements.txt           # Dependencies
```

</details>

### **Example Page Object Model**

<details open>
Page Object Model examples</summary>

```python
# pages/base_page.py
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class BasePage:
    def __init__(self, driver):
        self.driver = driver
        self.wait = WebDriverWait(driver, 10)
    
    def click_element(self, locator):
        element = self.wait.until(EC.element_to_be_clickable(locator))
        element.click()
    
    def enter_text(self, locator, text):
        element = self.wait.until(EC.visibility_of_element_located(locator))
        element.clear()
        element.send_keys(text)

# pages/login_page.py
from selenium.webdriver.common.by import By
from .base_page import BasePage

class LoginPage(BasePage):
    USERNAME_FIELD = (By.ID, "username")
    PASSWORD_FIELD = (By.ID, "password")
    LOGIN_BUTTON = (By.CSS_SELECTOR, ".login-btn")
    ERROR_MESSAGE = (By.CLASS_NAME, "error-message")
    
    def login(self, username, password):
        self.enter_text(self.USERNAME_FIELD, username)
        self.enter_text(self.PASSWORD_FIELD, password)
        self.click_element(self.LOGIN_BUTTON)
    
    def get_error_message(self):
        return self.driver.find_element(*self.ERROR_MESSAGE).text
```

</details>

### **Selenium Basics**

<details open>
Selenium basics</summary>

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Element Location & Interaction
driver.find_element(By.ID, "username").send_keys("test")
driver.find_element(By.CSS_SELECTOR, ".login-btn").click()

# Waits (Always Use Explicit)
wait = WebDriverWait(driver, 10)
element = wait.until(EC.element_to_be_clickable((By.ID, "submit")))
```

</details>

## **Test Plan Structure**

### **Standard Test Plan Outline**

<details open>
<summary>Standard test plan outline</summary>

1. **Introduction**
2. **Scope**
   - 2.1 In Scope
   - 2.2 Out of Scope
3. **Testing Strategy**
   - 3.1 Unit Test
   - 3.2 Integration Test
   - 3.3 API End-to-End Test
   - 3.4 UI End-to-End Test
   - 3.5 Performance Test
   - 3.6 Regression Test
   - 3.7 Test Resources
   - 3.8 Entry Criteria
   - 3.9 Exit Criteria
4. **Features To Be Tested**
5. **Features Not To Be Tested**
6. **Configuring Environment For Testing**
7. **Diagrams**
8. **Technical Documents Summary**
9. **Test Cases**
10. **Resources & Responsibilities & Preconditions**
11. **Schedule**
12. **Dependencies**
13. **Risks/Assumptions**
14. **Point of Contacts**
15. **Reporting**
    - 15.1 Status Report
    - 15.2 Defect Tracking
16. **Glossary**
17. **Additional Resources**

</details>

### **Test Plan Template (Gherkin Format)**

<details open>
<summary>Gherkin test plan template</summary>

```gherkin
# Feature: User Authentication
# Stakeholders: Dev, QA, Product

## Test Strategy
- **Unit Tests**: Input validation, business logic (Dev owned)
- **Integration Tests**: API endpoints, database (QA owned)  
- **E2E Tests**: Full user workflows (QA owned)

## Preconditions
- Test environment with clean database
- Feature flags: AUTH_V2_ENABLED=true
- Test user accounts provisioned

## Test Scenarios
Given a valid user account exists
When the user enters correct credentials
Then they should be logged in successfully

Given an invalid user account
When the user enters incorrect credentials  
Then they should see "Invalid credentials" error
```

</details>

### **Test Case Components**

<details open>
<summary>Test case template</summary>

```
Test Case ID: TC_001
Title: Successful User Login
Priority: P1 (Critical)
Test Level: Integration
Prerequisites: User account exists in system
Test Data: username="testuser", password="validpass"

Steps:
1. Navigate to login page
2. Enter valid username
3. Enter valid password  
4. Click login button

Expected Result:
- User redirected to dashboard
- Welcome message displays user name
- Session cookie created

Cleanup:
- Logout user
- Clear session data
```

</details>


## **4 Testing Pillars Framework**

### **1. User Experience**

<details open>
<summary>User Experience testing</summary>

```python
# What to Test:
- Error message clarity and helpfulness
- UI responsiveness and feedback
- Accessibility compliance (WCAG)
- Cross-browser compatibility
- Mobile responsiveness

# Example Tests:
def test_error_message_clarity():
    login_page.enter_invalid_credentials()
    error = login_page.get_error_message()
    assert "Invalid username or password" in error
    assert error != "Error 401"  # Not user-friendly
```

</details>

### **2. Performance**

<details open>
<summary>Performance testing</summary>

```python
# What to Test:
- Response times under normal load
- Database query performance
- Memory usage and leaks
- Scalability with large datasets
- API rate limiting

# Example Tests:
def test_api_response_time():
    start_time = time.time()
    response = requests.get("/api/users")
    duration = time.time() - start_time
    assert duration < 2.0
    assert response.status_code == 200
```

</details>

### **3. Security**

<details open>
<summary>Security testing</summary>

```python
# What to Test:
- Input validation (SQL injection, XSS)
- Authentication and authorization
- Data encryption in transit/rest
- Session management
- CSRF protection

# Example Tests:
def test_sql_injection_protection():
    malicious_input = "'; DROP TABLE users; --"
    response = requests.post("/api/search", 
                           json={"query": malicious_input})
    assert response.status_code != 500  # Should handle gracefully
    assert "error" not in response.json().get("query_result", "")
```

</details>

### **4. Usability**

<details open>
<summary>Usability testing</summary>

```python
# What to Test:
- Intuitive navigation and workflows
- Consistent UI patterns
- Clear labeling and instructions
- Keyboard navigation support
- Help documentation accuracy

# Example Tests:
def test_keyboard_navigation():
    login_page.focus_username_field()
    login_page.press_tab()
    assert login_page.is_password_field_focused()
    login_page.press_tab()
    assert login_page.is_login_button_focused()
```

</details>

## **Test Data & Fixtures**

### **Dynamic Test Data**

<details open>
<summary>Dynamic test data examples</summary>

```python
from faker import Faker
import uuid

fake = Faker()
test_data = {
    "name": fake.name(),
    "email": fake.email(),
    "id": str(uuid.uuid4())
}
```

</details>

### **Fixture Pattern**

<details open>
<summary>Fixture pattern examples</summary>

```python
@pytest.fixture
def test_user():
    # Setup
    user = create_user_via_api()
    yield user
    # Teardown
    delete_user_via_api(user["id"])
```

</details>

## **Common Patterns**

### **Parameterized Tests**

<details open>
<summary>Parameterized test examples</summary>

```python
@pytest.mark.parametrize("input,expected", [
    ("valid_data", True),
    ("invalid_data", False)
])
def test_scenarios(input, expected):
    assert process(input) == expected
```

</details>

### **Error Handling**

<details open>
<summary>Error handling examples</summary>

```python
# API Errors
response = requests.get("/nonexistent")
assert response.status_code == 404

# UI Errors
login_page.login("invalid", "creds")
assert login_page.is_error_displayed()
```

</details>


## **Debugging Test Failures**

### **Test Failure Debugging Checklist**

<details>
<summary>Click to expand debugging checklist</summary>

#### **1. Environment & Setup Issues**
- [ ] **Test environment accessible?** Check URLs, services running
- [ ] **Authentication valid?** Tokens, credentials not expired
- [ ] **Database state clean?** Previous test data interfering
- [ ] **Feature flags correct?** Environment-specific configurations
- [ ] **Network connectivity?** API endpoints reachable
- [ ] **Browser/driver versions?** Compatibility issues

#### **2. Timing & Synchronization**
- [ ] **Implicit waits sufficient?** Elements loading slowly
- [ ] **Explicit waits used?** `WebDriverWait` for dynamic content
- [ ] **Race conditions?** Tests running too fast
- [ ] **Async operations?** API calls, AJAX requests completing
- [ ] **Page load timing?** Heavy pages, slow responses
- [ ] **Animation/transitions?** UI elements still moving

#### **3. Test Data Issues**
- [ ] **Data dependencies?** Required records exist
- [ ] **Unique constraints?** Duplicate data conflicts
- [ ] **Data cleanup?** Previous test pollution
- [ ] **Dynamic data generation?** Faker, UUID working correctly
- [ ] **Test isolation?** Tests affecting each other
- [ ] **Permissions/roles?** User has required access

#### **4. Element Location Problems**
- [ ] **Locators still valid?** UI changes breaking selectors
- [ ] **Element visibility?** Hidden, overlapped, or off-screen
- [ ] **Frame/window context?** Switched to correct frame
- [ ] **Dynamic IDs?** Generated IDs changing
- [ ] **CSS/XPath accuracy?** Selectors too brittle
- [ ] **Multiple matches?** Locator finding wrong element

#### **5. API-Specific Issues**
- [ ] **HTTP status codes?** Expected vs actual responses
- [ ] **Request headers?** Authentication, content-type
- [ ] **Request payload?** JSON structure, required fields
- [ ] **Response format?** JSON parsing, field names
- [ ] **Rate limiting?** Too many requests
- [ ] **Endpoint changes?** API version, URL modifications

#### **6. Test Logic & Code**
- [ ] **Assertions correct?** Expected vs actual values
- [ ] **Exception handling?** Proper try/catch blocks
- [ ] **Variable scope?** Accessing correct variables
- [ ] **Method calls?** Correct parameters passed
- [ ] **Loop logic?** Infinite loops, wrong conditions
- [ ] **Test sequence?** Dependent tests in correct order

#### **7. Infrastructure & CI/CD**
- [ ] **Resource availability?** Memory, CPU sufficient
- [ ] **Parallel execution?** Tests conflicting with each other
- [ ] **Container health?** Docker containers running
- [ ] **Service dependencies?** External services available
- [ ] **Build artifacts?** Correct version deployed
- [ ] **Environment variables?** Configuration values set

</details>

### **Debugging Workflow**

<details>
<summary>Click to expand systematic debugging approach</summary>

#### **Step 1: Reproduce Locally**
```bash
# Run the specific failing test
pytest tests/test_failing.py::test_method -v -s

# Run with maximum logging
pytest tests/test_failing.py::test_method -v -s --log-cli-level=DEBUG

# Run in headed mode (for UI tests)
pytest tests/test_failing.py::test_method --headed
```

#### **Step 2: Gather Information**
```python
# Add debugging prints
print(f"Current URL: {driver.current_url}")
print(f"Page title: {driver.title}")
print(f"Response status: {response.status_code}")
print(f"Response body: {response.text}")

# Take screenshots (UI tests)
driver.save_screenshot("debug_failure.png")

# Capture page source
with open("debug_page_source.html", "w") as f:
    f.write(driver.page_source)
```

#### **Step 3: Isolate the Problem**
```python
# Test individual components
def test_debug_login_only():
    login_page.login("user", "pass")
    assert login_page.is_logged_in()  # Does login work?

def test_debug_api_only():
    response = api_client.get_user(user_id)
    assert response.status_code == 200  # Does API work?
```

#### **Step 4: Add Resilience**
```python
# Implement retry logic
@pytest.mark.retry(3)
def test_with_retry():
    # Test implementation
    pass

# Add better waits
def wait_for_element_clickable(locator, timeout=10):
    return WebDriverWait(driver, timeout).until(
        EC.element_to_be_clickable(locator)
    )

# Implement fallback strategies
def robust_click(locator):
    try:
        element = wait_for_element_clickable(locator)
        element.click()
    except TimeoutException:
        # Fallback: JavaScript click
        element = driver.find_element(*locator)
        driver.execute_script("arguments[0].click();", element)
```

</details>

### **Common Failure Patterns & Solutions**

<details>
<summary>Click to expand common failure patterns</summary>

#### **Flaky Tests**
```python
# Problem: Test passes sometimes, fails others
# Solutions:
- Add explicit waits instead of sleep()
- Use dynamic test data (avoid hardcoded values)
- Implement proper cleanup in fixtures
- Check for race conditions in parallel execution
```

#### **Environment-Specific Failures**
```python
# Problem: Works locally, fails in CI
# Solutions:
- Check environment variables and configurations
- Verify service dependencies are available
- Add environment-specific waits (CI often slower)
- Use containerization for consistency
```

#### **Data-Dependent Failures**
```python
# Problem: Test depends on specific database state
# Solutions:
@pytest.fixture
def clean_database():
    # Setup known state
    db.clear_all_users()
    yield
    # Cleanup
    db.clear_all_users()
```

#### **Stale Element Exceptions**
```python
# Problem: Element reference becomes invalid
# Solution: Re-locate elements
def robust_send_keys(locator, text):
    element = driver.find_element(*locator)  # Fresh reference
    element.clear()
    element.send_keys(text)
```

</details>

