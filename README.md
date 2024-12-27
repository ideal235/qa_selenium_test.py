# selenium_script.py-
Python Selenium Automation 
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pytest

class RowsCountCondition:
    def __init__(self, expected_count):
        self.expected_count = expected_count

    def __call__(self, driver):
        rows = [row for row in driver.find_elements(By.CSS_SELECTOR, "#example tbody tr") if row.is_displayed()]
        return len(rows) == self.expected_count

@pytest.fixture
def setup():
    """Sets up the Selenium WebDriver with Chrome."""
    print("Initializing the browser...")
    options = webdriver.ChromeOptions()
    options.add_argument("--start-maximized")
    driver = webdriver.Chrome(options=options)
    yield driver
    print("Closing the browser...")
    driver.quit()

@pytest.mark.parametrize("search_term, expected_visible_rows", [("New York", 5)])
def test_search_functionality(setup, search_term, expected_visible_rows):
    """Tests the search functionality of the Selenium Playground Table Search Demo."""
    driver = setup

    # Navigate to the website
    print(f"Navigating to the website...")
    driver.get("https://www.lambdatest.com/selenium-playground/table-sort-search-demo")

    # Wait for the search box to be visible
    print(f"Waiting for the search box to be visible...")
    wait = WebDriverWait(driver, 10)
    search_box = wait.until(EC.visibility_of_element_located((By.CSS_SELECTOR, "input[aria-controls='example']")))
    print(f"Search box located. Entering search term: '{search_term}'")
    search_box.clear()
    search_box.send_keys(search_term)

    # Use custom condition to wait for rows to update
    print(f"Waiting for rows to update based on the search term '{search_term}'...")
    wait.until(RowsCountCondition(expected_visible_rows))

    # Validate visible rows
    visible_rows = [row for row in driver.find_elements(By.CSS_SELECTOR, "#example tbody tr") if row.is_displayed()]
    print(f"Visible rows count: {len(visible_rows)}. Expected: {expected_visible_rows}.")
    assert len(visible_rows) == expected_visible_rows, f"Expected {expected_visible_rows} visible rows, but found {len(visible_rows)}. Rows: {[row.text for row in visible_rows]}"

    # Wait for total entries text to be updated
    print(f"Validating total entries text...")
    total_entries_text = wait.until(EC.presence_of_element_located((By.ID, "example_info"))).text
    print(f"Total entries text found: '{total_entries_text}'")
    assert "filtered from" in total_entries_text, f"Expected 'filtered from' in total entries text, but found: {total_entries_text}"

    print(f"Test completed successfully for search term: '{search_term}'!")
