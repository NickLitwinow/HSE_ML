# 🎵 Music Stack: Cheat Sheet

> Быстрый справочник по командам и настройкам

---

## 🔑 Доступы

```bash
# SSH
ssh root@167.253.157.7

# Домены
https://music.litwein.bond         # Navidrome (работает)
https://lidarr.litwein.bond        # Lidarr (нужно настроить NPM)
https://prowlarr.litwein.bond      # Prowlarr (нужно настроить NPM)
https://qbit.litwein.bond          # qBittorrent (нужно настроить NPM)

# Прямой доступ по IP
http://167.253.157.7:4533          # Navidrome
http://167.253.157.7:8686          # Lidarr
http://167.253.157.7:9696          # Prowlarr
http://167.253.157.7:8080          # qBittorrent
```

---

## 🔐 API Keys

```bash
Lidarr:   308430cf14d547c9aada25d71ad925e4
Prowlarr: ff149a593ccc4580b45b95a5fd1c8a36
```

---

## 📁 Пути

```bash
# На сервере
Конфиги (SSD):    /root/Homeserver/data/ssd/
Музыка (SATA):    /mnt/sata/data/media/music/
Загрузки (SATA):  /mnt/sata/data/downloads/

# В контейнерах
Lidarr music:     /music
Lidarr downloads: /downloads
Navidrome music:  /music (readonly)
qBittorrent:      /downloads
```

---

## 🐳 Docker Commands

```bash
# Статус всех контейнеров
docker ps | grep -E '(navidrome|lidarr|prowlarr|qbit)'

# Статус музыкального стека
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' | grep -E '(NAME|navidrome|lidarr|prowlarr|qbit)'

# Логи (real-time)
docker logs navidrome -f
docker logs lidarr -f
docker logs prowlarr -f
docker logs qbittorrent -f

# Логи (последние 50 строк)
docker logs navidrome --tail 50
docker logs lidarr --tail 50

# Перезапуск контейнера
docker restart navidrome
docker restart lidarr

# Перезапуск всего стека
cd /root/Homeserver
docker-compose restart navidrome lidarr prowlarr qbittorrent

# Остановить/запустить стек
docker-compose down
docker-compose up -d

# Обновить контейнер
docker-compose pull navidrome
docker-compose up -d navidrome

# Ресурсы контейнеров
docker stats --no-stream | grep -E '(NAME|navidrome|lidarr|prowlarr|qbit)'

# Инспекция контейнера
docker inspect navidrome | jq '.[0].Mounts'
docker inspect lidarr -f '{{range .Mounts}}{{.Source}} -> {{.Destination}}{{println}}{{end}}'
```

---

## 💾 Файловая система

```bash
# Проверить место на диске
df -h /mnt/sata
du -sh /mnt/sata/data/media/music/
du -sh /mnt/sata/data/downloads/

# Список файлов в музыке
ls -lah /mnt/sata/data/media/music/
tree -L 2 /mnt/sata/data/media/music/

# Права доступа (если Navidrome не видит файлы)
chown -R 1000:1000 /mnt/sata/data/media/music/
chmod -R 755 /mnt/sata/data/media/music/

# Очистить кэш Navidrome
docker exec navidrome rm -rf /data/cache/*
docker restart navidrome

# Очистить загрузки (старше 7 дней)
find /mnt/sata/data/downloads/ -type f -mtime +7 -delete
```

---

## 🔍 Диагностика

```bash
# Проверить, что музыка видна в контейнерах
docker exec navidrome ls -lah /music/
docker exec lidarr ls -lah /music/

# Проверить порты
netstat -tuln | grep -E '(4533|8686|9696|8080|6881)'

# Проверить подключение к базе данных
docker exec postgres psql -U postgres -c '\l'

# Проверить RAM
free -h

# Проверить CPU
top -bn1 | grep "Cpu(s)"

# Проверить swap
swapon --show

# Проверить сеть
docker network ls
docker network inspect internal-net
docker network inspect proxy-net

# Проверить лимиты контейнеров
docker inspect navidrome | jq '.[0].HostConfig.Memory'
docker inspect lidarr | jq '.[0].HostConfig.Memory'
```

---

## 🎵 Navidrome

```bash
# Ручной скан библиотеки
docker exec navidrome curl -X POST http://localhost:4533/api/scan

# Проверить статистику
docker logs navidrome | grep "library"

# Рестарт
docker restart navidrome

# Изменить переменные (в docker-compose.yml)
# ND_SCANSCHEDULE: 1h (по умолчанию)
# ND_LOGLEVEL: info
# ND_SESSIONTIMEOUT: 24h

# Применить изменения
cd /root/Homeserver
docker-compose up -d navidrome
```

---

## 📊 Lidarr

