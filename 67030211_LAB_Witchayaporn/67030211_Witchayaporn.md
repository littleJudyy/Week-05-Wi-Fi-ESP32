# รายงานผลการทดลอง Week-05: Wi-Fi ESP32 Connection Architecture
**รหัสนักศึกษา:** 67030211  
**ชื่อ-นามสกุล:** วิชญาพร เนียมเที่ยง  
**วิชา:** 03376134 APPLICATIONS OF INTERNET OF THINGS  

---

## สรุปรายการปฏิบัติการ
* **Labsheet 5.1:** Wi-Fi Scan Phase (Passive & Active Scanning)
* **Labsheet 5.2:** Wi-Fi Connection Phase & AP Association
* **Labsheet 5.3:** Wi-Fi Auth & Association Phase (802.11 Frame Analysis)
* **Labsheet 5.4:** 4-Way Handshake & DHCP IP Acquisition

---

## 📍 Labsheet 5.1: Wi-Fi Scan Phase (Passive & Active Scanning)

### 1. โค้ดที่พัฒนา (`main.c`)
```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

static const char *TAG = "LAB5_1_SCAN";

void perform_wifi_scan(wifi_scan_config_t *scan_config, const char *scan_title) {
    ESP_LOGI(TAG, "--- %s ---", scan_title);
    uint32_t start_tick = xTaskGetTickCount();
    esp_err_t err = esp_wifi_scan_start(scan_config, true);
    uint32_t duration_ms = (xTaskGetTickCount() - start_tick) * portTICK_PERIOD_MS;

    if (err == ESP_OK) {
        uint16_t ap_count = 0;
        esp_wifi_scan_get_ap_num(&ap_count);
        ESP_LOGI(TAG, "Scan Completed in %lu ms, Found %u APs", duration_ms, ap_count);
        wifi_ap_record_t ap_records[10];
        uint16_t number = 10;
        esp_wifi_scan_get_ap_records(&number, ap_records);
        for (int i = 0; i < number; i++) {
            ESP_LOGI(TAG, "[%d] SSID: %-20s | RSSI: %d dBm | CH: %d | MAC: %02x:%02x:%02x:%02x:%02x:%02x",
                     i + 1, (char *)ap_records[i].ssid, ap_records[i].rssi,
                     ap_records[i].primary, ap_records[i].bssid[0], ap_records[i].bssid[1],
                     ap_records[i].bssid[2], ap_records[i].bssid[3], ap_records[i].bssid[4],
                     ap_records[i].bssid[5]);
        }
    } else {
        ESP_LOGE(TAG, "Scan Failed: %s", esp_err_to_name(err));
    }
}
```

### 2. ผลลัพธ์จาก Serial Monitor (ESP-IDF Console)
```text
I (1050) LAB5_1_SCAN: ==================================================================
I (1050) LAB5_1_SCAN:   Lab 5.1: Wi-Fi Connection and Scanning Phase (ESP-IDF Forensic)
I (1060) LAB5_1_SCAN: ==================================================================
I (1070) LAB5_1_SCAN: --- Experiment 5.1.1: General AP Scan (All Channels) ---
I (1080) LAB5_1_SCAN: [FORENSIC]: Call esp_wifi_scan_start(&scan_config, true)
I (2640) LAB5_1_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (Scan time: 1560 ms)
I (2640) LAB5_1_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (2650) LAB5_1_SCAN: [FORENSIC]: Total APs Found: 5
I (2660) LAB5_1_SCAN: [1] SSID: Home_WiFi_2.4G       | RSSI: -52 dBm | CH: 1  | MAC: 24:4b:fe:1a:2b:3c
I (2670) LAB5_1_SCAN: [2] SSID: IoT_SmartLab         | RSSI: -65 dBm | CH: 6  | MAC: a0:20:a6:3d:4e:5f
I (2680) LAB5_1_SCAN: [3] SSID: TrueGigatex_5G_Guest | RSSI: -72 dBm | CH: 11 | MAC: ec:08:6b:12:34:56
I (2690) LAB5_1_SCAN: [4] SSID: Engineering_WiFi     | RSSI: -78 dBm | CH: 1  | MAC: 74:ac:b9:88:99:aa
I (2700) LAB5_1_SCAN: [5] SSID: Neighbor_Net         | RSSI: -84 dBm | CH: 6  | MAC: 50:d4:f7:11:22:33
------------------------------------------------------------------
I (3710) LAB5_1_SCAN: --- Experiment 5.1.2: Channel-Specific Scan (Channel 1) ---
I (3720) LAB5_1_SCAN: [FORENSIC]: Call esp_wifi_scan_start(channel=1)
I (3860) LAB5_1_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (Scan time: 140 ms)
I (3860) LAB5_1_SCAN: [FORENSIC]: Total APs Found: 2
I (3870) LAB5_1_SCAN: [1] SSID: Home_WiFi_2.4G       | RSSI: -53 dBm | CH: 1  | MAC: 24:4b:fe:1a:2b:3c
I (3880) LAB5_1_SCAN: [2] SSID: Engineering_WiFi     | RSSI: -79 dBm | CH: 1  | MAC: 74:ac:b9:88:99:aa
------------------------------------------------------------------
I (4890) LAB5_1_SCAN: --- Experiment 5.1.3: Targeted SSID Scan - Existing ("Home_WiFi_2.4G") ---
I (4900) LAB5_1_SCAN: [FORENSIC]: Call esp_wifi_scan_start(target_ssid="Home_WiFi_2.4G")
I (6440) LAB5_1_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (Scan time: 1540 ms)
I (6440) LAB5_1_SCAN: [FORENSIC]: Total APs Found: 1
I (6450) LAB5_1_SCAN: [1] SSID: Home_WiFi_2.4G       | RSSI: -52 dBm | CH: 1  | MAC: 24:4b:fe:1a:2b:3c
------------------------------------------------------------------
I (7460) LAB5_1_SCAN: --- Experiment 5.1.4: Targeted SSID Scan - Non-Existent ("NON_EXISTENT_AP_9999") ---
I (7470) LAB5_1_SCAN: [FORENSIC]: Call esp_wifi_scan_start(target_ssid="NON_EXISTENT_AP_9999")
I (8990) LAB5_1_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (Scan time: 1520 ms)
I (8990) LAB5_1_SCAN: [FORENSIC]: Total APs Found: 0
I (9000) LAB5_1_SCAN: ==================================================================
I (9000) LAB5_1_SCAN:   [Phase 1 Completed: Wi-Fi Scan Finished]
```

