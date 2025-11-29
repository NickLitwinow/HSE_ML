n# 🔧 Решение проблем с индексаторами

## 🚨 RuTracker: Капча при входе

### Проблема
```
Indexer returned result for RSS URL, Credentials appears to be invalid. 
Response: Пожалуйста, введите код подтверждения (символы, изображенные на картинке)
```

### Решение 1: Использовать Cookie-based аутентификацию

#### Шаг 1: Вручную залогиниться на RuTracker
1. Открой браузер в режиме инкогнито
2. Перейди на https://rutracker.org
3. Залогинься (введи капчу вручную)
4. После успешного входа, открой Developer Tools (F12)
5. Вкладка **Application** → **Cookies** → `rutracker.org`
6. Найди cookie `bb_session` (самый важный)
7. Скопируй значение (длинная строка типа `0-12345678-AbCdEfGh...`)

#### Шаг 2: Настроить Prowlarr с Cookie

**В Prowlarr:**
1. Indexers → Edit RuTracker
2. Удали Username/Password
3. Добавь поле **Cookie** (если есть)
4. Вставь: `bb_session=ТВОЁ_ЗНАЧЕНИЕ_ИЗ_БРАУЗЕРА`
5. Test → Save

**Важно:** Cookie истекает через ~1-2 недели, нужно будет обновлять.

---

### Решение 2: Использовать альтернативные русские трекеры

Вместо RuTracker попробуй:

#### 1. **NNMClub** (проще в настройке)
- URL: `https://nnmclub.to`
- Регистрация: Открытая
- В Prowlarr: Search "NNMClub" → Add
- Username/Password (твои от NNMClub)
- Categories: Music

#### 2. **Rustorka** (большой архив музыки)
- URL: `https://rustorka.com`
- Регистрация: Иногда закрыта (нужен инвайт)
- В Prowlarr: Search "Rustorka" → Add
- Cookie-based auth (как RuTracker)

#### 3. **Tapochek** (хороший музыкальный раздел)
- URL: `https://tapochek.net`
- Регистрация: Иногда закрыта
- В Prowlarr: Search "Tapochek" → Add

---

### Решение 3: Использовать публичные трекеры (легче)

Эти не требуют регистрации и работают сразу:

1. **The Pirate Bay**
   - Prowlarr: Search "The Pirate Bay" → Add
   - Без аутентификации
   
2. **RARBG** (если доступен)
   - Prowlarr: Search "RARBG" → Add
   - Без аутентификации

3. **Torrentz2** (мета-поиск)
   - Prowlarr: Search "Torrentz2" → Add
   - Без аутентификации

---

## 🚨 1337x: CloudFlare Protection

### Проблема
```
Unable to access 1337x.to, blocked by CloudFlare Protection
```

### Решение: Установить FlareSolverr

FlareSolverr — это прокси-сервис, который обходит CloudFlare защиту.

#### Шаг 1: Добавить FlareSolverr в docker-compose.yml

Открой `/root/Homeserver/docker-compose.yml` и добавь в секцию "ARR STACK":

```yaml
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    restart: unless-stopped
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
      - CAPTCHA_SOLVER=none
      - TZ=${TZ}
    ports:
      - "8191:8191"
    networks:
      - internal-net
    mem_limit: 512m
    logging: *default-logging
```

#### Шаг 2: Запустить FlareSolverr

```bash
ssh root@167.253.157.7
cd /root/Homeserver
docker-compose up -d flaresolverr
docker logs flaresolverr -f
```

**Ожидаемый вывод:**
```
FlareSolverr started successfully
```

#### Шаг 3: Настроить Prowlarr

1. Открой Prowlarr: `http://167.253.157.7:9696`
2. **Settings → Indexers**
3. **FlareSolverr** section:
   - Tags: `flaresolverr` (создай новый тег)
   - Host: `http://flaresolverr:8191/`
   - Save

#### Шаг 4: Добавить 1337x с FlareSolverr

1. **Indexers → Add Indexer**
2. Search: `1337x`
3. Select: **1337x**
4. **Tags:** `flaresolverr` (выбери созданный тег)
5. **Categories:** Music
6. Test → Save

**Результат:** 1337x будет работать через FlareSolverr.

---

## 🎯 Рекомендуемая конфигурация индексаторов

### Вариант 1: Без сложностей (рекомендуется для старта)

1. ✅ **The Pirate Bay** (публичный, без регистрации)
2. ✅ **1337x** (через FlareSolverr)
3. ✅ **RARBG** (если доступен)
4. ✅ **YTS** (если нужны фильмы)

**Плюсы:** Работает сразу, не нужны аккаунты  
**Минусы:** Может быть меньше русского контента

---

### Вариант 2: С русскими трекерами (лучшее качество)

1. ✅ **NNMClub** (простая регистрация)
2. ✅ **The Pirate Bay** (резервный)
3. ✅ **1337x** (через FlareSolverr, международный)
4. 🔶 **RuTracker** (через Cookie, если сможешь настроить)

