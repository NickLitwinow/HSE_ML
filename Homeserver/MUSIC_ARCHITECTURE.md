# 🎵 Архитектура музыкального стека

## 📐 Общая схема компонентов

```mermaid
graph TB
    subgraph "🌐 Internet"
        A[Spotify API]
        B[RuTracker]
        C[1337x Torrents]
        D[Soulseek P2P]
    end

    subgraph "🔒 Nginx Proxy Manager"
        NPM[NPM<br/>proxy-net]
    end

    subgraph "🎵 Music Stack - internal-net"
        L[Lidarr<br/>8686<br/>512MB]
        P[Prowlarr<br/>9696<br/>256MB]
        Q[qBittorrent<br/>8080<br/>512MB]
        S[Slskd<br/>5030<br/>512MB]
        N[Navidrome<br/>4533<br/>256MB]
    end

    subgraph "💾 Storage"
        SSD[SSD<br/>Configs & DBs<br/>/root/Homeserver/data/ssd/]
        SATA[SATA 2TB<br/>Media & Downloads<br/>/mnt/sata/data/]
    end

    subgraph "📱 Clients"
        W[Web Browser<br/>music.litwein.bond]
        M[Symfonium<br/>Android App]
    end

    A -->|Import Lists| L
    L -->|Search Request| P
    P -->|Find Torrents| B
    P -->|Find Torrents| C
    L -->|Download Command| Q
    L -->|Download Command| S
    S -->|P2P Search| D
    Q -->|Torrent Download| B
    Q -->|Torrent Download| C
    Q -->|Save to| SATA
    S -->|Save to| SATA
    L -->|Organize Files| SATA
    N -->|Scan Music| SATA
    L -.Config.-> SSD
    P -.Config.-> SSD
    Q -.Config.-> SSD
    S -.Config.-> SSD
    N -.Database.-> SSD
    NPM -->|Proxy| N
    NPM -->|Proxy| L
    NPM -->|Proxy| P
    NPM -->|Proxy| Q
    NPM -->|Proxy| S
    W -->|HTTPS| NPM
    M -->|Subsonic API| NPM

    style L fill:#ff6b6b
    style P fill:#4ecdc4
    style Q fill:#45b7d1
    style S fill:#96ceb4
    style N fill:#feca57
    style NPM fill:#ee5a6f
    style SSD fill:#c7ecee
    style SATA fill:#dfe6e9
```

---

## 🔄 Workflow: От поиска до прослушивания

```mermaid
sequenceDiagram
    actor User
    participant Spotify
    participant Lidarr
    participant Prowlarr
    participant qBittorrent
    participant Storage
    participant Navidrome
    participant Symfonium

    User->>Spotify: Добавляет трек в плейлист "Server Download"
    Spotify->>Lidarr: Import List синхронизируется (каждые 6 часов)
    Lidarr->>Lidarr: Определяет артиста и альбом
    Lidarr->>Prowlarr: Ищет "Artist - Album"
    Prowlarr->>Prowlarr: Ищет на RuTracker, 1337x, TPB
    Prowlarr-->>Lidarr: Возвращает 5 результатов
    Lidarr->>Lidarr: Выбирает лучший (Quality + Seeds)
    Lidarr->>qBittorrent: Отправляет .torrent в категорию "lidarr"
    qBittorrent->>qBittorrent: Скачивает в /downloads/lidarr/
    qBittorrent-->>Lidarr: Уведомление "Download Complete"
    Lidarr->>Storage: Переименовывает и перемещает в /music/Artist/Album/
    Lidarr->>Navidrome: (Опционально) Триггер ресканирования через webhook
    Note over Navidrome: Автоматический скан каждый час (ND_SCANSCHEDULE=1h)
    Navidrome->>Storage: Сканирует /music/
    Navidrome->>Navidrome: Обновляет базу (tracks, albums, artists)
    User->>Symfonium: Открывает приложение
    Symfonium->>Navidrome: Синхронизирует библиотеку (Subsonic API)
    Navidrome-->>Symfonium: Возвращает обновленную библиотеку
    User->>Symfonium: Выбирает новый альбом
    Symfonium->>Navidrome: Стримит трек
    Navidrome-->>Symfonium: Отправляет аудио (FLAC/MP3)
    Symfonium->>Symfonium: (Опционально) Кэширует для offline
```