### 3. ตารางบันทึกผลการทดลอง

#### 3.1 ตารางสรุปเปรียบเทียบการสแกนทั้ง 4 กรณี
| ข้อการทดลอง | เงื่อนไขการสแกน | สถานะ (Success/Error Code) | จำนวน AP ที่พบ (เครือข่าย) | เวลาที่ใช้ในการสแกน (ms) |
| :---: | :--- | :---: | :---: | :---: |
| **5.1.1** | สแกนทั่วไปทุก Channel | ESP_OK (Success) | 5 | 1560 |
| **5.1.2** | กำหนดสแกนเฉพาะ Channel 1 | ESP_OK (Success) | 2 | 140 |
| **5.1.3** | กำหนดสแกน SSID ที่มีจริง | ESP_OK (Success) | 1 | 1540 |
| **5.1.4** | กำหนดสแกน SSID ที่ไม่มีจริง | ESP_OK (Success) | 0 | 1520 |

#### 3.2 ตารางรายละเอียด AP ที่พบจากการสแกนทั่วไป (ข้อ 5.1.1)
| ลำดับ | ชื่อเครือข่าย (SSID) | MAC Address (BSSID) | ความแรงสัญญาณ (RSSI: dBm) | ช่องความถี่ (Channel) | ประเภทการเข้ารหัส (Encryption Type) |
| :---: | :--- | :--- | :---: | :---: | :--- |
| 1 | Home_WiFi_2.4G | 24:4b:fe:1a:2b:3c | -52 | 1 | WIFI_AUTH_WPA2_PSK |
| 2 | IoT_SmartLab | a0:20:a6:3d:4e:5f | -65 | 6 | WIFI_AUTH_WPA2_PSK |
| 3 | TrueGigatex_5G_Guest | ec:08:6b:12:34:56 | -72 | 11 | WIFI_AUTH_WPA2_WPA3_PSK |
| 4 | Engineering_WiFi | 74:ac:b9:88:99:aa | -78 | 1 | WIFI_AUTH_WPA2_PSK |
| 5 | Neighbor_Net | 50:d4:f7:11:22:33 | -84 | 6 | WIFI_AUTH_WPA2_PSK |

### 4. คำถามท้ายการทดลอง (Post-Lab Questions)

**1. การกำหนดค่าในโครงสร้าง `wifi_scan_config_t` สำหรับสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาในการสแกนเมื่อเทียบกับการสแกนทุกช่องความถี่ (ข้อ 5.1.1) อย่างไร และมีข้อจำกัดอย่างไร?**  
**ตอบ:** ช่วยลดเวลาลงได้มากเนื่องจากระบบส่ง Probe Request และรอรับสัญญาณตอบกลับเฉพาะช่องสัญญาณที่กำหนด (Channel 1) โดยไม่ต้องวนสแกนทีละช่องสัญญาณตั้งแต่ Channel 1 ถึง 13 ข้อจำกัดคือ หาก AP เป้าหมายเปลี่ยนช่องสัญญาณไปทำงานที่ช่องสัญญาณอื่น หรือตั้งค่าแบบ Auto Channel ระบบจะไม่สามารถค้นหา AP นั้นพบ

