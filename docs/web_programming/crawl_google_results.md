# Парсинг результатов из поиска Google

Парсер поисковой выдачи __Google__ является одним из самых популярных инструментов. С его помощью вы сможете собирать огромные базы ссылок, которые будут готовы к дальнейшему использованию. Вы сможете использовать запросы в том виде, в котором они вводятся в __Google__, включая различные поисковые операторы, такие как `inurl`, `intitle` и другие.

## Реализация

```python title="python"
import sys
import webbrowser

import requests
from bs4 import BeautifulSoup
from fake_useragent import UserAgent

if __name__ == "__main__":
    print("Googling.....")
    url = "https://www.google.com/search?q=" + " ".join(sys.argv[1:])
    res = requests.get(url, headers={"UserAgent": UserAgent().random}, timeout=10)
    with open("project1a.html", "wb") as out_file:
        for data in res.iter_content(10000):
            out_file.write(data)
    soup = BeautifulSoup(res.text, "html.parser")
    links = list(soup.select(".eZt8xd"))[:5]

    print(len(links))
    for link in links:
        if link.text == "Maps":
            webbrowser.open(link.get("href"))
        else:
            webbrowser.open(f"https://google.com{link.get('href')}")
```
