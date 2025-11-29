# 🚀 Быстрая настройка музыкального стека

**Время на настройку:** ~20 минут  
**Дата:** 29 ноября 2025

---

## 📋 Что нужно

- [x] SSH доступ к серверу: `ssh root@167.253.157.7`
- [x] Все контейнеры запущены (Navidrome, Lidarr, Prowlarr, qBittorrent)
- [ ] Spotify Developer Account (для интеграции)
- [ ] Регистрация на RuTracker.org (для скачивания)

---

## 🔑 Ключи и доступы

### API Keys (уже созданы):
```
Lidarr API Key:   308430cf14d547c9aada25d71ad925e4
Prowlarr API Key: ff149a593ccc4580b45b95a5fd1c8a36
```

### Домены:
```
Navidrome:   https://music.litwein.bond
Lidarr:      https://lidarr.litwein.bond (нужно настроить NPM)
Prowlarr:    https://prowlarr.litwein.bond (нужно настроить NPM)
qBittorrent: https://qbit.litwein.bond (нужно настроить NPM)
```

### Порты (внутренние):
```
Navidrome:   4533
Lidarr:      8686
Prowlarr:    9696
qBittorrent: 8080
```

---

## 🎯 Настройка за 5 шагов

### ШАГ 1: Настроить Nginx Proxy Manager (5 мин)

#### 1.1 Открой NPM
```
https://litwein.bond:81
```

#### 1.2 Добавь Proxy Hosts для внутренних сервисов

