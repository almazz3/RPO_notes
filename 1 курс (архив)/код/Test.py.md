#Python 

```python
'''
1. Зайти на сайт qa-practice.com
2. Найти пункт 2 "simple button"
3. Кликнуть на элемент и перейти на 
след. страницу
4. Проверить кнопку "click"
'''
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.firefox.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

@pytest.fixture(scope="session")
def browser():
    options = Options()
    options.page_load_strategy = 'eager'
    driver = webdriver.Firefox(options=options)
    yield driver
    driver.quit()

def test_openMainPage(browser):
    browser.get('https://practice-automation.com/')
    WebDriverWait(browser, 10).until(EC.presence_of_element_located((By.TAG_NAME, "body")))
    assert "Practice Automation" in browser.title

def test_findAndClickEvents(browser):
    wait = WebDriverWait(browser, 15)
    button = wait.until(EC.element_to_be_clickable((By.LINK_TEXT, 'Click Events')))
    button.click()
    
    wait.until(EC.url_contains('click-events'))
    assert 'click-events' in browser.current_url
```
