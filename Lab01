`# Лаб 01 — Тайлан: Хоёр давхаргат платформ ба ирмэг–үүлний гүүр

**Баг:** Team06   **Огноо:** 2026-10-22  
**Гишүүд ба хувь нэмэр:**

| Гишүүн | Хийсэн ажил |
|---|---|
| B231950001 М.Нэргүй | Edge орчны тохиргоо, Edge agent ажиллуулах, систем мониторинг ба хоцролтын хэмжилт |
| B231950003 А.Тэлмэн | Mosquitto bridge тохиргоо, Cloud stack болон ThingsBoard сервисуудыг эхлүүлэх |
| B231950027 Г.Энхбилгүүн | Ачааллын туршилт (Load testing 50, 100, 200 холболт) болон Grafana dashboard хяналт |

**Git commit:** `a463305`   **Tag:** `lab01-done`

---

## 0. Хэмжилтийн орчин

Тоон үр дүн зөвхөн орчноо мэдэж байж давтагдана. Хэмжилт бүрийн өмнө бөглөнө.

| Үзүүлэлт | 🥧 Raspberry Pi 3B | 💻 Зөөврийн компьютер |
|---|---|---|
| **Загвар / процессор** | Raspberry Pi 3 Model B Plus Rev 1.4 / 4 цөм (Cortex-A53 @ 1.2 GHz, aarch64) | DESKTOP-109Q6M3 / 12 ширхэг цөм (x86_64) |
| **RAM (нийт / боломжтой)** | 905 MiB нийт / 529 MiB боломжтой | 7942 MiB нийт / 6383 MiB боломжтой (Docker-т 7.8 GB) |
| **OS, цөмийн хувилбар** | Linux 6.x (aarch64) | Ubuntu (WSL2 / x86_64) |
| **Сүлжээ (кабель / Wi-Fi, хурд)** | 100 Mb/s Ethernet (IP: `172.16.158.166`) | LAN / Wi-Fi сүлжээ (IP: `172.16.157.157` / `172.16.157.205`) |
| **Хадгалалт (microSD ангилал)** | 29G нийт, 23G сул (20% дүүрсэн) | 1007G нийт, 954G сул |
| **Docker хувилбар** | Docker version 29.9.1, build dfc520d | Docker version 29.8.1, build 4a63305 |

**Хэмжилтийн өмнөх шалгалт (🥧):**

| Шалгалт | Утга | Хэвийн үү |
|---|---|---|
| `vcgencmd get_throttled` | `throttled=0x0` | `0x0` байх ЁСТОЙ (Тийм) |
| `vcgencmd measure_temp` | `52.6 °C` – `58.5 °C` | **< 70 °C** (Тийм) |
| `free -m` → available | `498 MiB` – `540 MiB` | **> 400 MiB** (Тийм) |
| `timedatectl` synchronized | `NTP OK` | **yes** (Тийм) |
| VS Code сервер таслагдсан уу | `pkill -f vscode-server` | **Тийм** |

---

## 1. Гүйцэтгэсэн ажил

Төслийн бэлтгэл ба ажиллагааг алхам бүрээр тохируулан туршив.

### 1.1 Edge (Raspberry Pi 3B) орчны шалгалт ба Bridge тохиргоо
```bash
$ cd ~/cnc302/edge && make check
docker       OK
mosquitto    OK
bridge.conf  OK
swap         OK (2047 MB)
cgroup mem   OK
NTP          OK
үлдэгдэл     510 MB available

$ uname -m
aarch64

$ vcgencmd get_throttled
throttled=0x0

$ grep -E '^(CLOUD_HOST|DEVICE_ID)=' .env
CLOUD_HOST=172.16.157.157
DEVICE_ID=nergui-telmen-enhbilgvvn

$ make bridge
✓ mosquitto/conf.d/bridge.conf үүсчээ -> 172.16.157.205:1883

