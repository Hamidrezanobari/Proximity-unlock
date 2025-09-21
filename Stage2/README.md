
هدف کلی این مرحله

تا آخر این مرحله باید بتوانی یک کامپوننت BLE Scanner بسازی که:

تبلیغات (advertisements) را پیوسته می‌خواند و از آن EID / Manufacturer data را استخراج می‌کند،

برای هر دیده شدن، addr + RSSI + timestamp را به یک queue می‌فرستد،

بتوانی روی همان برد به یک گوشی (که به‌عنوان Peripheral/Server عمل می‌کند) GATT Client وصل شوی، CHAL ارسال کنی و RESP دریافت و بررسی کنی (timeout قابل تنظیم، هدف اولیه ≤ 500 ms برای چرخه CHAL→RESP).

یعنی در عمل: BLE Scan → parse AD → queue sighting → (گزینش/اتخاذ تصمیم) → connect GATT → challenge/response.

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
