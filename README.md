from selenium import webdriver
from selenium.webdriver.common.by import By
from datetime import datetime
import os
import csv

# Create screenshots folder if not exists
os.makedirs("screenshots", exist_ok=True)

# CSV file to log bugs
csv_file = "bug_report.csv"

# Create CSV file with headers if not exists
if not os.path.exists(csv_file):
    with open(csv_file, "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["Scenario", "Input Used", "Issue Found", "Screenshot Path", "Timestamp"])

# Test cases: (username, password, description)
test_cases = [
    ("wronguser", "Password123", "Wrong Username"),
    ("student", "wrongpass", "Wrong Password"),
    ("wronguser", "wrongpass", "Both Incorrect")
]

for username, password, label in test_cases:
    driver = webdriver.Chrome()
    driver.get("https://practicetestautomation.com/practice-test-login/")

    # Fill login form
    driver.find_element(By.ID, "username").send_keys(username)
    driver.find_element(By.ID, "password").send_keys(password)
    driver.find_element(By.ID, "submit").click()

    # Try to detect error message
    try:
        error_text = driver.find_element(By.ID, "error").text
        if "invalid" in error_text.lower():
            print(f"{label} Test Passed: Error message shown.")
        else:
            raise Exception("Unexpected error message")
    except:
        # Bug detected: Take screenshot and log to CSV
        timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
        screenshot_path = f"screenshots/{label.replace(' ', '_')}_{timestamp}.png"
        driver.save_screenshot(screenshot_path)

        with open(csv_file, "a", newline="") as file:
            writer = csv.writer(file)
            writer.writerow(["Negative Login", label, "No or incorrect error message", screenshot_path, timestamp])

        print(f"{label} Test Failed: Error message not shown or incorrect.")

    driver.quit()