$ docker compose restart mosquitto
$ make link
гуурний төлөв (Ctrl+C зогсооно):
cnc302/shutis/mhts/lab/team06nte/bridge/state 1
```

### 1.2 Cloud Stack болон ThingsBoard эхлүүлэлт
```bash
$ cd ~/cnc302/stack && make health
EMQX      200
InfluxDB  200
Grafana   200
Registry  200

$ docker info --format '{{.MemTotal}}' | awk '{printf "Docker-т %.1f GB\n", $1/1024/1024/1024}'
Docker-т 7.8 GB

$ docker compose -f docker-compose.yml -f docker-compose.tb.yml --profile core --profile tb up -d
$ sleep 180
$ docker stats --no-stream cnc302-thingsboard
CONTAINER ID   NAME                 CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O
423440febf95   cnc302-thingsboard   12.26%    1.179GiB / 2.441GiB   48.30%    7.2MB/220kB   7.51MB/305MB
```

### 1.3 Сүлжээний хоцролт (Latency) ба ачааллын хэмжилт
```bash
# Loopback хоцролт
$PY tools/qos_latency.py --host localhost --port 1883 \
    --path loopback --qos 1 --count 300 --interval 0.05 \
    --csv lab01/out/04-latency-loopback.csv

# Bridge хоцролт
$PY tools/qos_latency.py --host localhost --port 1883 \
    --sub-host $CLOUD_HOST --sub-port 1883 \
    --topic cnc302/shutis/mhts/lab/$DEVICE_ID/bench \
    --path bridge --qos 1 --count 300 --interval 0.05 \
    --csv lab01/out/04-latency-bridge.csv

# LAN хоцролт
$PY tools/qos_latency.py --host $CLOUD_HOST --port 1883 \
    --path lan --qos 1 --count 300 --interval 0.05 \
    --csv lab01/out/04-latency-lan.csv