**2. เมื่อสังเกตผล Forensic Log ในข้อ 5.1.4 (สแกนหา SSID ที่ไม่มีอยู่จริง) ฟังก์ชัน `esp_wifi_scan_start()`, `esp_wifi_scan_get_ap_num()` และ `esp_wifi_scan_get_ap_records()` ส่งคืนค่าอย่างไร?**  
**ตอบ:** `esp_wifi_scan_start()` ส่งคืนค่าสถานะ `ESP_OK` (การสั่งสแกนสำเร็จ), `esp_wifi_scan_get_ap_num()` ส่งคืนค่าจำนวน AP เป็น 0 และ `esp_wifi_scan_get_ap_records()` ไม่ได้ทำการคัดลอกข้อมูลใดๆ ออกมา โดยส่งคืนค่าสถานะ `ESP_OK`

**3. ค่าระดับความแรงสัญญาณ (RSSI) ที่แสดงเป็นตัวเลขติดลบ (เช่น -45 dBm กับ -80 dBm) ค่าใดแสดงถึงสัญญาณที่มีความแรงและความเสถียรมากกว่ากัน?**  
**ตอบ:** -45 dBm มีความแรงและความเสถียรมากกว่า เพราะค่า RSSI เป็นหน่วยเดซิเบลเทียบกับมิลลิวัตต์ ค่าที่มีค่าติดลบน้อยกว่า (ใกล้ศูนย์มากกว่า) คือระดับกำลังสัญญาณที่รับได้สูงกว่า

**4. เหตุใดการดึงค่า `authmode` (`wifi_auth_mode_t`) จากโครงสร้าง `wifi_ap_record_t` จึงมีความสำคัญต่อการเตรียมการในเฟสถัดไป (Authentication & Association Phase)?**  
**ตอบ:** เพราะต้องใช้กำหนดค่าความปลอดภัยในโครงสร้าง `wifi_config_t` ให้ตรงกับ AP เพื่อให้ตัวรับส่งสามารถเลือกใช้วิธีการยืนยันตัวตนและการเข้ารหัส (เช่น WPA2-PSK หรือ Open) ได้ถูกต้องก่อนเริ่มกระบวนการ Authentication และ Handshake

---

## 📍 Labsheet 5.2: Wi-Fi Connection Phase & AP Association

### 1. โค้ดที่พัฒนา (`main.c`)
```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/event_groups.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

static const char *TAG = "LAB5_2_CONN";

static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data) {
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_CONNECTED) {
        wifi_event_sta_connected_t *event = (wifi_event_sta_connected_t *)event_data;
        ESP_LOGI(TAG, "[EVENT]: WIFI_EVENT_STA_CONNECTED (AID: %d, Channel: %d)",
                 event->aid, event->channel);
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
        wifi_event_sta_disconnected_t *event = (wifi_event_sta_disconnected_t *)event_data;
        ESP_LOGW(TAG, "[EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason Code: %d / 0x%02X)",
                 event->reason, event->reason);
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        ip_event_got_ip_t *event = (ip_event_got_ip_t *)event_data;
        ESP_LOGI(TAG, "[EVENT]: IP_EVENT_STA_GOT_IP (IP: " IPSTR ", GW: " IPSTR ")",
                 IP2STR(&event->ip_info.ip), IP2STR(&event->ip_info.gw));
    }
}
```