```bash
# API запрос (статус)
curl -H "X-Api-Key: 308430cf14d547c9aada25d71ad925e4" \
  http://localhost:8686/api/v1/system/status | jq

# API запрос (список артистов)
curl -H "X-Api-Key: 308430cf14d547c9aada25d71ad925e4" \
  http://localhost:8686/api/v1/artist | jq

# API запрос (поиск артиста)
curl -H "X-Api-Key: 308430cf14d547c9aada25d71ad925e4" \
  "http://localhost:8686/api/v1/artist/lookup?term=Radiohead" | jq

# Проверить download clients
docker exec lidarr cat /config/config.xml | grep -A 10 DownloadClient

# Проверить root folder
docker exec lidarr cat /config/config.xml | grep RootFolder

# Ручной ресканирование
# (через WebUI: Settings → Media Management → Rescan Artist Folder)
```

---

## 🔎 Prowlarr

```bash
# API запрос (список индексаторов)
curl -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  http://localhost:9696/api/v1/indexer | jq

# API запрос (тест индексатора)
curl -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  http://localhost:9696/api/v1/indexer/1/test | jq

# Синхронизация с приложениями
curl -X POST -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  http://localhost:9696/api/v1/applications/sync

# Проверить Apps
docker exec prowlarr cat /config/config.xml | grep -A 10 Applications
```

---

## 🌊 qBittorrent

```bash
# Логин (дефолтный)
Username: admin
Password: adminadmin (ИЗМЕНИ!)

# API login
curl -i --header "Referer: http://localhost:8080" \
  --data "username=admin&password=adminadmin" \
  http://localhost:8080/api/v2/auth/login

# Список торрентов
curl "http://localhost:8080/api/v2/torrents/info" \
  --cookie "SID=YOUR_SESSION_ID"

# Проверить категории
docker exec qbittorrent cat /config/qBittorrent/qBittorrent.conf | grep Category

# Проверить загрузки
ls -lah /mnt/sata/data/downloads/

# Статистика
curl "http://localhost:8080/api/v2/transfer/info" \
  --cookie "SID=YOUR_SESSION_ID" | jq
```

---

## 🔧 Настройка NPM через CLI

```bash
# (NPM обычно настраивается через WebUI)
# Но если нужно проверить конфигурацию:

docker exec npm cat /data/database.sqlite

# Список прокси хостов (если установлен sqlite3)
docker exec npm sqlite3 /data/database.sqlite \
  "SELECT id, domain_names, forward_host, forward_port FROM proxy_host;"

# Проверить Let's Encrypt сертификаты
docker exec npm ls -lah /etc/letsencrypt/live/

# Логи NPM
docker logs npm --tail 100
```

---

## 🎛️ Переменные окружения (.env)

```bash
# Редактировать .env
ssh root@167.253.157.7
nano /root/Homeserver/.env

# Основные переменные:
DOMAIN=litwein.bond
TZ=Asia/Singapore
PUID=1000
PGID=1000
SSD_ROOT=./data/ssd
SATA_ROOT=/mnt/sata/data
DB_PASSWORD=Fydx8353_

# После изменения:
cd /root/Homeserver
docker-compose down
docker-compose up -d
```

---

## 🚀 Быстрые тесты

```bash
# Тест 1: Проверить, что все контейнеры работают
docker ps | grep -E '(navidrome|lidarr|prowlarr|qbit)' | wc -l
# Должно быть: 4

# Тест 2: Проверить доступность WebUI
curl -I http://localhost:4533  # Navidrome
curl -I http://localhost:8686  # Lidarr
curl -I http://localhost:9696  # Prowlarr
curl -I http://localhost:8080  # qBittorrent

# Тест 3: Проверить маунты
docker exec navidrome ls /music
docker exec lidarr ls /music
docker exec lidarr ls /downloads

# Тест 4: Проверить RAM usage
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | \
  grep -E '(NAME|navidrome|lidarr|prowlarr|qbit)'

# Тест 5: Проверить логи на ошибки
docker logs lidarr 2>&1 | grep -i error | tail -10
docker logs prowlarr 2>&1 | grep -i error | tail -10
```

---

## 🛠️ Автоматизация

```bash
# Создать скрипт для быстрого ресканирования Navidrome
cat << 'EOF' > /root/scripts/rescan_music.sh
#!/bin/bash
docker exec navidrome curl -X POST http://localhost:4533/api/scan
echo "Navidrome rescan triggered"
EOF

chmod +x /root/scripts/rescan_music.sh

# Использование:
/root/scripts/rescan_music.sh

# Добавить в cron (каждый час)
echo "0 * * * * /root/scripts/rescan_music.sh >> /var/log/music_scan.log 2>&1" | crontab -

# Скрипт для проверки статуса стека
cat << 'EOF' > /root/scripts/music_status.sh
#!/bin/bash
echo "=== Docker Containers ==="
docker ps --format 'table {{.Names}}\t{{.Status}}' | grep -E '(NAME|navidrome|lidarr|prowlarr|qbit)'

echo -e "\n=== RAM Usage ==="
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | grep -E '(NAME|navidrome|lidarr|prowlarr|qbit)'

echo -e "\n=== Disk Space ==="
df -h /mnt/sata | grep -v tmpfs

echo -e "\n=== Music Library Size ==="
du -sh /mnt/sata/data/media/music/

echo -e "\n=== Recent Logs (Errors) ==="
docker logs lidarr --tail 20 2>&1 | grep -i error | tail -3 || echo "No errors"
docker logs prowlarr --tail 20 2>&1 | grep -i error | tail -3 || echo "No errors"
EOF

chmod +x /root/scripts/music_status.sh

# Использование:
/root/scripts/music_status.sh
```