```

---

## 2. Хэмжилтийн үр дүн

Зааврын заавал бөглөх Хүснэгт 1.1 – 1.6-г доорх тоон утгуудаар бөглөв.

### Хүснэгт 1.1 — Pi 3B-гийн суурь төлөв (юу ч ажиллаагүй)

| Хэмжигдэхүүн | Утга | Хаанаас | Тэмдэглэл |
|---|---|---|---|
| **Pi загвар** | Raspberry Pi 3 Model B Plus Rev 1.4 | Модель : | 4 цөм (Cortex-A53 @ 1.2 GHz) |
| **MemTotal (MiB)** | 905 MiB | Санах ой : — "нийт" | ~925 хүлээгдэнэ |
| **MemAvailable (MiB)** | 529 MiB | Санах ой : — "боломжтой" | 529 – 540 MiB |
| **Swap ашигласан (MiB)** | 2 MiB | Swap : | 0 – 2 MiB (хэвийн) |
| **CPU температур (°C)** | 52.6 °C | Температур : | < 70 °C |
| **get_throttled** | 0x0 | Throttle : | **0x0** (Хэвийн) |
| **gpu_mem** | 16M | gpu=...M | SETUP Б.3 хийсэн |
| **microSD чөлөөтэй (GB)** | 23G | microSD : — "сул" | 29G нийт, 20% дүүрсэн |
| **Ачаалал (1 мин)** | 0.03 | Ачаалал : — эхний тоо | (0.03 0.11 0.16) |

### Хүснэгт 1.2 — Үүлний үйлчилгээний эхлэх хугацаа

| Үйлчилгээ | Эхэлсэн (сек) | Эрүүл болсон (сек) | RAM (MiB) | CPU % |
|---|---|---|---|---|
| **emqx** | 0.7s | Healthy | 227.2 MiB | 1.65% |
| **influxdb** | 0.7s | Healthy | 28.65 MiB | 0.21% |
| **grafana** | 0.0s | Running | 67.93 MiB | 0.23% |
| **registry** | 0.0s | Running | 50.27 MiB | 0.20% |
| **Нийт** | **1.4s** | **Healthy** | **374.05 MiB** | **2.29%** |

### Хүснэгт 1.3 — Ирмэгийн үйлчилгээний нөөц

| Хэсэг | RAM (MiB) | CPU % | Хаанаас |
|---|---|---|---|
| **mosquitto** | 2.71 MiB | 0.07% | `03-edge-running.txt` (docker stats) |
| **ирмэгийн агент** | 4.24 MiB | 0.08% | `ps ... edge_agent` (RSS / 1024) |
| **Docker демон** | 87.5 MiB | 0.50% | `ps ... dockerd` (RSS / 1024) |
| **Нийт нэмэгдсэн** | 6.95 MiB | 0.15% | Хүснэгт 1.1-ийн MemAvailable − одоогийн MemAvailable |
| **Үлдэгдэл MemAvailable** | 538 MiB | — | `03-edge-running.txt` (боломжтой санах ой) |

### Хүснэгт 1.4 — Гурван замын саатал (QoS 1, 300 мессеж)

| Зам | p50 (мс) | p95 (мс) | **p99 (мс)** | max (мс) | Алдагдал % |
|---|---|---|---|---|---|
| **loopback (Pi доторх)** | 2.544 | 3.091 | **3.725** | 5.307 | 0.0% |
| **LAN (Pi → EMQX)** | 15.970 | 520.095 | **846.397** | 981.760 | 0.0% |
| **гүүрээр (Bridge)** | 12.784 | 26.008 | **38.348** | 68.385 | 0.0% |

### Хүснэгт 1.5 — Ирмэгийн ачааллын хариу үйлдэл

| Виртуал төхөөрөмж | mosquitto RAM (MiB) | MemAvailable (MiB) | Swap si/so | Темп (°C) | get_throttled |
|---|---|---|---|---|---|
| **0 (суурь)** | 4.24 MiB | 538 MiB | 0/0 | 55.8 °C | 0x0 |
| **50** | 4.50 MiB | 525 MiB | 0/0 | 56.9 °C | 0x0 |
| **100** | 10.20 MiB | 519 MiB | 0/0 | 58.0 °C | 0x0 |
| **200** | 44.93 MiB | 498 MiB | 0/0 | 56.9 °C | 0x0 |

> **Тэмдэглэл:** Ачааллын явцад `MemAvailable` 498 MiB-аас доош унаагүй тул 150 MiB аюултай хязгаарт хүрээгүй, Pi 3B тогтвортой ажилласан.

### Хүснэгт 1.6 — Хэрэв бүгдийг Pi дээр ажиллуулах гэвэл

| Үйлчилгээ | Компьютер дээрх бодит RAM (MiB) | Pi-д багтах уу |
|---|---|---|
| **emqx** | 227.2 MiB | Хүнд |
| **influxdb** | 28.65 MiB | Тийм |
| **grafana** | 67.93 MiB | Тийм |
| **registry** | 50.27 MiB | Тийм |
| **thingsboard** *(сонголтот)* | 1179.0 MiB (1.18 GiB) | **Үгүй** |
| **Нийт** | **1553.05 MiB** | **Багтахгүй** |
| **Pi-д боломжтой (Хүснэгт 1.1)** | **529 MiB** | — |
| **Дутагдал** | **~1024 MiB (1 GB гаруй)** | — |

---

## 3. Дүгнэлт

1. **Хоцролтын ялгаа ба Bridge-ийн ач холбогдол:**  
   Loopback замд p50 хоцролт ердөө **2.54 мс** байгаа бөгөөд Bridge горимд (Edge Mosquitto -> Cloud EMQX) p50 хоцролт **12.78 мс** байна. Шууд LAN холболтын үед p99 хоцролт **846.4 мс** хүртэл өсөж байгаа нь сүлжээний сүүлийн саатал (tail latency)-ийг харуулж байна. Mosquitto bridge нь өгөгдлийг ирмэг дээр локалаар буферлэн дамжуулдаг тул сүлжээний саатлаас хамаарах эрсдэлийг бууруулдаг.

2. **Ачаалал ба Ресурсын ашиглалт:**  
   50, 100, 200 төхөөрөмжийн зэрэгцээ ачааллын туршилтын үед нийт 45,688 мессежийг **198.7 мсж/с** хурдтайгаар 0% алдагдалтай дамжуулсан. Энэ үед Pi 3B дээрх Mosquitto сервис ердөө **44.93 MiB RAM** болон **3.32% CPU** ашиглаж, санах ойн үлдэгдэл **498 MiB**-аас буураагүй.

3. **Архитектурын зааглалт:**  
   Платформын хүнд сервисууд болох ThingsBoard (1.18 GB RAM) болон EMQX (227.2 MiB RAM)-ийг Pi 3B (боломжит 529 MiB RAM) дээр ажиллуулах боломжгүй болох нь **~1024 MiB санах ойн дутагдлаар** тоон утгаар нотлогдлоо.

---

## 4. Хяналтын асуултын хариулт

1. **Хүснэгт 1.4-д гурван замын p99 хэд дахин ялгаатай вэ? Ялгааны гол шалтгаан юу вэ — сүлжээ, брокерын боловсруулалт, эсвэл дараалал? Хэрхэн ялгаж тогтоох вэ?**  
   - Loopback p99 = **3.725 мс**, Bridge p99 = **38.348 мс** (loopback-аас ~10.3 дахин их), LAN p99 = **846.397 мс** (loopback-аас ~227.2 дахин их).
   - **Гол шалтгаан:** Сүлжээний орчны саатал ба сүлжээний хэлбэлзэл (jitter). Loopback нь IPC/kernel socket ашигладаг тул сүлжээний физик орчин шаардлагагүй. LAN зам нь Wi-Fi/Ethernet ачаалал болон пакетын дарааллаас шалтгаалан саатал үүсгэдэг.
   - **Ялгах арга:** ICMP ping RTT болон локал socket latency-г тусад нь хэмжиж, брокерын процессор дээрх боловсруулах хугацаанаас идэвхтэй зааглаж тогтооно.

2. **Хүснэгт 1.5-д хязгаарлагч нөөц юу байв — RAM, CPU, эсвэл сүлжээ? Хариултаа тоогоор нотол.**  
   - 200 холболттой туршилтын үед CPU ашиглалт **3.32%**, Mosquitto RAM **44.93 MiB**, санах ойн үлдэгдэл **498 MiB** (Swap `si=0, so=0`), `throttled=0x0` байв.
   - Иймд RAM болон CPU ханах дээрээ тулаагүй. Хязгаарлагч нөөц нь **100 Mb/s Ethernet сүлжээний зурвасын хурд ба сүлжээний дараалал** байв.

3. **Хүснэгт 1.6-гийн дутагдал хэдэн MiB вэ? Хэрэв Pi 3B-д 2 GB RAM байсан бол платформыг түүн дээр ажиллуулах уу? Яагаад тийм / үгүй?**  
   - Санах ойн дутагдал **~1024 MiB (1 GB гаруй)** (ThingsBoard-тай нийт хэрэгцээ 1553 MiB, Pi-д боломжит 529 MiB).
   - **Ажиллуулахгүй.** Хэдийгээр 2 GB RAM-тай байсан ч Pi 3B-ийн процессор (Cortex-A53 @ 1.2 GHz) болон microSD дискний I/O хурд нь InfluxDB3-ийн дискний бичилт ба ThingsBoard-ийн JVM санах ойн цэвэрлэгээ (GC pause)-г даахгүй тул систем гацна.

4. **cleansession false -ийг true болговол юу өөрчлөгдөх вэ? Ямар тохиолдолд true нь зөв сонголт байх вэ?**  
   - `cleansession false` нь холболт тасрах үед MQTT сесс болон хүргэгдээгүй мессежүүдийг дискийн буферт хадгалж, эргэж холбогдоход нөхөн дамжуулдаг.
   - `true` болгобол тасалдлын үеийн бүх мессеж арилна. Мессеж гээгдэх нь чухал биш, зөвхөн одоогийн хамгийн сүүлийн үеийн төлөвийг мэдээлэх шаардлагатай тохиолдолд `cleansession=true` нь зөв сонголт болно.

5. **Гүүр нь `cnc302/shutis/#` сэдвийг дамжуулдаг. Хэрэв өөр баг `SITE=shutis` ашиглаад нэг брокерт холбогдвол юу болох вэ? Үүнээс хэрхэн сэргийлэх вэ (хоёр өөр арга нэрлэ)?**  
   - **Асуудал:** Сэдвийн давхцал (topic collision) үүсэж, хоёр багийн өгөгдөл холилдон буруу аналитик гарна.
   - **Арга 1:** Topic prefix/namespace-д багийн ID-г оруулах (жишээ нь: `cnc302/shutis/team06/#`).
   - **Арга 2:** MQTT хэрэглэгчийн нэвтрэх эрх ба ACL (Access Control List) тохируулж, баг бүрт зөвхөн өөрийн сэдэвт бичих/унших эрх олгох.