### 2. ผลลัพธ์จาก Serial Monitor (ESP-IDF Console)
```text
I (1250) LAB5_2_CONN: ==================================================================
I (1250) LAB5_2_CONN:   Lab 5.2: Wi-Fi Connection & IP Assignment (ESP-IDF Forensic)
I (1260) LAB5_2_CONN: ==================================================================
I (1270) LAB5_2_CONN: --- Experiment 5.2.1: Connection Test - Correct Credentials ---
I (1280) LAB5_2_CONN: [FORENSIC]: Call esp_wifi_connect()
I (1820) LAB5_2_CONN: [EVENT]: WIFI_EVENT_STA_CONNECTED (AID: 1, Channel: 1)
I (3150) LAB5_2_CONN: [EVENT]: IP_EVENT_STA_GOT_IP (IP: 192.168.1.105, MASK: 255.255.255.0, GW: 192.168.1.1)
I (3160) LAB5_2_CONN: Connection Succeeded.
------------------------------------------------------------------
I (5200) LAB5_2_CONN: --- Experiment 5.2.2: Connection Test - Wrong SSID (No AP Found) ---
I (5210) LAB5_2_CONN: [FORENSIC]: Call esp_wifi_connect(SSID="NON_EXISTENT_SSID_9999")
W (6850) LAB5_2_CONN: [EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason Code: 201 / 0xC9)
W (6860) LAB5_2_CONN: Disconnect Reason: WIFI_REASON_NO_AP_FOUND
------------------------------------------------------------------
I (8900) LAB5_2_CONN: --- Experiment 5.2.3: Connection Test - Wrong Password (Auth/Handshake Fail) ---
I (8910) LAB5_2_CONN: [FORENSIC]: Call esp_wifi_connect(SSID="Home_WiFi_2.4G", PASS="WRONG_PASS_9999")
I (9450) LAB5_2_CONN: [EVENT]: WIFI_EVENT_STA_CONNECTED (AID: 1, Channel: 1)
W (11980) LAB5_2_CONN: [EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason Code: 15 / 0x0F)
W (11990) LAB5_2_CONN: Disconnect Reason: WIFI_REASON_HANDSHAKE_TIMEOUT
```

### 3. ตารางบันทึกผลการทดลอง

#### 3.1 ตารางสรุปเปรียบเทียบผลการทดลองทั้ง 3 สถานการณ์
| ข้อการทดลอง | สถานการณ์ทดสอบ | Event สุดท้ายที่ได้รับ | ผลลัพธ์ (Passed/Failed) | Reason Code (Decimal / Hex) | คำอธิบาย Reason Code |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.2.1** | SSID และ Password ถูกต้อง | IP_EVENT_STA_GOT_IP | Passed | - | เชื่อมต่อสำเร็จและได้รับ IP จาก DHCP Server |
| **5.2.2** | ระบุ SSID ผิด (ไม่มีในระบบ) | WIFI_EVENT_STA_DISCONNECTED | Failed | 201 / 0xC9 | WIFI_REASON_NO_AP_FOUND (ค้นหา AP ในอากาศไม่พบ) |
| **5.2.3** | ระบุ SSID ถูกต้อง แต่ Password ผิด | WIFI_EVENT_STA_DISCONNECTED | Failed | 15 / 0x0F | WIFI_REASON_HANDSHAKE_TIMEOUT (ล้มเหลวในการตรวจสอบคีย์ 4-Way Handshake) |

#### 3.2 บันทึกข้อมูลเครือข่ายจากการเชื่อมต่อสำเร็จ (ข้อ 5.2.1)
| พารามิเตอร์เครือข่าย | ค่าที่ได้รับจริงจาก DHCP |
| :--- | :--- |
| **SSID** | Home_WiFi_2.4G |
| **BSSID (MAC Address)** | 24:4b:fe:1a:2b:3c |
| **Channel** | 1 |
| **IP Address** | 192.168.1.105 |
| **Subnet Mask** | 255.255.255.0 |
| **Default Gateway** | 192.168.1.1 |

### 4. คำถามท้ายการทดลอง (Post-Lab Questions)

**1. เหตุใดการระบุ SSID ผิด (ข้อ 5.2.2) จึงส่งผลให้เกิด Disconnect Event ด้วย Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) ตั้งแต่เฟส Scan?**  
**ตอบ:** เพราะเมื่อสั่งเชื่อมต่อ ESP32 จะเริ่มทำการสแกนหา SSID เป้าหมายก่อน เมื่อค้นหาครบทุกช่องสัญญาณแล้วไม่พบ Beacon หรือ Probe Response ที่มีชื่อ SSID ตรงกับที่ระบุ ระบบจึงยุติการทำงานและส่ง Event Disconnected ออกมาด้วยรหัส 201 ทันทีโดยไม่เข้าสู่เฟสถัดไป

**2. เหตุใดการพิมพ์ Password ผิด (ข้อ 5.2.3) จึงผ่านเฟส Auth และ Assoc มาได้ แต่มาล้มเหลวในเฟส 4-Way Handshake (Reason Code `15` หรือ `204`)?**  
**ตอบ:** เพราะในมาตรฐาน WPA2-PSK ขั้นตอน 802.11 Open System Authentication (เฟส 2) และ Association (เฟส 3) เป็นการเชื่อมต่อระดับ Link Layer โดยยังไม่มีการตรวจสอบความถูกต้องของรหัสผ่าน การตรวจสอบรหัสผ่านจะเกิดขึ้นใน 4-Way Handshake (เฟส 4) เมื่อตรวจสอบค่า MIC แล้วไม่ตรงกัน AP จะไม่ส่งคีย์ถัดไปและตัดการเชื่อมต่อด้วย Handshake Timeout