---

## 📱 Symfonium

```bash
# Настройка в приложении:
Server URL:  https://music.litwein.bond
Server Type: Subsonic / Navidrome
Username:    [твой username из Navidrome]
Password:    [твой password из Navidrome]

# После подключения:
Settings → Cache → Max Cache Size: 10 GB
Settings → Downloads → Download Quality: Original
Settings → Playback → Gapless Playback: Enabled
```

---

## 🔐 Безопасность

```bash
# Проверить открытые порты
ufw status
netstat -tuln | grep LISTEN

# Открыть порты (если нужно)
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 6881/tcp      # qBittorrent
ufw allow 50300/tcp     # Slskd (если установлен)

# Закрыть прямой доступ к внутренним портам
ufw deny 8686/tcp       # Lidarr (только через NPM)
ufw deny 9696/tcp       # Prowlarr (только через NPM)
ufw deny 8080/tcp       # qBittorrent (только через NPM)

# Проверить логины
docker exec lidarr cat /config/config.xml | grep AuthenticationMethod
docker exec prowlarr cat /config/config.xml | grep AuthenticationMethod
```

---

## 🆘 Emergency Commands

```bash
# Полный рестарт стека
cd /root/Homeserver
docker-compose restart navidrome lidarr prowlarr qbittorrent

# Если контейнер не отвечает (hard restart)
docker stop navidrome && docker start navidrome

# Очистить всё и начать заново (ОПАСНО!)
docker-compose down
docker volume prune -f
docker-compose up -d

# Бэкап конфигураций перед изменениями
tar -czf /root/backup_music_configs_$(date +%Y%m%d).tar.gz \
  /root/Homeserver/data/ssd/{navidrome,lidarr,prowlarr,qbittorrent}

# Восстановление из бэкапа
tar -xzf /root/backup_music_configs_YYYYMMDD.tar.gz -C /
docker-compose restart navidrome lidarr prowlarr qbittorrent
```

---

## 📚 Полезные запросы

```bash
# Сколько треков в библиотеке?
docker logs navidrome | grep "library" | tail -1

# Сколько артистов в Lidarr?
curl -s -H "X-Api-Key: 308430cf14d547c9aada25d71ad925e4" \
  http://localhost:8686/api/v1/artist | jq '. | length'

# Сколько активных загрузок?
docker exec qbittorrent ls -1 /downloads/ | wc -l

# Последняя активность Lidarr?
docker logs lidarr --tail 50 | grep "Info"

# Список индексаторов в Prowlarr?
curl -s -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  http://localhost:9696/api/v1/indexer | jq '.[].name'
```

---

## 🎓 Документация

```bash
# Локальная документация
cat /root/Homeserver/MUSIC_README.md
cat /root/Homeserver/MUSIC_QUICK_SETUP.md
cat /root/Homeserver/MUSIC_STACK_AUDIT.md
cat /root/Homeserver/MUSIC_ARCHITECTURE.md
cat /root/Homeserver/MUSIC_SLSKD_ADDON.md
cat /root/Homeserver/MUSIC_STACK_SUMMARY.md

# Официальная документация
xdg-open https://navidrome.org/docs/
xdg-open https://wiki.servarr.com/lidarr
xdg-open https://wiki.servarr.com/prowlarr
```

---

## 🎯 One-Liners

```bash
# Всё в одном: статус + RAM + диск + ошибки
docker ps --format '{{.Names}}\t{{.Status}}' | grep -E '(navidrome|lidarr|prowlarr|qbit)' && \
docker stats --no-stream --format "{{.Name}}\t{{.MemUsage}}" | grep -E '(navidrome|lidarr|prowlarr|qbit)' && \
df -h /mnt/sata | tail -1 && \
docker logs lidarr --tail 10 2>&1 | grep -i error | tail -1

# Быстрый full restart музыкального стека
cd /root/Homeserver && docker-compose restart navidrome lidarr prowlarr qbittorrent && docker ps | grep -E '(navidrome|lidarr|prowlarr|qbit)'

# Проверка здоровья стека
[ $(docker ps -q -f name=navidrome -f name=lidarr -f name=prowlarr -f name=qbittorrent | wc -l) -eq 4 ] && echo "✅ All containers running" || echo "❌ Some containers down"

# Топ процессов по RAM в контейнерах
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | grep -E '(navidrome|lidarr|prowlarr|qbit)' | sort -k2 -hr
```

---

**Создано:** 29 ноября 2025  
**Версия:** 1.0  
**Для:** Быстрого доступа к командам музыкального стека