---

## 🗂️ Структура данных

```mermaid
graph LR
    subgraph "SSD - Configs (/root/Homeserver/data/ssd/)"
        L1[lidarr/<br/>config.xml<br/>lidarr.db]
        P1[prowlarr/<br/>config.xml<br/>prowlarr.db]
        Q1[qbittorrent/<br/>qBittorrent.conf<br/>torrents/]
        S1[slskd/<br/>slskd.yml<br/>cache/]
        N1[navidrome/<br/>navidrome.db<br/>cache/]
    end

    subgraph "SATA - Data (/mnt/sata/data/)"
        D1[downloads/<br/>lidarr/<br/>slskd/]
        M1[media/music/<br/>Artist1/<br/>Artist2/]
    end

    Q1 -.Download.-> D1
    S1 -.Download.-> D1
    L1 -.Process.-> D1
    L1 -.Move.-> M1
    N1 -.Scan.-> M1

    style L1 fill:#ff6b6b,color:#fff
    style P1 fill:#4ecdc4,color:#fff
    style Q1 fill:#45b7d1,color:#fff
    style S1 fill:#96ceb4,color:#fff
    style N1 fill:#feca57,color:#fff
```

---

## 🔌 Сетевая архитектура

```mermaid
graph TB
    subgraph "🌐 External"
        I[Internet]
    end

    subgraph "Docker Network: proxy-net (172.18.0.0/16)"
        NPM[nginx-proxy-manager<br/>172.18.0.2]
        N[navidrome<br/>172.18.0.10]
        L[lidarr<br/>172.18.0.11]
        P[prowlarr<br/>172.18.0.12]
        Q[qbittorrent<br/>172.18.0.13]
        S[slskd<br/>172.18.0.14]
    end

    subgraph "Docker Network: internal-net"
        L2[lidarr]
        P2[prowlarr]
        Q2[qbittorrent]
        S2[slskd]
        N2[navidrome]
        PG[postgres]
        R[redis]
    end

    I -->|:80/:443| NPM
    NPM -->|music.litwein.bond| N
    NPM -->|lidarr.litwein.bond| L
    NPM -->|prowlarr.litwein.bond| P
    NPM -->|qbit.litwein.bond| Q
    NPM -->|slskd.litwein.bond| S

    L2 <-->|API Calls| P2
    L2 <-->|Download API| Q2
    L2 <-->|Download API| S2

    style NPM fill:#ee5a6f,color:#fff
    style N fill:#feca57
    style L fill:#ff6b6b,color:#fff
    style P fill:#4ecdc4,color:#fff
    style Q fill:#45b7d1,color:#fff
    style S fill:#96ceb4,color:#fff
```

---

## 🔐 Security Layers

```mermaid
graph TD
    subgraph "Layer 1: Network"
        FW[Firewall<br/>UFW]
        FW -->|Allow| P80[Port 80/443<br/>NPM]
        FW -->|Allow| P6881[Port 6881<br/>qBittorrent]
        FW -->|Allow| P50300[Port 50300<br/>Slskd]
        FW -->|Block| OTHER[Other Ports]
    end

    subgraph "Layer 2: Reverse Proxy"
        NPM[Nginx Proxy Manager]
        NPM -->|SSL/TLS| LE[Let's Encrypt]
        NPM -->|Basic Auth| BA[Access Lists]
    end

    subgraph "Layer 3: Application"
        N[Navidrome<br/>User Auth]
        L[Lidarr<br/>No Auth by default]
        P[Prowlarr<br/>No Auth by default]
        Q[qBittorrent<br/>Basic Auth]
        S[Slskd<br/>User Auth]
    end

    subgraph "Layer 4: File System"
        FS[File Permissions<br/>PUID=1000, PGID=1000]
    end

    BA -->|Protect| L
    BA -->|Protect| P
    BA -->|Protect| Q
    BA -->|Protect| S

    style FW fill:#e74c3c,color:#fff
    style NPM fill:#ee5a6f,color:#fff
    style LE fill:#27ae60,color:#fff
    style BA fill:#f39c12,color:#fff
```

