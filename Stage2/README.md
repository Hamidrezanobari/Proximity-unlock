
هدف کلی این مرحله

تا آخر این مرحله باید بتوانی یک کامپوننت BLE Scanner بسازی که:

تبلیغات (advertisements) را پیوسته می‌خواند و از آن EID / Manufacturer data را استخراج می‌کند،

برای هر دیده شدن، addr + RSSI + timestamp را به یک queue می‌فرستد،

بتوانی روی همان برد به یک گوشی (که به‌عنوان Peripheral/Server عمل می‌کند) GATT Client وصل شوی، CHAL ارسال کنی و RESP دریافت و بررسی کنی (timeout قابل تنظیم، هدف اولیه ≤ 500 ms برای چرخه CHAL→RESP).

یعنی در عمل: BLE Scan → parse AD → queue sighting → (گزینش/اتخاذ تصمیم) → connect GATT → challenge/response.

چیزهایی که مو به مو باید یاد بگیری (فهرست دقیق + هدف هرکدام)

مبانی BLE (نظری کوتاه، اما ضروری)

چی یاد بگیری: فرق GAP و GATT، Advertising vs. Scanning، Peripheral vs. Central، AD fields (فرمت [len][type][data])، AD type 0xFF (Manufacturer Specific).

هدف: بفهمی کجا دنبال EID باشی و چرا RSSI در advertising هست.

تمرین: با nRF Connect روی گوشی ببین چگونه advertising ساخته می‌شود (Manufacturer data را وارد کن).

RSSI — مفهوم و رفتار عملی

چی یاد بگیری: RSSI مقدار به دسی‌بل (dBm) است؛ نویز، فاصله، جهت و مانع روی آن تاثیر می‌گذارد؛ نوسان دارد → باید صاف‌سازی (EMA) و hysteresis داشته باشی.

هدف: بدان چگونه مقدار RSSI را جمع‌آوری، smooth و برای «نزدیکی» تصمیم‌گیری کنی.

تمرین: با یک beacon گوشی را از نزدیک به دور منتقل کن و در لاگ ESP32 تغییر RSSI را ثبت کن.

NimBLE در ESP-IDF — راه‌اندازی و APIهای پایه

چی یاد بگیری: چگونه host+controller را init کنی، چگونه اسکن را با ble_gap_disc() شروع کنی و callback رو (ble_gap_event) ثبت کنی.

هدف: بتوانی اسکن را start/stop کنی و event‌های BLE_GAP_EVENT_DISC را دریافت کنی.

تمرین: از مثال‌های رسمی (blecent, bleprph, NimBLE_GATT_Server) کپی‌پیست کن و روی برد اجرا کن.

پارسر Advertisement (AD fields)

چی یاد بگیری: الگوریتم خواندن [len][type][data] و استخراج type==0xFF و گرفتن 16 بایت اول (EID).

هدف: ساختن ble_sighting_t با eid[16], rssi, addr[6], timestamp.

تمرین: در callback اسکن، AD را پارس کن و اگر Manufacturer data با حداقل 16 بایت بود، یک struct بساز و به queue بفرست.

FreeRTOS integration (Task + Queue)

چی یاد بگیری: ایجاد queue برای sightingها، یک task برای اسکن و یک consumer task برای پردازش/چاپ.

هدف: جداسازی وظایف — اسکن real-time و پردازش در task دیگر.

تمرین: بساز xQueueCreate(32, sizeof(ble_sighting_t))، xQueueSend() در callback و در task دیگر xQueueReceive().

GATT Client ↔ GATT Server عملی

چی یاد بگیری: مراحل کلی (connect → discover service → discover characteristics → subscribe notifications → write/read) و APIهای مربوطه (ble_gattc_open, discovery calls, ble_gattc_write_flat, subscription).

هدف: بتوانی از ESP32 به گوشی (که با nRF Connect به‌صورت Peripheral/Server ساخته‌ای) وصل شوی، داده بنویسی و notification بگیری.

تمرین: با دو characteristic ساده (CHAL + RESP) کار کن: ESP به‌عنوان client CHAL می‌فرستد و برای RESP subscribe می‌کند تا notification دریافت کند.

چالش ساده (Challenge) — طراحی و پیاده‌سازی

چی یاد بگیری: پروتکل چالش ساده (نحوه ساخت CHAL، چه چیزی داخلش باشد: nonce, door_id, ts) و نحوه اعتبارسنجی پاسخ (مثلاً پاسخ == hash(nonce+door_id) یا در حد اولیه nonce+1).

هدف: پیاده‌سازی چرخه CHAL→RESP با timeout و log latency.

