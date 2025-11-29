# 🎵 Музыкальный стек: Документация

> **Self-hosted замена Spotify** на базе Navidrome, Lidarr, Prowlarr и qBittorrent

---

## 📚 Навигация по документации

| Документ | Описание | Когда читать |
|----------|----------|--------------|
| **[MUSIC_STACK_SUMMARY.md](MUSIC_STACK_SUMMARY.md)** | 📋 Итоговый отчёт и executive summary | **Начни отсюда** |
| **[MUSIC_QUICK_SETUP.md](MUSIC_QUICK_SETUP.md)** | 🚀 Пошаговая настройка за 20 минут | После прочтения summary |
| **[MUSIC_STACK_AUDIT.md](MUSIC_STACK_AUDIT.md)** | 🔍 Подробный аудит и troubleshooting | Если что-то не работает |
| **[MUSIC_ARCHITECTURE.md](MUSIC_ARCHITECTURE.md)** | 📐 Схемы и архитектура (Mermaid) | Для понимания как это работает |
| **[MUSIC_SLSKD_ADDON.md](MUSIC_SLSKD_ADDON.md)** | 🎵 Добавление Soulseek (опционально) | После базовой настройки |

---

## ⚡ Quick Start

### Если спешишь:

1. **Прочитай:** [MUSIC_STACK_SUMMARY.md](MUSIC_STACK_SUMMARY.md) (5 минут)
2. **Настрой:** [MUSIC_QUICK_SETUP.md](MUSIC_QUICK_SETUP.md) (20 минут)
3. **Проверь:** Добавь первого артиста и убедись, что всё работает
4. **Готово!** Теперь у тебя своя замена Spotify

---

## 🎯 Что это даёт?

### Возможности:

- ✅ **Автоматическое скачивание** музыки из Spotify плейлистов
- ✅ **Стриминг** на любые устройства (Web, Android, iOS)
- ✅ **Offline режим** (умное кэширование на телефоне)
- ✅ **Высокое качество** (FLAC/Lossless вместо 320kbps)
- ✅ **Радио/Mixes** на основе твоей библиотеки
- ✅ **Scrobbling** на Last.fm (статистика прослушиваний)
- ✅ **Self-hosted** (полный контроль, никакой цензуры)

### Экономия:

- **$120/год** (стоимость Spotify Premium)
- **Неограниченная библиотека** (не привязан к каталогу Spotify)
- **Никакой рекламы** (даже на бесплатной версии)

---

## 📊 Текущее состояние

**Дата аудита:** 29 ноября 2025

| Компонент | Статус | Готовность |
|-----------|--------|------------|
| Инфраструктура | ✅ Готова | 100% |
| Контейнеры | ✅ Запущены | 100% |
| Конфигурация | ⚠️ Не настроено | 10% |
| Интеграции | ❌ Не подключены | 0% |
| Контент | ❌ Библиотека пустая | 0% |

**Общая готовность:** 18%  
**Время до полной готовности:** 20 минут настройки

---

## 🏗️ Архитектура

### Компоненты:

```
┌─────────────────────────────────────────────────────────────┐
│                      Spotify API                            │
│            (Источник для Import Lists)                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                         Lidarr                              │
│         (Мозг: управление библиотекой)                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        Prowlarr                             │
│           (Ищейка: агрегатор трекеров)                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌───────────────────────┬─────────────────────────────────────┐
│    qBittorrent        │          Slskd (optional)           │
│   (Torrent Client)    │         (Soulseek P2P)              │
└───────────────────────┴─────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                     /music/ Storage                         │
│            (Организованная библиотека)                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                       Navidrome                             │
│              (Streaming Server)                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌───────────────────────┬─────────────────────────────────────┐
│     Web Browser       │           Symfonium                 │
│  (Desktop/Mobile)     │        (Android App)                │
└───────────────────────┴─────────────────────────────────────┘
```

**Подробные схемы:** См. [MUSIC_ARCHITECTURE.md](MUSIC_ARCHITECTURE.md)

---

## 🔑 Ключевая информация

### Доступы:

```
Сервер:      ssh root@167.253.157.7
Домен:       litwein.bond

Navidrome:   https://music.litwein.bond ✅
Lidarr:      https://lidarr.litwein.bond ⚠️ (нужно настроить NPM)
Prowlarr:    https://prowlarr.litwein.bond ⚠️ (нужно настроить NPM)
qBittorrent: https://qbit.litwein.bond ⚠️ (нужно настроить NPM)
```

### API Keys:

```
Lidarr API Key:   308430cf14d547c9aada25d71ad925e4
Prowlarr API Key: ff149a593ccc4580b45b95a5fd1c8a36
```

### Пути:

```
Конфигурации (SSD):  /root/Homeserver/data/ssd/
Музыка (SATA):       /mnt/sata/data/media/music/
Загрузки (SATA):     /mnt/sata/data/downloads/
```

---

## 🚨 Критические проблемы (требуют исправления)

1. **⚠️ Библиотека пустая** (0 треков)
   - **Решение:** Добавить первого артиста через Lidarr

2. **⚠️ Lidarr не может искать музыку** (нет индексаторов)
   - **Решение:** Настроить Prowlarr → Добавить трекеры → Подключить к Lidarr

3. **⚠️ Spotify интеграция отключена**
   - **Решение:** Создать Spotify App → Добавить Client ID/Secret

