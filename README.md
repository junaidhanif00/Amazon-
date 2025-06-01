# Amazon-

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import pandas as pd
import os
import time

# Create the 'data' directory if it doesn't exist
os.makedirs("data", exist_ok=True)

driver = webdriver.Chrome()
Query = "Food"
file = 0

for i in range(1, 5):
    driver.get(f"https://www.amazon.in/s?k={Query}&page={i}&crid=3EQKLSU9TWCSK&qid=1724821595&sprefix=Food%2Caps%2C496&ref=sr_pg_2")
    elems = driver.find_elements(By.CLASS_NAME, "puis-card-container")
    print(f"{len(elems)} Items Found")

    for elem in elems:
        d = elem.get_attribute("outerHTML")
        with open(f"data/{Query}_{file}.html", "w", encoding="utf-8") as f:
            f.write(d)
        file += 1  # Increment the file name for each element

    time.sleep(2)

driver.close()
d = {'title': [], 'price': [], 'link': []}

for file in os.listdir("data"):
    try:
        with open(f"data/{file}", 'r', encoding='utf-8') as f:
            html_doc = f.read()
        soup = BeautifulSoup(html_doc, 'html.parser')
        
        t = soup.find("h2")
        title = t.get_text() if t else "No title found"

        l = soup.find('a')
        link = 'https://amazon.in/' + l['href'] if l else "No link found"

        p = soup.find("span", attrs={"class": 'a-price-whole'})
        price = p.get_text() if p else "No price found"

        d['title'].append(title)
        d['link'].append(link)
        d['price'].append(price)

    except Exception as e:
        print(f"An error occurred while processing the file {file}: {e}")

df = pd.DataFrame(data=d)
df.to_csv("data.csv", index=False)
