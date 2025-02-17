# Получение информации с Github

Для аутентификации используется токен доступа. Чтобы создать свой персональный токен, перейдите настройки своей учетной записи.

__ВНИМАНИЕ__:  
Никогда не записывайте учетные данные непосредственно в код. Вместо этого используйте файл среды для безопасного хранения конфиденциальной информации. Во время выполнения приложения вы можете получить доступ к этой информации с помощью модуля `os`.

Чтобы создать файл среды, откройте корневой каталог и создайте новый файл с именем `.env`. Затем добавьте в этот файл две строки с вашим токеном:

```bash
#!/usr/bin/env bash
export USER_TOKEN=""
```

## Реализация

```python title="python"
from __future__ import annotations

import os
from typing import Any

import requests

BASE_URL = "https://api.github.com"

# (1)!
AUTHENTICATED_USER_ENDPOINT = BASE_URL + "/user"

# (2)!
USER_TOKEN = os.environ.get("USER_TOKEN", "")


def fetch_github_info(auth_token: str) -> dict[Any, Any]:
    headers = {
        "Authorization": f"token {auth_token}",
        "Accept": "application/vnd.github.v3+json",
    }
    return requests.get(AUTHENTICATED_USER_ENDPOINT, headers=headers, timeout=10).json()


if __name__ == "__main__":
    if USER_TOKEN:
        for key, value in fetch_github_info(USER_TOKEN).items():
            print(f"{key}: {value}")
    else:
        raise ValueError("'USER_TOKEN' field cannot be empty.")
```

1.  https://docs.github.com/en/rest/users/users?apiVersion=2022-11-28#get-the-authenticated-user

2.  Получите токен в настройках своей учетной записи