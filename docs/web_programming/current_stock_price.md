# Текущая биржевая цена

Поиск и получение текущей биржевой цены с __Yahoo Finance__.

__Yahoo Finance__ — это медиа-ресурс, входящий в состав __Yahoo network__. Он предлагает разнообразные финансовые новости, данные и комментарии, включая биржевые котировки, пресс-релизы, финансовые отчеты и оригинальный контент. Кроме того, на сайте доступны некоторые онлайн-инструменты для управления личными финансами.

## Реализация

```python title="python"
import requests
from bs4 import BeautifulSoup


# (1)!
def stock_price(symbol: str = "AAPL") -> str:
    url = f"https://finance.yahoo.com/quote/{symbol}?p={symbol}"
    yahoo_finance_source = requests.get(
        url, headers={"USER-AGENT": "Mozilla/5.0"}, timeout=10
    ).text
    soup = BeautifulSoup(yahoo_finance_source, "html.parser")

    if specific_fin_streamer_tag := soup.find("span", {"data-testid": "qsp-price"}):
        return specific_fin_streamer_tag.get_text()
    return "No <fin-streamer> tag with the specified data-testid attribute found."
```

1.  Получите HTML-код с __Yahoo Finance__ и выберите текущую цену  
    __AAPL__ stock price is   `228.43`  
    __AMZN__ stock price is   `201.85`  
    __IBM__  stock price is   `210.30`  
    __GOOG__ stock price is   `177.86`  
    __MSFT__ stock price is   `414.82`  
    __ORCL__ stock price is   `188.87`