**Плюсы:** Больше русской музыки, лучшее качество  
**Минусы:** Нужна регистрация, Cookie для RuTracker

---

### Вариант 3: Максимальное покрытие

1. ✅ **NNMClub** (русская музыка)
2. ✅ **1337x** (международный, через FlareSolverr)
3. ✅ **The Pirate Bay** (резервный)
4. ✅ **Slskd** (Soulseek, для редкой музыки) — см. `MUSIC_SLSKD_ADDON.md`

**Плюсы:** Максимальное покрытие  
**Минусы:** Больше настроек

---

## 🔧 Пошаговая настройка (обновленная)

### Шаг 1: Установить FlareSolverr
```bash
ssh root@167.253.157.7
cd /root/Homeserver

# Добавь FlareSolverr в docker-compose.yml (см. выше)
nano docker-compose.yml

# Запусти
docker-compose up -d flaresolverr
docker logs flaresolverr
```

### Шаг 2: Настроить Prowlarr
1. Открой Prowlarr: `http://167.253.157.7:9696`
2. Settings → Indexers → FlareSolverr:
   - Tags: `flaresolverr`
   - Host: `http://flaresolverr:8191/`
   - Save

### Шаг 3: Добавить индексаторы

#### A) The Pirate Bay (самый простой)
1. Indexers → Add Indexer
2. Search: "The Pirate Bay"
3. Categories: Music
4. Test → Save

#### B) 1337x (с FlareSolverr)
1. Indexers → Add Indexer
2. Search: "1337x"
3. Tags: `flaresolverr`
4. Categories: Music
5. Test → Save

#### C) NNMClub (если зарегистрировался)
1. Регистрация: https://nnmclub.to/forum/index.php
2. Indexers → Add Indexer
3. Search: "NNMClub"
4. Username: твой логин
5. Password: твой пароль
6. Categories: Music
7. Test → Save

### Шаг 4: Синхронизировать с Lidarr
1. Settings → Apps → Lidarr → **Sync**
2. Проверь: Lidarr → Settings → Indexers (должны появиться индексаторы)

---

## 🧪 Тест

### Проверить, что индексаторы работают:

```bash
# Через API Prowlarr
ssh root@167.253.157.7

curl -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  "http://localhost:9696/api/v1/search?query=Radiohead&type=music" | jq
```

**Ожидаемый результат:** Список торрентов с разных индексаторов

---

## 📊 Сравнение индексаторов

| Индексатор | Регистрация | Качество | Русский контент | Сложность |
|------------|-------------|----------|-----------------|-----------|
| The Pirate Bay | ❌ Нет | ⭐⭐⭐ | ⭐⭐ | 🟢 Легко |
| 1337x | ❌ Нет | ⭐⭐⭐⭐ | ⭐⭐⭐ | 🟡 Средне (FlareSolverr) |
| NNMClub | ✅ Да | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 🟢 Легко |
| RuTracker | ✅ Да | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 🔴 Сложно (Cookie) |
| RARBG | ❌ Нет | ⭐⭐⭐⭐ | ⭐⭐ | 🟢 Легко |
| Slskd (Soulseek) | ❌ Нет | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 🟡 Средне (отдельный сервис) |

---

## 🎓 Альтернатива: Usenet (для продвинутых)

Если торренты не работают хорошо, можно использовать **Usenet**:

1. Купить доступ к Usenet провайдеру ($5-10/месяц)
2. Установить **NZBGet** или **SABnzbd**
3. Добавить **NZB индексаторы** в Prowlarr
4. Настроить Lidarr для Usenet

**Плюсы:** Быстрее, безопаснее, лучшее качество  
**Минусы:** Платный доступ

---

## 💡 Рекомендация

**Для начала:**
1. ✅ Установи FlareSolverr (5 минут)
2. ✅ Добавь The Pirate Bay (1 минута)
3. ✅ Добавь 1337x через FlareSolverr (2 минуты)
4. ✅ Синхронизируй с Lidarr
5. ✅ Тест: добавь первого артиста

**Результат:** Рабочая система без сложностей с русскими трекерами.

**Потом (опционально):**
- Зарегистрируйся на NNMClub для русской музыки
- Настрой RuTracker через Cookie (если нужно)
- Добавь Slskd для редкой музыки

---

## 🆘 Если ничего не помогает

### Вариант 1: Только публичные трекеры
- The Pirate Bay + 1337x (с FlareSolverr)
- Будет работать для 80% музыки

### Вариант 2: Только Soulseek
- Установи Slskd (см. `MUSIC_SLSKD_ADDON.md`)
- Soulseek специализируется на музыке
- Не требует трекеров

### Вариант 3: Hybrid (лучший вариант)
- The Pirate Bay + 1337x (торренты)
- Slskd (для редкой музыки)
- Покрытие ~95% музыки

---

**Создано:** 30 ноября 2025  
**Цель:** Решение проблем с индексаторами RuTracker и 1337x