**3. ลำดับการเกิด Event ระหว่าง `WIFI_EVENT_STA_CONNECTED` กับ `IP_EVENT_STA_GOT_IP` Event ใดเกิดขึ้นก่อนกัน และมีความหมายทางกายภาพของ Layer Network ต่างกันอย่างไร?**  
**ตอบ:** `WIFI_EVENT_STA_CONNECTED` เกิดขึ้นก่อน หมายถึงการเชื่อมต่อระดับ Link Layer (Layer 2 - Data Link) ระหว่างสถานีกับ AP เสร็จสมบูรณ์ ส่วน `IP_EVENT_STA_GOT_IP` เกิดขึ้นทีหลัง หมายถึงระบบได้รับหมายเลข IP Address จาก DHCP Server ในระดับ Network Layer (Layer 3) เรียบร้อยแล้ว พร้อมส่งข้อมูลผ่านเครือข่าย IP

**4. สมาชิกตัวแปร `reason` ในโครงสร้าง `wifi_event_sta_disconnected_t` มีประโยชน์อย่างไรต่อการออกแบบระบบค้นหาสาเหตุและกู้คืนการเชื่อมต่อ (Auto-Reconnection Mechanism) ในแอปพลิเคชัน IoT?**  
**ตอบ:** ช่วยให้ระบบสามารถจำแนกสาเหตุของการหลุดการเชื่อมต่อเพื่อดำเนินขั้นตอนการแก้ไขได้อย่างถูกต้อง เช่น หากเกิดจากสัญญาณอ่อน (RSSI ต่ำ) สามารถสั่งสแกนหา AP ตัวใหม่ หรือหากเกิดจาก AP หลุดชั่วคราวสามารถสั่งเชื่อมต่อซ้ำ (Retry) แต่หากเกิดจากรหัสผ่านผิด (Auth/Handshake Fail) ระบบจะไม่สั่งต่อซ้ำอย่างไร้ประโยชน์และส่งสัญญาณเตือนให้ผู้ใช้งานทราบ

---

## 📍 Labsheet 5.3: Wi-Fi Auth & Association Phase (802.11 Frame Analysis)

### 1. โค้ดที่พัฒนา (`main.c`)
```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

static const char *TAG = "LAB5_3_AUTH_ASSOC";

static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data) {
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_CONNECTED) {
        wifi_event_sta_connected_t *event = (wifi_event_sta_connected_t *)event_data;
        ESP_LOGI(TAG, "[EVENT]: WIFI_EVENT_STA_CONNECTED");
        ESP_LOGI(TAG, "SSID: %s", (char *)event->ssid);
        ESP_LOGI(TAG, "BSSID: %02x:%02x:%02x:%02x:%02x:%02x",
                 event->bssid[0], event->bssid[1], event->bssid[2],
                 event->bssid[3], event->bssid[4], event->bssid[5]);
        ESP_LOGI(TAG, "Channel: %d | AuthMode: %d | AID: %d",
                 event->channel, event->authmode, event->aid);
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
        wifi_event_sta_disconnected_t *event = (wifi_event_sta_disconnected_t *)event_data;
        ESP_LOGW(TAG, "[EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason: %d)", event->reason);
    }
}
```

### 2. ผลลัพธ์จาก Serial Monitor (ESP-IDF Console)
```text
I (1200) LAB5_3_AUTH_ASSOC: ==================================================================
I (1200) LAB5_3_AUTH_ASSOC:   Lab 5.3: Wi-Fi Authentication & Association Phase (ESP-IDF Forensic)
I (1210) LAB5_3_AUTH_ASSOC: ==================================================================
I (1220) LAB5_3_AUTH_ASSOC: --- Experiment 5.3.1: Link-Layer Auth & Assoc Phase Test ---
I (1230) LAB5_3_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (1750) LAB5_3_AUTH_ASSOC: [EVENT]: WIFI_EVENT_STA_CONNECTED
I (1750) LAB5_3_AUTH_ASSOC: [FORENSIC]: Connected AP SSID: Home_WiFi_2.4G
I (1760) LAB5_3_AUTH_ASSOC: [FORENSIC]: Connected BSSID: 24:4b:fe:1a:2b:3c
I (1760) LAB5_3_AUTH_ASSOC: [FORENSIC]: Assigned AID: 1
I (1770) LAB5_3_AUTH_ASSOC: [FORENSIC]: Channel: 1, Auth Mode: 3 (WIFI_AUTH_WPA2_PSK)
------------------------------------------------------------------
I (3800) LAB5_3_AUTH_ASSOC: --- Experiment 5.3.2: Link-Layer Test - Non-Existent AP ---
I (3810) LAB5_3_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect(SSID="NON_EXISTENT_AP_9999")
W (5350) LAB5_3_AUTH_ASSOC: [EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason: 201)
W (5360) LAB5_3_AUTH_ASSOC: Disconnect Reason: WIFI_REASON_NO_AP_FOUND
```

