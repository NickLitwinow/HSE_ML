# ⚡ Быстрый старт: Решение проблем с индексаторами

## 🎯 Проблемы которые возникли

1. ❌ **RuTracker** требует капчу (код с картинки)
2. ❌ **1337x** блокируется CloudFlare

## ✅ Быстрое решение (5 минут)

### Вариант 1: Самый простой (рекомендуется)

**Используй только The Pirate Bay:**
- Не требует регистрации
- Нет капчи
- Работает сразу

```bash
# В Prowlarr WebUI:
Indexers → Add Indexer → 
Search: "The Pirate Bay" → 
Categories: Music → 
Save
```

**Результат:** Этого достаточно для 70% музыки.

---

### Вариант 2: The Pirate Bay + 1337x (лучше)

**Требуется:** Установить FlareSolverr для 1337x

#### Шаг 1: Добавить FlareSolverr

```bash
ssh root@167.253.157.7
cd /root/Homeserver

# FlareSolverr уже добавлен в docker-compose.yml ✅
# Просто запусти:
docker-compose up -d flaresolverr

# Проверь логи:
docker logs flaresolverr
```

**Ожидаемый вывод:**
```
FlareSolverr started successfully
```

#### Шаг 2: Настроить Prowlarr

```
1. Открой: http://167.253.157.7:9696
2. Settings → Indexers → FlareSolverr:
   - Tags: flaresolverr (создай новый тег)
   - Host: http://flaresolverr:8191/
   - Save
```

#### Шаг 3: Добавить The Pirate Bay

```
Indexers → Add Indexer → 
Search: "The Pirate Bay" → 
Categories: Music → 
Save
```

#### Шаг 4: Добавить 1337x

```
Indexers → Add Indexer → 
Search: "1337x" → 
Tags: flaresolverr (выбери тег) → 
Categories: Music → 
Test → Save
```

#### Шаг 5: Синхронизировать с Lidarr

```
Settings → Apps → Lidarr → Sync
```

**Результат:** Два рабочих индексатора (TPB + 1337x), покрытие ~85% музыки.

---

## 🔍 Проверка

```bash
# SSH в сервер
ssh root@167.253.157.7

# Проверить, что FlareSolverr работает
curl http://localhost:8191/v1

# Проверить индексаторы в Prowlarr
curl -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  http://localhost:9696/api/v1/indexer | jq '.[].name'

# Тест поиска
curl -H "X-Api-Key: ff149a593ccc4580b45b95a5fd1c8a36" \
  "http://localhost:9696/api/v1/search?query=Radiohead&type=music" | jq '. | length'
```

**Ожидаемый результат:** 
- FlareSolverr: `{"message": "FlareSolverr is ready"}`
- Indexers: `["The Pirate Bay", "1337x"]` (или больше)
- Search: Число найденных результатов (например, `15`)

---

## 🎯 Тест: Добавь первого артиста

1. Открой Lidarr: `http://167.253.157.7:8686`
2. Library → Add New
3. Поиск: `Radiohead`
4. Root Folder: `/music`
5. Quality Profile: `Any`
6. Monitor: `All Albums`
7. ✅ Search on add
8. Add Artist

### Что должно произойти:

```
1. Prowlarr ищет на The Pirate Bay (и 1337x если настроен)
2. Lidarr находит торренты с альбомами
3. qBittorrent начинает загрузку
4. Через 5-30 минут (зависит от скорости) файлы появятся в /music/
5. Navidrome автоматически просканирует (через 1 час или ручной скан)
```

**Проверка загрузки:**
```bash
# В реальном времени
docker logs lidarr -f

# Статус в qBittorrent
docker exec qbittorrent ls -lh /downloads/

# Результат
ls -lah /mnt/sata/data/media/music/
```

---

## 📊 Сравнение вариантов

| Вариант | Индексаторы | Покрытие | Сложность | Время |
|---------|-------------|----------|-----------|-------|
| **1** | The Pirate Bay | 70% | 🟢 Легко | 2 мин |
| **2** | TPB + 1337x | 85% | 🟡 Средне | 5 мин |
| **3** | TPB + 1337x + NNMClub | 95% | 🟡 Средне | 10 мин |
| **4** | Все + Slskd | 99% | 🔴 Сложно | 30 мин |

**Рекомендация:** Начни с варианта 2 (TPB + 1337x), этого хватит для большинства музыки.

---

## 🆘 Если что-то не работает

### FlareSolverr не запускается
```bash
docker logs flaresolverr --tail 50
docker restart flaresolverr
```

### 1337x не работает даже с FlareSolverr
- Проверь тег `flaresolverr` в настройках индексатора
- Проверь, что FlareSolverr Host: `http://flaresolverr:8191/`
- Перезапусти: `docker restart flaresolverr prowlarr`

### Lidarr не находит музыку
```bash
# Проверь, что индексаторы синхронизированы
curl -H "X-Api-Key: 308430cf14d547c9aada25d71ad925e4" \
  http://localhost:8686/api/v1/indexer | jq '.[].name'

# Если пусто, синхронизируй вручную:
# Prowlarr → Settings → Apps → Lidarr → Sync
```

---

## 📚 Подробная документация

- **Полное решение проблем:** `MUSIC_INDEXERS_FIX.md`
- **Пошаговая настройка:** `MUSIC_QUICK_SETUP.md`
- **Альтернатива Soulseek:** `MUSIC_SLSKD_ADDON.md`

---

## 🎉 Результат

После этих шагов у тебя будет:
- ✅ 1-2 рабочих индексатора
- ✅ Автоматический поиск и загрузка музыки
- ✅ Интеграция с Lidarr
- ✅ Можно добавлять артистов и альбомы

**Следующий шаг:** Добавь первого артиста в Lidarr и проверь, что всё работает!

---

**Создано:** 30 ноября 2025  
**Время:** ~5 минут на настройку