**Для Lidarr:**
- Domain Names: `lidarr.litwein.bond`
- Forward Hostname / IP: `lidarr`
- Forward Port: `8686`
- ✅ Block Common Exploits
- ✅ Websockets Support
- SSL: Request a new SSL Certificate (Let's Encrypt)

**Повтори для:**
- `prowlarr.litwein.bond` → `prowlarr:9696`
- `qbit.litwein.bond` → `qbittorrent:8080`

#### 1.3 Добавь защиту (ВАЖНО!)
- Access Lists → Add Access List → "Internal Services"
- Authorization: Basic Auth
- Username: `admin`
- Password: `Fydx8353_` (или свой)
- Примени к Lidarr, Prowlarr, qBittorrent

---

### ШАГ 2: Настроить Prowlarr (5 мин)

#### 2.1 Открой Prowlarr
```
https://prowlarr.litwein.bond
Username: admin
Password: Fydx8353_
```

#### 2.2 Добавь индексаторы (трекеры)

⚠️ **ВАЖНО:** RuTracker и 1337x могут требовать дополнительную настройку:
- **RuTracker:** Может просить капчу → используй альтернативы (NNMClub, The Pirate Bay)
- **1337x:** Требует FlareSolverr для обхода CloudFlare

**Рекомендуемая конфигурация (без сложностей):**

**Indexers → Add Indexer:**

1. **The Pirate Bay** (самый простой):
   - Search: `The Pirate Bay`
   - URL: Auto
   - Categories: Music
   - **Без регистрации!**

2. **1337x** (требует FlareSolverr, см. ниже):
   - ⚠️ Сначала установи FlareSolverr (см. раздел "Установка FlareSolverr")
   - Search: `1337x`
   - Tags: `flaresolverr`
   - Categories: Music

3. **NNMClub** (альтернатива RuTracker):
   - Регистрация: https://nnmclub.to
   - Search: `NNMClub`
   - Username: `твой_логин_nnmclub`
   - Password: `твой_пароль_nnmclub`
   - Categories: Music

#### 2.2.1 Установка FlareSolverr (для 1337x)

FlareSolverr нужен для обхода CloudFlare защиты на 1337x.

**Шаг 1:** FlareSolverr уже добавлен в `docker-compose.yml` ✅

**Шаг 2:** Запусти контейнер:
```bash
ssh root@167.253.157.7
cd /root/Homeserver
docker-compose up -d flaresolverr
docker logs flaresolverr
```

**Шаг 3:** Настрой в Prowlarr:
- Settings → Indexers → FlareSolverr:
  - Tags: `flaresolverr` (создай новый тег)
  - Host: `http://flaresolverr:8191/`
  - Save

**Шаг 4:** Теперь можешь добавить 1337x с тегом `flaresolverr`

---

**Альтернативная конфигурация (если не хочешь возиться):**

Просто добавь **The Pirate Bay** — этого достаточно для начала!
- Работает без регистрации
- Без капчи
- Без FlareSolverr
- Есть музыка на русском и английском

**Подробнее о проблемах с индексаторами:** См. `MUSIC_INDEXERS_FIX.md`

#### 2.3 Подключи Lidarr к Prowlarr

**Settings → Apps → Add Application:**
- Application: `Lidarr`
- Sync Level: `Full Sync`
- Prowlarr Server: `http://prowlarr:9696`
- Lidarr Server: `http://lidarr:8686`
- API Key: `308430cf14d547c9aada25d71ad925e4`
- ✅ Sync Categories: `Music`
- Test → Save

**Результат:** Prowlarr автоматически добавит все трекеры в Lidarr.

---

### ШАГ 3: Настроить qBittorrent (2 мин)

#### 3.1 Открой qBittorrent
```
https://qbit.litwein.bond
Default Username: admin
Default Password: adminadmin (измени после первого входа!)
```

#### 3.2 Настрой категории
**Правая кнопка мыши → Add category:**
- Name: `lidarr`
- Save path: `/downloads/lidarr`

#### 3.3 Измени пароль
**Tools → Options → Web UI:**
- Username: `admin`
- New Password: `Fydx8353_` (или свой, запомни!)
- Save

---

### ШАГ 4: Настроить Lidarr (5 мин)

#### 4.1 Открой Lidarr
```
https://lidarr.litwein.bond
Username: admin
Password: Fydx8353_
```

#### 4.2 Подключи qBittorrent

**Settings → Download Clients → Add → qBittorrent:**
- Name: `qBittorrent`
- Host: `qbittorrent`
- Port: `8080`
- Username: `admin`
- Password: `Fydx8353_` (или тот, что ты поставил)
- Category: `lidarr`
- Test → Save

#### 4.3 Проверь Root Folder

**Settings → Media Management → Root Folders:**
- Должен быть: `/music`
- Если нет → Add: `/music`

#### 4.4 Настрой качество

**Settings → Profiles → Quality Profiles:**
- Edit: `Any` (или создай новый)
- Порядок предпочтений (сверху вниз):
  1. FLAC
  2. MP3-320
  3. MP3-V0
  4. MP3-256

#### 4.5 (Опционально) Включи автоматическое переименование

**Settings → Media Management:**
- ✅ Rename Tracks
- ✅ Replace Illegal Characters
- Standard Track Format:
```
{Album Title} ({Release Year})/{medium:00}{track:00} - {Track Title}
```
- Artist Folder Format:
```
{Artist Name}
```

---

### ШАГ 5: Spotify Integration (3 мин)

#### 5.1 Создай Spotify App
1. Открой: https://developer.spotify.com/dashboard
2. Log in (с твоим Spotify аккаунтом)
3. Create App:
   - App name: `Lidarr Import`
   - App description: `Self-hosted music automation`
   - Redirect URI: `https://lidarr.litwein.bond/oauth/callback`
   - API: ✅ Web API
4. Copy:
   - Client ID
   - Client Secret (Show Client Secret)

#### 5.2 Добавь в Lidarr

**Settings → Import Lists → Add → Spotify:**
- Name: `Spotify - My Saved Albums`
- Monitor: `All Albums`
- Root Folder: `/music`
- Quality Profile: `Any` (или твой)
- Spotify User ID: `твой_spotify_username`
- Auth with Spotify → Authorize
- ✅ Import Saved Albums
- ✅ Monitor New Items
- Save

**Опционально:** Добавь плейлист для мониторинга:
- Add → Spotify Playlists
- Name: `Spotify - Download Playlist`
- Playlist ID: (из URL плейлиста)

---

## ✅ Тест: Добавь первого артиста

### Ручное добавление

1. Открой Lidarr: `https://lidarr.litwein.bond`
2. Library → Add New
3. Поиск: `Radiohead` (или любой артист)
4. Root Folder: `/music`
5. Quality Profile: `Any`
6. Monitor: `All Albums` (или `Studio Albums`)
7. ✅ Search on add
8. Add Artist

### Что должно произойти:

1. **Prowlarr:** Ищет на трекерах
2. **Lidarr:** Находит торренты
3. **qBittorrent:** Начинает загрузку
4. **Lidarr:** После загрузки переносит файлы в `/music/Radiohead/`
5. **Navidrome:** Автоматически сканирует новую музыку (через 1 час, или ручной скан)

---

## 📱 Настройка Symfonium

### Установка
- Google Play: https://play.google.com/store/apps/details?id=app.symfonium.music
- Цена: $4.99

### Первое подключение

1. Открой Symfonium
2. Add Server → Subsonic/Navidrome
3. Server URL: `https://music.litwein.bond`
4. Username: `твой_navidrome_username`
5. Password: `твой_navidrome_password`
6. Test → Save
7. Sync Library

### Рекомендуемые настройки

**Settings → Playback:**
- ✅ Gapless Playback
- ✅ Normalize Volume
- Audio Focus: Always duck

**Settings → Cache:**
- ✅ Smart Cache
- Max Cache Size: `10 GB` (или сколько можешь)
- Cache Quality: `Original` (FLAC)

**Settings → Downloads:**
- Download Location: Internal Storage
- Download Quality: Original

**Settings → Interface:**
- Theme: Dark (или на вкус)
- ✅ Show Now Playing Bar

---

## 🧪 Проверка работы стека

### Команды для диагностики

```bash
# SSH в сервер
ssh root@167.253.157.7

# Статус контейнеров
docker ps | grep -E '(navidrome|lidarr|prowlarr|qbit)'

# Логи Lidarr (в реальном времени)
docker logs lidarr -f

# Проверить, что музыка появилась
ls -lah /mnt/sata/data/media/music/

# Ручной скан Navidrome
docker exec navidrome curl -X POST http://localhost:4533/api/scan

# Размер библиотеки
du -sh /mnt/sata/data/media/music/
```

### Проверка через WebUI

1. **Prowlarr → History:**
   - Должны быть запросы от Lidarr

2. **Lidarr → Activity → Queue:**
   - Должны быть загрузки (если добавил артиста)

3. **qBittorrent → Transfers:**
   - Активные торренты с категорией `lidarr`

4. **Navidrome → Settings → Scan Library Now:**
   - Должны появиться треки после загрузки

---

## 🐛 Troubleshooting

### Lidarr не находит музыку
**Решение:**
```bash
# Проверь, что Prowlarr подключен
docker logs lidarr | grep -i prowlarr

# Проверь индексаторы в Lidarr
# Settings → Indexers → Должно быть 3+ индексатора
```

### qBittorrent не качает
**Решение:**
```bash
# Проверь порты
docker exec qbittorrent netstat -tuln | grep 6881

# Проверь логи
docker logs qbittorrent | tail -50

# Проверь настройки Download Client в Lidarr
# Settings → Download Clients → Test Connection
```

### Navidrome не видит музыку
**Решение:**
```bash
# Проверь права доступа
ssh root@167.253.157.7
chown -R 1000:1000 /mnt/sata/data/media/music/
chmod -R 755 /mnt/sata/data/media/music/

# Ручной скан
docker exec navidrome curl -X POST http://localhost:4533/api/scan

# Проверь логи
docker logs navidrome | tail -30
```

### Spotify Import List не работает
**Проблемы:**
- Не авторизовался в Lidarr
- Неправильный Client ID/Secret
- Redirect URI не совпадает

**Решение:**
1. Удали Import List в Lidarr
2. Проверь Redirect URI в Spotify Dashboard:
   - Должно быть: `https://lidarr.litwein.bond/oauth/callback`
3. Добавь Import List заново и авторизуйся

---

## 🎯 Что дальше?

### Улучшения стека

1. **Добавить Slskd** (Soulseek для редкой музыки):
   - См. `MUSIC_STACK_AUDIT.md` → Раздел "Slskd"

2. **Настроить уведомления:**
   - Lidarr → Settings → Connect → Telegram/Discord

3. **Автоматизировать Spotify плейлисты:**
   - Создать плейлист "Server Download"
   - Добавлять туда новую музыку
   - Lidarr будет автоматически качать

4. **Настроить Last.fm scrobbling:**
   - Navidrome → Settings → Last.fm → Link Account
   - Symfonium автоматически будет scrobblить

5. **Создать smart playlists:**
   - Navidrome → Playlists → Create Smart Playlist
   - Критерии: Год, Жанр, Рейтинг

---

## 📊 Ожидаемые результаты

После настройки:

- ✅ Lidarr автоматически мониторит Spotify плейлисты
- ✅ Новая музыка скачивается с торрентов
- ✅ Файлы автоматически организованы (Артист/Альбом/Трек)
- ✅ Navidrome видит всю библиотеку
- ✅ Symfonium стримит музыку на телефон
- ✅ Offline режим работает (умное кэширование)
- ✅ Радио/Mixes генерируются автоматически

---

## 🔒 Безопасность

### Обязательно:
- [x] NPM → Access Lists → Basic Auth для Lidarr/Prowlarr/qBit
- [ ] Изменить пароли по умолчанию в qBittorrent
- [ ] Включить VPN для qBittorrent (если нужно)

### Опционально:
- [ ] Fail2ban для защиты от брутфорса
- [ ] CloudFlare Tunnel вместо прямого доступа

---

## 📞 Контакты и ресурсы

### Документация:
- Lidarr: https://wiki.servarr.com/lidarr
- Prowlarr: https://wiki.servarr.com/prowlarr
- Navidrome: https://navidrome.org/docs/
- Symfonium: https://symfonium.app/docs/

### Reddit Communities:
- r/selfhosted
- r/homeserver
- r/navidrome
- r/usenet (для альтернативных источников)

### Telegram:
- @selfhosted_ru
- @arrstack

---

**Готово!** 🎉 Теперь у тебя полноценная замена Spotify с автоматическим скачиванием.

**Следующий шаг:** Открой Prowlarr и добавь первый индексатор.