---

## 📊 Data Flow: Music Organization

```mermaid
graph TD
    A[Torrent Downloaded<br/>/downloads/lidarr/Artist - Album - 2020 - FLAC/] -->|Lidarr Processing| B{Check Quality}
    B -->|FLAC ✓| C[Extract Metadata]
    B -->|MP3 320 ✓| C
    B -->|Low Quality ✗| X[Reject & Delete]
    
    C --> D[Rename Files]
    D --> E[Apply Tags:<br/>Artist, Album, Year, Genre]
    E --> F[Create Directory Structure]
    
    F --> G[/music/<br/>Artist Name/<br/>Album Title Year/<br/>01 - Track Name.flac]
    
    G --> H[Set Permissions<br/>chmod 755]
    H --> I[Trigger Navidrome Scan]
    I --> J[Navidrome Updates DB]
    J --> K[Available for Streaming]
    
    X --> Z[Report Failed Download]

    style A fill:#45b7d1
    style G fill:#27ae60,color:#fff
    style K fill:#feca57
    style X fill:#e74c3c,color:#fff
```

---

## 🧩 Integration Points

```mermaid
graph LR
    subgraph "Prowlarr API"
        P1[Add Indexer]
        P2[Search Torrents]
        P3[Sync to Apps]
    end

    subgraph "Lidarr API"
        L1[Import Lists]
        L2[Music Search]
        L3[Download Management]
        L4[Post-Processing]
    end

    subgraph "qBittorrent API"
        Q1[Add Torrent]
        Q2[Monitor Progress]
        Q3[Complete Notification]
    end

    subgraph "Navidrome Subsonic API"
        N1[Library Sync]
        N2[Stream Music]
        N3[Scrobble]
    end

    P3 -->|Indexer Config| L1
    L2 -->|Search Request| P2
    L3 -->|Add Download| Q1
    Q3 -->|Complete| L4
    N1 <-->|Sync| M[Symfonium]
    N2 <-->|Stream| M
    N3 -->|Last.fm| LF[Last.fm API]

    style P2 fill:#4ecdc4
    style L3 fill:#ff6b6b,color:#fff
    style Q1 fill:#45b7d1,color:#fff
    style N2 fill:#feca57
```

---

## 🎯 Quality Selection Logic

```mermaid
graph TD
    A[Lidarr finds releases] --> B{Multiple sources?}
    B -->|Yes| C[Score each release]
    B -->|No| Z[Download only option]
    
    C --> D[Quality Score:<br/>FLAC = 100<br/>MP3-320 = 80<br/>MP3-V0 = 70<br/>MP3-256 = 60]
    
    C --> E[Source Score:<br/>Torrent w/ 50+ seeds = +20<br/>Torrent w/ 10-49 seeds = +10<br/>Soulseek = +5]
    
    C --> F[Size Score:<br/>Full Album = +10<br/>Single/EP = 0]
    
    D --> G[Total Score]
    E --> G
    F --> G
    
    G --> H{Score > Cutoff?}
    H -->|Yes| I[Send to Download Client]
    H -->|No| J[Wait for better release]
    
    I --> K{Monitor Enabled?}
    K -->|Yes| L[Check for upgrades weekly]
    K -->|No| M[Done]

    style D fill:#27ae60,color:#fff
    style I fill:#f39c12
    style L fill:#3498db,color:#fff
```

---

## 🔄 Monitoring & Maintenance

```mermaid
graph TB
    subgraph "Automatic Tasks"
        A1[Lidarr RSS Sync<br/>Every 15 min]
        A2[Navidrome Music Scan<br/>Every 1 hour]
        A3[Prowlarr Indexer Sync<br/>Every 1 hour]
        A4[Docker Log Rotation<br/>Max 10MB x 3 files]
    end

    subgraph "Manual Checks"
        M1[Check Logs<br/>docker logs -f]
        M2[Check Disk Space<br/>df -h]
        M3[Check Container Stats<br/>docker stats]
        M4[Test Downloads<br/>Add test artist]
    end

    subgraph "Alerts Needed"
        AL1[Disk 90% Full]
        AL2[Download Failed 3x]
        AL3[Container Down]
        AL4[High RAM Usage]
    end

    A1 -.-> M1
    A2 -.-> M1
    M2 --> AL1
    M4 --> AL2
    M3 --> AL3
    M3 --> AL4

    style A1 fill:#3498db,color:#fff
    style A2 fill:#3498db,color:#fff
    style AL1 fill:#e74c3c,color:#fff
    style AL2 fill:#e74c3c,color:#fff
    style AL3 fill:#e74c3c,color:#fff
    style AL4 fill:#e74c3c,color:#fff
```