6. **Pi дээр `vcgencmd get_throttled` нь `0x50000` буцаав. Энэ юу гэсэн үг вэ, өмнөх хэмжилтүүд хүчинтэй юу?**  
   - `0x50000` нь "Under-voltage has occurred" (тэжээлийн хүчдэл унаж байсан) гэсэн түүхэн флангийг илэрхийлнэ.
   - Хэрэв хэмжилт хийж байх мөчид одоогийн төлөв идэвхтэй байсан бол CPU-ийн давтамж буурсан байх тул тухайн хэмжилт **хүчингүй**. Тэжээлийн блок болон кабелийг сольж хөргөсний дараа хэмжилтийг дахин хийнэ.

---

## 5. Тулгарсан асуудал ба шийдэл

| Асуудал | Шалтгаан | Шийдсэн арга | Хугацаа |
|---|---|---|---|
| Mosquitto гүүрний холболт тасрах (`Host is unreachable`) | `.env` файлд `CLOUD_HOST` IP хаяг буруу тохируулагдсан | `.env` файлд `CLOUD_HOST=172.16.157.157` / `172.16.157.205` IP-г зааж `make bridge`, `docker compose restart mosquitto` хийсэн | 15 мин |
| Apt install зөвшөөрлийн алдаа (`Authentication failed`) | Sudo нууц үг буруу сэргээгдсэн | Sudo нууц үгийг залруулан оруулан `bc`, `jq` багцуудыг суулгасан | 5 мин |
| ThingsBoard санах ойн ачаалал | ThingsBoard ба Postgres эхлэхэд RAM их шаардсан | Docker орчинд 7.8 GB санах ой хуваарилж `sleep 180` хүлээж бэлэн болгосон | 10 мин |

---

## 6. Ашигласан эх сурвалж

- CNC302 IoT Platform лабораторийн заавар болон `tools/qos_latency.py`, `tools/measure_stack.sh` скриптүүд
- Docker, EMQX Dashboard, InfluxDB3 ба Grafana тохиргооны баримт бичгүүд
- Raspberry Pi `vcgencmd` болон Linux систем мониторингийн зааварчилгаа

---

## 7. Хэмжилтийн файлууд

`lab01/out/` болон `lab01/` доторх хэмжилтийн файлууд:

- `lab01/dashboard.json`
- `lab01/out/01-edge-baseline.txt`
- `lab01/out/02-cloud-snapshot.txt`
- `lab01/out/03-edge-running.txt`
- `lab01/out/04-latency-loopback.csv`
- `lab01/out/04-latency-bridge.csv`
- `lab01/out/04-latency-lan.csv`
- `lab01/out/05-load-000.txt`
- `lab01/out/05-monitor.txt`
- `lab01/out/06-cloud-mem.txt`
- `lab01/out/stack-edge-20261022-111140.csv`
- `lab01/out/stack-edge-20261022-115248.csv`
- `lab01/out/stack-edge-20261022-121100.csv`
- `lab01/out/stack-cloud-20261022-104415.csv`
