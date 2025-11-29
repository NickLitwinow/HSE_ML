# 🎵 Расширение музыкального стека: Slskd (Soulseek)

## 🤔 Зачем нужен Slskd?

**Проблема торрентов:**
- Редкие синглы не раздаются
- FLAC качество не всегда доступно
- Некоторые артисты есть только в частных коллекциях

**Решение: Soulseek**
- P2P сеть, специализированная на музыке
- Огромная база редких треков
- Lidarr умеет интегрироваться с Slskd

---

## 📦 Добавление в docker-compose.yml

### Добавь в раздел "ARR STACK":

```yaml
  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    restart: unless-stopped
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - SLSKD_HTTP_PORT=5030
      - SLSKD_SLSK_LISTEN_PORT=50300
      - SLSKD_SHARED_DIR=/music
      - SLSKD_INCOMPLETE_DIR=/downloads/slskd/incomplete
      - SLSKD_DOWNLOADS_DIR=/downloads/slskd/complete
      - SLSKD_NO_AUTH=false
    volumes:
      - ${SSD_ROOT}/slskd:/app
      - ${SATA_ROOT}/media/music:/music:ro
      - ${SATA_ROOT}/downloads/slskd:/downloads/slskd
    ports:
      - "50300:50300"
      - "50301:50301"
    networks:
      - internal-net
      - proxy-net
    mem_limit: 512m
    logging: *default-logging
```

---

## 🚀 Установка

### 1. Остановить стек
```bash
ssh root@167.253.157.7
cd /root/Homeserver
docker-compose down
```

### 2. Обновить docker-compose.yml
Скопируй блок выше в секцию после `qbittorrent`.

### 3. Создать директории
```bash
mkdir -p /mnt/sata/data/downloads/slskd/{incomplete,complete}
chown -R 1000:1000 /mnt/sata/data/downloads/slskd
```

### 4. Запустить стек
```bash
docker-compose up -d slskd
```

### 5. Проверить логи
```bash
docker logs slskd -f
```

**Ожидаемый вывод:**
```
[Info] Slskd version x.x.x starting
[Info] HTTP server listening on port 5030
[Info] Soulseek listening on port 50300
```

---

## ⚙️ Настройка Slskd

### Первый вход

1. Открой: `http://167.253.157.7:5030` (или настрой NPM для `slskd.litwein.bond`)
2. Создай пользователя:
   - Username: `admin`
   - Password: `Fydx8353_` (или свой)

### Подключение к Soulseek

**Settings → Connection:**
- Username: `придумай_уникальный` (будет виден другим юзерам)
- Password: `твой_пароль_soulseek`
- Port: `50300`
- ✅ Enable Distributed Network
- Save → Connect

**Важно:** Регистрация не требуется, просто придумай username.

### Настройка шаринга

**Settings → Shares:**
- Shared Directories: `/music` (уже настроено)
- ✅ Share Files
- ✅ Allow Uploads from Friends
- Cache: Update Cache (для индексации библиотеки)

**Зачем шарить:**
- Без шаринга многие пользователи не дадут скачать
- Karma system: чем больше отдаёшь, тем быстрее качаешь

### Настройка загрузок

**Settings → Downloads:**
- Incomplete Directory: `/downloads/slskd/incomplete`
- Complete Directory: `/downloads/slskd/complete`
- Max Concurrent Downloads: `3`
- ✅ Overwrite Existing Files: `false`

---

## 🔗 Интеграция с Lidarr

### 1. Открой Lidarr
```
https://lidarr.litwein.bond
```

### 2. Добавь Slskd как Download Client

**Settings → Download Clients → Add → Slskd:**
- Name: `Slskd (Soulseek)`
- Host: `slskd`
- Port: `5030`
- Username: `admin`
- Password: `Fydx8353_`
- API Key: (из Slskd WebUI → Settings → API Key)
- ✅ Enable
- ✅ Remove Completed
- Priority: `10` (ниже qBittorrent, чтобы торренты были приоритетнее)
- Test → Save

### 3. Настройка приоритета источников

**Settings → Indexers → Advanced:**
- Для каждого индексатора (кроме Soulseek):
  - Priority: `25`
  
**Settings → Download Clients:**
- qBittorrent Priority: `1` (высший приоритет)
- Slskd Priority: `10` (резервный)

**Логика:**
1. Lidarr сначала ищет на торрентах
2. Если не нашёл → ищет в Soulseek
3. Если нашёл и там, и там → выбирает торрент (быстрее)

---

## 🧪 Тест

### Ручной поиск в Slskd

1. Открой Slskd WebUI: `http://167.253.157.7:5030`
2. Searches → New Search
3. Поиск: `Radiohead OK Computer FLAC`
4. Wait 10-20 seconds
5. Результаты появятся в списке
6. Кликни на пользователя → Browse
7. Выбери файлы → Download