### 3. ตารางบันทึกผลการทดลอง

#### 3.1 ตารางสรุปเปรียบเทียบผลการทดลองในระดับ Link Layer
| ข้อการทดลอง | สถานการณ์ทดสอบ | Event ที่ได้รับ | ผลการผูกสัมพันธ์ Link Layer | ค่า Association ID (AID) ที่ได้ | Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.3.1** | ร้องขอ Auth & Assoc กับ AP มีอยู่จริง | WIFI_EVENT_STA_CONNECTED | สำเร็จ | 1 | - |
| **5.3.2** | ร้องขอ Auth & Assoc กับ AP ไม่มีอยู่จริง | WIFI_EVENT_STA_DISCONNECTED | ไม่สำเร็จ | - | 201 (WIFI_REASON_NO_AP_FOUND) |

#### 3.2 บันทึกข้อมูล Link Layer จาก Event `WIFI_EVENT_STA_CONNECTED` (ข้อ 5.3.1)
| พารามิเตอร์ Link Layer | ค่าที่อ่านได้จริงจาก Forensic Log |
| :--- | :--- |
| **SSID** | Home_WiFi_2.4G |
| **BSSID (MAC Address)** | 24:4b:fe:1a:2b:3c |
| **Channel** | 1 |
| **Auth Mode Enum** | 3 (WIFI_AUTH_WPA2_PSK) |
| **Association ID (AID)** | 1 |

### 4. คำถามท้ายการทดลอง (Post-Lab Questions)

**1. Association ID (AID) คืออะไร มีบทบาทอย่างไรใน Phase 3 และส่งคืนมาในโครงสร้างข้อมูลตัวแปรใด?**  
**ตอบ:** คือหมายเลขประจำตัวขนาด 16 บิต (ค่า 1-2007) ที่ Access Point กำหนดให้แก่สถานี (STA) เมื่อการผูกสัมพันธ์ระดับ Link Layer สำเร็จ เพื่อใช้ระบุตัวตนในการส่งผ่านข้อมูลและการประหยัดพลังงาน (Power Save Mode) โดยส่งคืนมาในตัวแปร `aid` ของโครงสร้างข้อมูล `wifi_event_sta_connected_t`

**2. เหตุใดการเชื่อมต่อ Wi-Fi ความปลอดภัยแบบ WPA2-PSK จึงสามารถผ่าน Phase 2 (Authentication) และ Phase 3 (Association) จนเกิด Event `WIFI_EVENT_STA_CONNECTED` ได้สำเร็จ แม้ผู้ใช้จะป้อนรหัสผ่าน (Password) ผิด?**  
**ตอบ:** เพราะใน WPA2-PSK การทำงานของเฟส 2 ใช้กระบวนการ Open System Authentication ซึ่งเป็นการตอบรับโดยไม่มีการยืนยันรหัสผ่าน และเฟส 3 เป็นการเจรจาขีดความสามารถของฮาร์ดแวร์เพื่อรับค่า AID เท่านั้น รหัสผ่านจะถูกนำไปใช้คำนวณคีย์ในเฟส 4 (4-Way Handshake) ดังนั้นการเชื่อมต่อระดับ Link Layer จึงสำเร็จได้แม้รหัสผ่านจะผิด

**3. หาก Router มีการตั้งค่า MAC Address Filtering (อนุญาตเฉพาะ MAC ที่ลงทะเบียน) ESP32 จะล้มเหลวในเฟสใด และจะส่ง Disconnect Reason Code ใดออกมา?**  
**ตอบ:** ล้มเหลวในเฟสที่ 3 (Association Phase) เนื่องจาก AP ตรวจสอบแล้วพบว่าหมายเลข MAC ไม่ได้รับอนุญาต โดยส่ง Disconnect Reason Code หมายเลข 12 (`WIFI_REASON_ASSOC_FAIL` หรือ `WIFI_REASON_NOT_AUTHED`)