4. **⚠️ Нет Proxy для внутренних сервисов**
   - **Решение:** Настроить NPM (добавить домены + Basic Auth)

---

## 📖 Пошаговая инструкция

### Шаг 1: Прочитай Summary (5 минут)
- Открой [MUSIC_STACK_SUMMARY.md](MUSIC_STACK_SUMMARY.md)
- Ознакомься с текущим состоянием
- Пойми, что нужно сделать

### Шаг 2: Настрой базовый стек (20 минут)
- Следуй [MUSIC_QUICK_SETUP.md](MUSIC_QUICK_SETUP.md)
- Настрой по порядку:
  1. NPM (прокси)
  2. Prowlarr (индексаторы)
  3. qBittorrent (категории)
  4. Lidarr (download clients + root folder)
  5. Тест (добавь первого артиста)

### Шаг 3: Spotify интеграция (5 минут)
- Создай Spotify Developer App
- Настрой Import Lists в Lidarr
- Создай плейлист для мониторинга

### Шаг 4: Мобильное приложение (10 минут)
- Установи Symfonium (Android, $5)
- Подключись к Navidrome
- Настрой offline кэширование

### Шаг 5 (опционально): Slskd (15 минут)
- Следуй [MUSIC_SLSKD_ADDON.md](MUSIC_SLSKD_ADDON.md)
- Добавь Soulseek для редкой музыки

**Итого:** 40-55 минут до полной готовности

---

## 🔧 Troubleshooting

### Lidarr не находит музыку
**Проблема:** `No available indexers`  
**Решение:** [MUSIC_STACK_AUDIT.md](MUSIC_STACK_AUDIT.md) → "Проблема #2"

### qBittorrent не качает
**Проблема:** Торренты не стартуют  
**Решение:** [MUSIC_QUICK_SETUP.md](MUSIC_QUICK_SETUP.md) → "Troubleshooting"

### Navidrome не видит музыку
**Проблема:** Tracks = 0  
**Решение:** [MUSIC_STACK_AUDIT.md](MUSIC_STACK_AUDIT.md) → "Команды для диагностики"

### Подробнее:
- [MUSIC_STACK_AUDIT.md](MUSIC_STACK_AUDIT.md) → Раздел "Troubleshooting"

---

## 📱 Рекомендуемые клиенты

### Android:
- **Symfonium** (Платный, $5) — **Лучший выбор**
  - Умное кэширование
  - Радио/Mixes
  - Скробблинг на Last.fm
  - Gapless playback

- **Subtracks** (Бесплатный) — Альтернатива
- **DSub** (Платный, $3) — Старый, но стабильный

### iOS:
- **play:Sub** ($5)
- **Amperfy** (Бесплатный, open-source)

### Desktop:
- **Web Browser** (встроенный в Navidrome)
- **Sonixd** (Electron app, cross-platform)
- **Sublime Music** (Linux)

---

## 🎓 Дополнительные ресурсы

### Официальная документация:
- Navidrome: https://navidrome.org/docs/
- Lidarr: https://wiki.servarr.com/lidarr
- Prowlarr: https://wiki.servarr.com/prowlarr
- Symfonium: https://symfonium.app/docs/

### Community:
- Reddit: r/selfhosted, r/navidrome, r/lidarr
- Discord: Servarr (Lidarr/Prowlarr), Navidrome
- Telegram: @selfhosted_ru, @arrstack

### Полезные ссылки:
- Spotify Developer: https://developer.spotify.com/dashboard
- Last.fm API: https://www.last.fm/api/account/create
- RuTracker: https://rutracker.org (регистрация)

---

## 📝 Changelog

### 2025-11-29: Initial Audit
- ✅ Создана полная документация (5 файлов)
- ✅ Проведён аудит текущего состояния
- ✅ Выявлены критические проблемы
- ✅ Составлен план настройки
- ✅ Добавлены визуальные схемы (Mermaid)

---

## 🙏 Credits

**Проект использует:**
- [Navidrome](https://github.com/navidrome/navidrome) by Deluan Quintao
- [Lidarr](https://github.com/Lidarr/Lidarr) by Servarr Team
- [Prowlarr](https://github.com/Prowlarr/Prowlarr) by Servarr Team
- [qBittorrent](https://github.com/qbittorrent/qBittorrent) by qBittorrent Team
- [Slskd](https://github.com/slskd/slskd) by Slskd Team
- [Symfonium](https://symfonium.app/) by Tolriq

**Документация создана:** AI Assistant (Claude Sonnet 4.5)  
**Дата:** 29 ноября 2025

---

## 📞 Поддержка

Если возникли проблемы:
1. Проверь [MUSIC_STACK_AUDIT.md](MUSIC_STACK_AUDIT.md) → Troubleshooting
2. Посмотри логи: `docker logs <service_name> -f`
3. Спроси в комьюнити (Reddit/Discord)
4. Открой issue в репозитории (если проблема в коде)

---

## 🎉 Готов начать?

1. **Открой:** [MUSIC_STACK_SUMMARY.md](MUSIC_STACK_SUMMARY.md)
2. **Потом:** [MUSIC_QUICK_SETUP.md](MUSIC_QUICK_SETUP.md)
3. **Через 20 минут:** У тебя будет своя замена Spotify!

---

**Happy Listening! 🎵**