**Результат:** Файлы появятся в `/downloads/slskd/complete/`

### Автоматический поиск через Lidarr

1. Lidarr → Library → Add New Artist
2. Поиск: `редкий артист` (например, `Молчат Дома`)
3. Add Artist → Search on Add
4. Activity → Queue → Должен появиться Slskd в источниках

---

## 📊 Мониторинг

### Статус подключения
```bash
docker exec slskd curl http://localhost:5030/api/v0/session/status
```

**Ожидаемый вывод:**
```json
{
  "connected": true,
  "username": "твой_username",
  "state": "Connected"
}
```

### Активные загрузки
```bash
docker exec slskd curl http://localhost:5030/api/v0/downloads
```

### Размер библиотеки
```bash
du -sh /mnt/sata/data/downloads/slskd/complete/
```

---

## 🔧 Настройка NPM для Slskd

### Добавь Proxy Host

**Domain Names:** `slskd.litwein.bond`  
**Scheme:** `http`  
**Forward Hostname / IP:** `slskd`  
**Forward Port:** `5030`  
**Advanced:**
```nginx
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

**SSL:** Let's Encrypt (автоматически)  
**Access List:** Internal Services (Basic Auth)

---

## 🚨 Troubleshooting

### Slskd не подключается к сети

**Симптомы:**
```
[Error] Failed to connect to Soulseek network
```

**Решение:**
```bash
# Проверь порты
docker exec slskd netstat -tuln | grep 50300

# Проверь firewall
ufw status
ufw allow 50300/tcp
ufw allow 50301/tcp

# Перезапусти контейнер
docker restart slskd
```

### Lidarr не видит Slskd

**Решение:**
1. Проверь API Key в Slskd: Settings → API → Copy Key
2. Lidarr → Download Clients → Edit Slskd → Вставь API Key
3. Test Connection

### Загрузки застревают в Incomplete

**Причина:** Пользователь офлайн или медленная скорость

**Решение:**
- Slskd → Settings → Downloads → Max Concurrent Downloads: `5`
- Включи больше источников (повтори поиск)

---

## 📈 Оптимизация

### 1. Увеличить приоритет для редких артистов

**Lidarr → Settings → Tags:**
- Создай тег: `rare`
- Применяй к редким артистам
- Lidarr → Settings → Download Clients → Slskd → Tags: `rare`

**Результат:** Для артистов с тегом `rare` Lidarr будет приоритетно использовать Soulseek.

### 2. Настроить автоматическую очистку

**Создай cron job на сервере:**
```bash
crontab -e
```

Добавь:
```bash
# Очистка завершённых загрузок Slskd каждый день в 3:00
0 3 * * * find /mnt/sata/data/downloads/slskd/complete -type f -mtime +7 -delete
```

### 3. Увеличить лимиты

**В docker-compose.yml:**
```yaml
slskd:
  mem_limit: 1g  # Было 512m
  environment:
    - SLSKD_MAX_CONCURRENT_DOWNLOADS=10  # Было 3
```

---

## 📊 Статистика и аналитика

### Slskd API (примеры запросов)

**Статус:**
```bash
curl -H "X-API-Key: YOUR_API_KEY" http://localhost:5030/api/v0/session
```

**Загрузки:**
```bash
curl -H "X-API-Key: YOUR_API_KEY" http://localhost:5030/api/v0/downloads | jq
```

**Поиск:**
```bash
curl -X POST -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"searchText": "Radiohead FLAC"}' \
  http://localhost:5030/api/v0/searches
```

---

## 🔒 Безопасность

### 1. Не шарить личную информацию
**Settings → Shares:**
- Исключи директории с личными файлами
- Шарь только `/music` (публичная библиотека)

### 2. Ограничить upload скорость
**Settings → Options → Connection:**
- Upload Speed Limit: `2000 KB/s` (чтобы не забить канал)

### 3. Защитить WebUI
- NPM → Access List → Basic Auth
- Или Slskd → Settings → Security → Enable Password

---

## 🎯 Ожидаемые результаты

После установки Slskd:

- ✅ Lidarr может находить редкую музыку, которой нет на торрентах
- ✅ FLAC/Lossless качество доступно для большинства треков
- ✅ Автоматическое скачивание из двух источников (Torrents + Soulseek)
- ✅ Karma system работает (шаришь музыку → быстрее качаешь)
- ✅ Увеличивается полнота библиотеки

---

## 📚 Дополнительные ресурсы

- Slskd Docs: https://github.com/slskd/slskd/wiki
- Lidarr + Slskd: https://wiki.servarr.com/lidarr/supported#slskd
- Soulseek Reddit: r/Soulseek

---

**Готово!** Теперь у тебя два источника музыки: торренты + Soulseek. 🎉

