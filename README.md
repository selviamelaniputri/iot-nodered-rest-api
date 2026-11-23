# REST API Node-RED untuk Sistem IoT

Simulator ESP32 mengirim data sensor lewat MQTT, Node-RED menjadi backend REST API + WebSocket, dan dashboard web menampilkan data serta mengirim kontrol.

```
simulasi_esp32.py --MQTT--> Mosquitto --> Node-RED --REST/WebSocket--> dashboard.html
   (iot/sensor)                            (SQLite)  <-- POST kontrol --
                                          --MQTT (iot/kontrol)--> simulator
```

## Isi Repo
| File | Fungsi |
|---|---|
| `simulasi_esp32.py` | Simulator ESP32 (publish sensor, subscribe kontrol) |
| `flows.json` | Flow Node-RED (backend API) |
| `dashboard.html` | Dashboard web |
| `settings-contoh.txt` | Cuplikan CORS untuk `settings.js` |
| `screenshots/` | Bukti pengujian |

## Prasyarat
Node.js, Node-RED, Mosquitto, Python 3 + `pip install paho-mqtt`, dan node `node-red-node-sqlite` (Manage palette).

## Cara Menjalankan
1. Jalankan broker: `mosquitto.exe -v`
2. Tambahkan isi `settings-contoh.txt` ke `settings.js` Node-RED, lalu jalankan `node-red`.
3. Import `flows.json` (menu > Import), ubah path database pada node SQLite sesuai komputer, lalu Deploy.
4. Ganti `GANTI_DENGAN_API_KEY` di `flows.json` dan `dashboard.html` dengan API key pilihan sendiri.
5. Jalankan simulator: `python simulasi_esp32.py`
6. Buka `dashboard.html` di browser.

## Endpoint
| Endpoint | Method | Fungsi |
|---|---|---|
| `/api/suhu` | GET | Data sensor terakhir |
| `/api/history` | GET | 50 data terakhir (dari SQLite) |
| `/api/kontrol` | POST | Kirim perintah ke ESP32 (`{"device":"lamp","action":"on"}`) |
| `/api/threshold` | PUT | Set batas suhu (`{"batas_suhu":28}`) |
| `ws://localhost:1880/ws/sensor` | WebSocket | Data sensor real-time |

Semua endpoint REST memakai header `X-API-Key`.

## Tugas Pengembangan (Nilai Tambah)
| # | Tugas | Implementasi |
|---|---|---|
| 1 | `PUT /api/threshold` + notifikasi | Batas disimpan di flow context. Jika suhu melewati batas, Node-RED mengirim notifikasi ke topik MQTT `iot/notifikasi`, debug panel, dan log alert di dashboard. |
| 2 | Grafik Chart.js | Grafik suhu dan kelembaban (20 data terakhir) di dashboard. |
| 3 | API key | Header `X-API-Key` dicek di tiap endpoint; salah/kosong membalas `401`. |
| 4 | WebSocket | Node `websocket out` di `/ws/sensor`; dashboard update tanpa polling dan reconnect otomatis. |
| 5 | SQLite | Data disimpan ke `iot.db`; `/api/history` membaca dari database sehingga data bertahan setelah restart. |

## Pengujian
```
curl http://localhost:1880/api/suhu -H "X-API-Key: <API_KEY>"
curl http://localhost:1880/api/history -H "X-API-Key: <API_KEY>"
curl -X POST http://localhost:1880/api/kontrol -H "Content-Type: application/json" -H "X-API-Key: <API_KEY>" -d "{\"device\":\"lamp\",\"action\":\"on\"}"
curl -X PUT http://localhost:1880/api/threshold -H "Content-Type: application/json" -H "X-API-Key: <API_KEY>" -d "{\"batas_suhu\":28}"
```

## Catatan Penyesuaian
- `paho-mqtt` v2 memakai `mqtt.CallbackAPIVersion.VERSION2`.
- Di Node-RED v5, CORS diaktifkan lewat `httpNodeCors` di `settings.js`, karena node `OPTIONS /api/*` tidak menangani preflight.
- Endpoint WebSocket tidak memakai API key karena browser tidak dapat mengirim header kustom pada WebSocket.
- Simulator juga subscribe ke `iot/kontrol` agar perintah dari dashboard dapat diverifikasi.
