# Текущая погода

Получние текущего прогноза погоды с __weatherstack.com__. 

__The Weather Company__ — это компания, специализирующееся на прогнозировании погоды и разработке инновационных информационных технологий.

## Реализация

```python title="python"
import requests


# (1)!
OPENWEATHERMAP_API_KEY = ""
WEATHERSTACK_API_KEY = ""

# (2)!
OPENWEATHERMAP_URL_BASE = "https://api.openweathermap.org/data/2.5/weather"
WEATHERSTACK_URL_BASE = "http://api.weatherstack.com/current"


def current_weather(location: str) -> list[dict]:
    weather_data = []
    if OPENWEATHERMAP_API_KEY:
        params_openweathermap = {"q": location, "appid": OPENWEATHERMAP_API_KEY}
        response_openweathermap = requests.get(
            OPENWEATHERMAP_URL_BASE, params=params_openweathermap, timeout=10
        )
        weather_data.append({"OpenWeatherMap": response_openweathermap.json()})
    if WEATHERSTACK_API_KEY:
        params_weatherstack = {"query": location, "access_key": WEATHERSTACK_API_KEY}
        response_weatherstack = requests.get(
            WEATHERSTACK_URL_BASE, params=params_weatherstack, timeout=10
        )
        weather_data.append({"Weatherstack": response_weatherstack.json()})
    if not weather_data:
        raise ValueError("No API keys provided or no valid data returned.")
    return weather_data


if __name__ == "__main__":
    from pprint import pprint

    location = "to be determined..."
    while location:
        location = input("Enter a location (city name or latitude,longitude): ").strip()
        if location:
            try:
                weather_data = current_weather(location)
                for forecast in weather_data:
                    pprint(forecast)
            except ValueError as e:
                print(repr(e))
                location = ""
```

1.  Необходимо добавить свой API-ключ

2.  Определите URL-адрес для API-интерфейсов