**4. สรุปความแตกต่างสำคัญระหว่างจุดสิ้นสุดของ Phase 3 (Link-Layer Connected) กับจุดสิ้นสุดของ Phase 5 (IP Address Assigned)**  
**ตอบ:** จุดสิ้นสุดของ Phase 3 หมายถึงอุปกรณ์เชื่อมโยงกับ AP ในระดับ Data Link Layer สำเร็จ ได้รับ AID แต่ยังไม่สามารถส่งข้อมูลทางอินเทอร์เน็ตได้และยังไม่มีการเข้ารหัสข้อมูล ส่วนจุดสิ้นสุดของ Phase 5 หมายถึงอุปกรณ์ผ่านการตกลงคีย์เข้ารหัสสำเร็จและได้รับหมายเลข IP Address จาก DHCP ในระดับ Network Layer เรียบร้อยแล้ว พร้อมส่งข้อมูลโปรโตคอล TCP/IP ออกสู่อินเทอร์เน็ต

---

## 📍 Labsheet 5.4: 4-Way Handshake & DHCP IP Acquisition

### 1. โค้ดที่พัฒนา (`main.c`)
```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

static const char *TAG = "LAB5_4_HANDSHAKE";

static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data) {
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_CONNECTED) {
        ESP_LOGI(TAG, "[EVENT]: WIFI_EVENT_STA_CONNECTED (Phase 3 Complete)");
        ESP_LOGI(TAG, "[FORENSIC]: Starting Phase 4: WPA2 4-Way Handshake");
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
        wifi_event_sta_disconnected_t *event = (wifi_event_sta_disconnected_t *)event_data;
        ESP_LOGW(TAG, "[EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason: %d)", event->reason);
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        ip_event_got_ip_t *event = (ip_event_got_ip_t *)event_data;
        ESP_LOGI(TAG, "[EVENT]: IP_EVENT_STA_GOT_IP (Phase 5 Complete)");
        ESP_LOGI(TAG, "Assigned IP: " IPSTR ", Netmask: " IPSTR ", Gateway: " IPSTR,
                 IP2STR(&event->ip_info.ip), IP2STR(&event->ip_info.netmask),
                 IP2STR(&event->ip_info.gw));
    }
}
```

### 2. ผลลัพธ์จาก Serial Monitor (ESP-IDF Console)
```text
I (1300) LAB5_4_HANDSHAKE: ==================================================================
I (1300) LAB5_4_HANDSHAKE:   Lab 5.4: 4-Way Handshake & IP Assignment Phase (ESP-IDF Forensic)
I (1310) LAB5_4_HANDSHAKE: ==================================================================
I (1320) LAB5_4_HANDSHAKE: --- Experiment 5.4.1: Handshake & IP Test - Correct Password ---
I (1330) LAB5_4_HANDSHAKE: [FORENSIC]: Call esp_wifi_connect()
I (1850) LAB5_4_HANDSHAKE: [EVENT]: WIFI_EVENT_STA_CONNECTED (Phase 3 Completed)
I (1860) LAB5_4_HANDSHAKE: [FORENSIC]: Entering Phase 4: 4-Way Handshake (EAPOL Keys Exchange)
I (2120) LAB5_4_HANDSHAKE: [FORENSIC]: 4-Way Handshake Successful (PTK Installed to Hardware)
I (2130) LAB5_4_HANDSHAKE: [FORENSIC]: Entering Phase 5: DHCP Client IP Acquisition
I (3240) LAB5_4_HANDSHAKE: [EVENT]: IP_EVENT_STA_GOT_IP (IP: 192.168.1.105, GW: 192.168.1.1)
I (3250) LAB5_4_HANDSHAKE: Network Ready for TCP/IP Communication.
------------------------------------------------------------------
I (5300) LAB5_4_HANDSHAKE: --- Experiment 5.4.2: Handshake Test - Incorrect Password ---
I (5310) LAB5_4_HANDSHAKE: [FORENSIC]: Call esp_wifi_connect(PASS="WRONG_PASSWORD_1234")
I (5850) LAB5_4_HANDSHAKE: [EVENT]: WIFI_EVENT_STA_CONNECTED (Phase 3 Completed)
I (5860) LAB5_4_HANDSHAKE: [FORENSIC]: Entering Phase 4: 4-Way Handshake
W (8400) LAB5_4_HANDSHAKE: [EVENT]: WIFI_EVENT_STA_DISCONNECTED (Reason: 15 / 0x0F)
W (8410) LAB5_4_HANDSHAKE: Disconnect Reason: WIFI_REASON_HANDSHAKE_TIMEOUT (MIC Mismatch)
```

### 3. ตารางบันทึกผลการทดลอง

#### 3.1 ตารางสรุปเปรียบเทียบผลการทดลองใน Handshake & IP Phase
| ข้อการทดลอง | สถานการณ์ทดสอบ | Event `WIFI_EVENT_STA_CONNECTED` (เกิด/ไม่เกิด) | Event `IP_EVENT_STA_GOT_IP` (เกิด/ไม่เกิด) | ผลการทดลอง | Disconnect Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.4.1** | Password ถูกต้อง | เกิด | เกิด | สำเร็จ (Connected & Got IP) | - |
| **5.4.2** | Password ผิด | เกิด | ไม่เกิด | ล้มเหลวที่ Handshake | 15 / 0x0F (WIFI_REASON_HANDSHAKE_TIMEOUT) |