تمرین: اجرای دستی با nRF Connect (در ابتدا) و سپس خودکار کردن پاسخ در یک اپ ساده روی گوشی یا با ابزار دسکتاپ.

اندازه‌گیری و دیباگ (latency, logs, reconnect)

چی یاد بگیری: استفاده از esp_timer_get_time() برای timestamp، لاگ‌گذاری دقیق و شمارنده‌ها (scan events, connections, challenges, success/fail).

هدف: بتوانی زمان CHAL→RESP و کل چرخه را اندازه بگیری و معیار ≤500 ms را بررسی کنی.

تمرین: لاگ بگیر و اختلاف timestamp‌ها را محاسبه کن؛ اگر زیاد است، پروفایل کن (کجا تاخیر می‌آید؟ اتصال؟ discovery؟).

1. BLE Fundamentals (Short Theory, but Essential)

What to learn:

Difference between GAP vs. GATT

Advertising vs. Scanning

Peripheral vs. Central

AD fields format: [len][type][data]

AD type 0xFF (Manufacturer Specific Data)

Goal:

Understand where to look for EID (Ephemeral Identifier) in advertising.

Know why RSSI is available in advertising packets.

Exercise:

Use nRF Connect app on your phone → check how advertising packets are structured.

Inspect Manufacturer Data fields.

2. RSSI — Concept and Practical Behavior

What to learn:

RSSI is measured in dBm.

Influenced by noise, distance, orientation, and obstacles.

RSSI values fluctuate → you need smoothing (e.g., EMA filter) and hysteresis.

Goal:

Learn how to collect, smooth, and interpret RSSI to decide device “proximity.”

Exercise:

Use a BLE beacon or smartphone.

Move it near/far and log RSSI values on ESP32.

Observe fluctuations and apply smoothing.

3. NimBLE in ESP-IDF — Initialization & Basic APIs

What to learn:

How to initialize host + controller.

Start scanning with ble_gap_disc().

Register a callback (ble_gap_event).

Goal:

Be able to start/stop scanning and handle BLE_GAP_EVENT_DISC events.

Exercise:

Run official examples (blecent, bleprph, NimBLE_GATT_Server).

Flash them to ESP32 and verify logs.

4. Advertisement Parser (AD Fields)

What to learn:

Algorithm for parsing [len][type][data].

Extract type == 0xFF (Manufacturer Data).

Take first 16 bytes → treat as EID.

Goal:

Build a ble_sighting_t struct with fields:

struct ble_sighting_t {
    uint8_t eid[16];
    int rssi;
    uint8_t addr[6];
    uint64_t timestamp;
};


Exercise:

In the scan callback:

Parse AD.

If Manufacturer Data ≥16 bytes, build ble_sighting_t and send to queue.

5. FreeRTOS Integration (Task + Queue)

What to learn:

Create a queue for sightings.

Create a task for scanning and another for processing/printing.

Goal:

Decouple real-time scanning from processing logic.

Exercise:

Use:

xQueueCreate(32, sizeof(ble_sighting_t));
xQueueSend();
xQueueReceive();


One task sends sightings, another task consumes them.

6. GATT Client ↔ GATT Server (Hands-on)

What to learn:

Typical flow:
connect → discover service → discover characteristics → subscribe to notifications → write/read.

Relevant APIs:

ble_gattc_open()

Discovery calls

ble_gattc_write_flat()

Subscription functions

Goal:

Connect ESP32 to a phone (running nRF Connect in Peripheral mode).

Be able to write data and subscribe to notifications.

Exercise:

Define two characteristics: CHAL + RESP.

ESP32 (Client) writes CHAL → waits for RESP via notification.

7. Simple Challenge (CHAL→RESP Protocol)

What to learn:

Protocol design for a basic challenge:

CHAL contains: nonce, door_id, timestamp.

RESP is a simple function of CHAL (e.g., nonce+1 or hash).

Goal:

Implement a challenge-response cycle with timeout + logging latency.

Exercise:

First: do it manually with nRF Connect.

Later: automate response with a small app or desktop script.

8. Measurement & Debugging (Latency, Logs, Reconnects)

What to learn:

Use esp_timer_get_time() for timestamps.

Log detailed counters: scan events, connections, challenges, success/fail.

Goal:

Measure end-to-end CHAL→RESP latency.

Target ≤ 500 ms.

Profile where delays come from (scan, connect, discovery, write).

Exercise:

Add timestamps to logs.

Compare times → analyze performance.

✅ By finishing all these, you’ll have real practical BLE skills: scanning, parsing, proximity detection, GATT communication, challenge-response security, and debugging.