---

## 📱 Client Interaction

```mermaid
graph TB
    subgraph "Symfonium Features"
        S1[Library Browser]
        S2[Search]
        S3[Playlists]
        S4[Radio/Mixes]
        S5[Offline Cache]
        S6[Scrobbling]
    end

    subgraph "Navidrome Subsonic API"
        N[Navidrome Server]
    end

    subgraph "Data Sources"
        D1[Music Files<br/>/music/]
        D2[Database<br/>navidrome.db]
        D3[Last.fm API]
    end

    S1 <-->|getMusicFolders<br/>getIndexes| N
    S2 <-->|search3| N
    S3 <-->|getPlaylists<br/>createPlaylist| N
    S4 <-->|getSimilarSongs<br/>getRadioStation| N
    S5 <-->|stream<br/>download| N
    S6 <-->|scrobble| N

    N <--> D1
    N <--> D2
    N --> D3

    style N fill:#feca57
    style S5 fill:#27ae60,color:#fff
    style S6 fill:#e84393,color:#fff
```

---

## 🎓 Legend

| Color | Meaning |
|-------|---------|
| 🔴 Red | Lidarr (Music Management) |
| 🔵 Blue | qBittorrent (Torrent Client) |
| 🟢 Green | Slskd (Soulseek P2P) |
| 🟡 Yellow | Navidrome (Streaming Server) |
| 🟣 Purple | Prowlarr (Indexer Manager) |
| 🔶 Orange | NPM (Reverse Proxy) |
| ⚪ Gray | Storage (SSD/SATA) |

---

## 📐 Hardware Requirements (Current Setup)

```
Server: root@167.253.157.7

CPU:     4 cores (sufficient)
RAM:     6 GB total
         - Used: 2.9 GB
         - Available: 2.8 GB
         - Swap: 18 GB (3.3 GB used)

Storage:
  - SSD (system):     ~100 GB (configs & databases)
  - SATA (data):      2 TB (47 GB used, 1.9 TB free)

Network:
  - Bandwidth: Unlimited (assuming VPS/Dedicated)
  - Ports: 80, 443, 6881, 50300 open
```

---

## 🚦 Current Status

```mermaid
graph LR
    A[Infrastructure] -->|100%| OK1[✅]
    B[Containers] -->|100%| OK2[✅]
    C[Configuration] -->|10%| WARN1[⚠️]
    D[Integrations] -->|0%| WARN2[⚠️]
    E[Content] -->|0%| WARN3[⚠️]
    F[Automation] -->|0%| WARN4[⚠️]

    style OK1 fill:#27ae60,color:#fff
    style OK2 fill:#27ae60,color:#fff
    style WARN1 fill:#f39c12,color:#fff
    style WARN2 fill:#e74c3c,color:#fff
    style WARN3 fill:#e74c3c,color:#fff
    style WARN4 fill:#e74c3c,color:#fff
```

**Overall: 18% Ready → 20 minutes of configuration needed**

---

## 🔗 References

- Docker Compose: `/root/Homeserver/docker-compose.yml`
- Environment: `/root/Homeserver/.env`
- Documentation:
  - `MUSIC_STACK_AUDIT.md` (Full audit)
  - `MUSIC_QUICK_SETUP.md` (Setup guide)
  - `MUSIC_SLSKD_ADDON.md` (Soulseek extension)
  - `MUSIC_STACK_SUMMARY.md` (Executive summary)

---

**Created:** November 29, 2025  
**Version:** 1.0  
**Purpose:** Visual reference for music stack architecture