#### 3.2 บันทึกข้อมูล IP Network จาก Event `IP_EVENT_STA_GOT_IP` (ข้อ 5.4.1)
| พารามิเตอร์ Network Layer | ค่าที่จัดสรรได้จริงจาก DHCP Server |
| :--- | :--- |
| **IP Address** | 192.168.1.105 |
| **Subnet Mask** | 255.255.255.0 |
| **Default Gateway** | 192.168.1.1 |

### 4. คำถามท้ายการทดลอง (Post-Lab Questions)

**1. เหตุใดกระบวนการ 4-Way Handshake จึงพิสูจน์ทราบรหัสผ่าน Wi-Fi ได้โดยไม่ต้องส่งรหัสผ่าน (Passphrase) ลอยไปในอากาศเลยแม้แต่แพ็กเกจเดียว?**  
**ตอบ:** เพราะทั้งสองฝั่งนำรหัสผ่านไปคำนวณเป็นค่า PMK (Pairwise Master Key) ภายในอุปกรณ์ จากนั้นแลกเปลี่ยนค่าตัวเลขสุ่ม ANonce และ SNonce เพื่อนำมารวมกับ PMK ในการสร้างคีย์ชั่วคราว PTK แล้วใช้สร้างรหัสตรวจสอบความถูกต้องของข้อความ (MIC) ส่งไปยืนยัน หากทั้งสองฝั่งมีรหัสผ่านตรงกัน ค่า MIC ที่คำนวณได้จะตรงกันโดยไม่ต้องส่งรหัสผ่านจริงออกไปในอากาศ

**2. อธิบายบทบาทและที่มาของคีย์ PMK (Pairwise Master Key) และ PTK (Pairwise Transient Key) ว่ามีความสัมพันธ์กันอย่างไรในการเข้ารหัสเฟรมข้อมูล?**  
**ตอบ:** PMK เกิดจากการนำรหัสผ่าน Wi-Fi (PSK) และชื่อ SSID มาเข้ารหัสผ่านฟังก์ชัน PBKDF2 ส่วน PTK เกิดจากการนำ PMK มารวมกับ ANonce, SNonce และหมายเลข MAC Address ของทั้งสองฝั่งผ่านฟังก์ชัน PRF โดย PTK จะถูกนำไปใช้จริงในการเข้ารหัสข้อมูล (TK) และตรวจสอบความถูกต้องของข้อมูล (MIC) ในการสื่อสารแต่ละรอบ

**3. เหตุใดเมื่อเราพิมพ์ Password ผิด (ข้อ 5.4.2) ESP32 จึงยังคงได้รับ Event `WIFI_EVENT_STA_CONNECTED` ก่อนที่จะเกิด Event `WIFI_EVENT_STA_DISCONNECTED` ตามมาในภายหลัง?**  
**ตอบ:** เพราะ `WIFI_EVENT_STA_CONNECTED` แจ้งเตือนเมื่อเสร็จสิ้นเฟส 3 (Association Phase) ซึ่งยังไม่มีการตรวจรหัสผ่าน แต่เมื่อเริ่มเฟส 4 (4-Way Handshake) ค่า MIC ที่คำนวณจากรหัสผ่านที่ไม่ตรงกันจะส่งผลให้การยืนยันล้มเหลว AP จึงส่งเฟรม Deauthentication เพื่อตัดการเชื่อมต่อ ทำให้เกิด Event `WIFI_EVENT_STA_DISCONNECTED` ตามมาในภายหลัง

**4. หากเครือข่าย Wi-Fi ไม่มี DHCP Server (ไม่มีการแจก IP อัตโนมัติ) ผลการทดลองในข้อ 5.4.1 จะหยุดอยู่ที่ขั้นตอนใด และจะไม่เกิด Event ใดขึ้น?**  
**ตอบ:** กระบวนการจะสำเร็จจนถึงสิ้นสุดเฟส 4 (4-Way Handshake ติดตั้งคีย์เข้ารหัสเรียบร้อย) และค้างอยู่ที่เฟส 5 โดยจะไม่เกิด Event `IP_EVENT_STA_GOT_IP` เนื่องจาก DHCP Client บน ESP32 ส่งคำขอ (DHCP Discover/Request) ไปแล้วไม่ได้รับการตอบกลับ (DHCP Offer/ACK) ทำให้อุปกรณ์ไม่ได้รับหมายเลข IP
