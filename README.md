# kinopoisk_top500.py
kino_top500
# Script
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import time
import pandas as pd

chromedriver_path = "C:/Users/Luda/Downloads/chromedriver-win64/chromedriver-win64/chromedriver.exe"


options = Options()
options.add_argument("--start-maximized")
service = Service(chromedriver_path)
driver = webdriver.Chrome(service=service, options=options)

url = "https://www.kinopoisk.ru/lists/movies/top500/"
driver.get(url)

try:
    WebDriverWait(driver, 20).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, "[data-test-id='movie-list-item']"))
    )
    print("The page has loaded successfully.")
except Exception as e:
    print(f"Error loading page: {e}")
    driver.quit()

all_movies = []

while True:
    soup = BeautifulSoup(driver.page_source, "html.parser")
    movies = soup.find_all("div", attrs={"data-test-id": "movie-list-item"})
    for movie in movies:
        title = movie.find("span", class_="styles_mainTitle__IFQyZ")
        secondary = movie.find("div", class_="desktop-list-main-info_secondaryTitleSlot__mc0mI")
        rating = movie.find("span", class_="styles_kinopoiskValue__nkZEC")
        link = movie.find("a", class_="base-movie-main-info_link__YwtP1")

        all_movies.append({
            "title": title.text.strip() if title else None,
            "info": secondary.text.strip() if secondary else None,
            "rating": rating.text.strip() if rating else None,
            "url": "https://www.kinopoisk.ru" + link['href'] if link else None
        })
    for film in all_movies[-len(movies):]:
        print(f"🎬 {film['title']}")
        print(f"📅 {film['info']}")
        print(f"⭐ {film['rating']}")
        print(f"🔗 {film['url']}")
        print("-" * 50)

    try:
        next_button = driver.find_element(By.XPATH, "//a[contains(@class, 'styles_end__aEsmB')]")
        if next_button:
            next_button.click()
            WebDriverWait(driver, 20).until(
                EC.presence_of_element_located((By.CSS_SELECTOR, "[data-test-id='movie-list-item']"))
            )  # Ждем загрузки страницы
            time.sleep(2)
        else:
            print("Could not find next page button")
            break
    except Exception as e:
        print("Error when moving to the next page:", e)
        break
driver.quit()
df = pd.DataFrame(all_movies)
df.to_csv("kinopoisk_top500.csv", index=False, encoding="utf-8-sig")

print("✅kinopoisk_top500.csv")
