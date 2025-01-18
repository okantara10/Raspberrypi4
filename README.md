# Visualisasi Data Sensor Suhu dan Kelembapan DHT11 pada Raspberry Pi Menggunakan InfluxDB dan Grafana

Proyek untuk memvisualisasikan data sensor suhu dan kelembapan menggunakan Raspberry Pi, InfluxDB Cloud, dan Grafana Cloud.

## 📝 Deskripsi

Proyek ini memanfaatkan Raspberry Pi model 4 untuk membaca data suhu dan kelembapan dari sensor DHT11. Data tersebut kemudian dikirimkan ke platform database InfluxDB Cloud dan divisualisasikan menggunakan dashboard Grafana Cloud.

### Komponen Utama:
- **Raspberry Pi**: Single board computer berbasis Linux untuk membaca data sensor
- **DHT11**: Sensor untuk mengukur suhu dan kelembapan
- **InfluxDB Cloud**: Database time-series untuk penyimpanan data
- **Grafana Cloud**: Platform visualisasi data interaktif

## 🛠️ Alat dan Bahan

| No | Nama Alat/Bahan | Jumlah |
|----|-----------------|---------|
| 1 | Raspberry Pi 4 | 1 buah |
| 2 | Sensor suhu DHT11 | 1 buah |
| 3 | InfluxDB Cloud | 1 akun |
| 4 | Grafana Cloud | 1 akun |
| 5 | Kabel jumper | Secukupnya |

## 📋 Koneksi Pin

| Pin DHT11 | Pin Raspberry Pi |
|-----------|-----------------|
| VCC+ | 3.3V |
| out | GPIO4 |
| GND- | GND |

## 🔧 Langkah Instalasi

### A. Persiapan Awal
1. Install PuTTY dan WinSCP
2. Buat akun InfluxDB Cloud dan Grafana Cloud
3. Pastikan Raspberry Pi terhubung ke internet

### B. Setup Virtual Environment
```bash
# Update sistem
sudo apt update
sudo apt upgrade -y

# Install Python dan dependensi
sudo apt install python3 python3-pip python3-venv -y

# Buat dan aktifkan virtual environment
cd Documents
mkdir arsikom2/sensor2 -p
cd arsikom2/sensor2
python3 -m venv .
source bin/activate
```

### C. Instalasi Library
```bash
# Install library yang diperlukan
python3 -m pip install adafruit-circuitpython-dht
pip install influxdb3-python
pip install pandas
```

## 💻 Kode Program

```python
import os
import time
from influxdb_client_3 import InfluxDBClient3, Point
import board
import adafruit_dht

# Konfigurasi InfluxDB
token = os.environ.get("INFLUXDB_TOKEN")
org = "JTE"
host = "https://us-east-1-1.aws.cloud2.influxdata.com"

# Inisialisasi klien
client = InfluxDBClient3(host=host, token=token, org=org)
database = "monitor_suhu"

# Inisialisasi sensor DHT11
sensor = adafruit_dht.DHT11(board.D4)

# Main loop
try:
    while True:
        try:
            temperature = sensor.temperature
            humidity = sensor.humidity
            
            # Create points
            point_temperature = (
                Point("datasuhu")
                .tag("location", "RaspberryPi")
                .field("temperature", temperature)
            )
            
            point_humidity = (
                Point("datasuhu")
                .tag("location", "RaspberryPi")
                .field("humidity", humidity)
            )
            
            # Write to database
            client.write(database=database, record=point_temperature)
            client.write(database=database, record=point_humidity)
            
            print(f"Data sent: Temperature={temperature}, Humidity={humidity}")
            
        except RuntimeError as e:
            print(f"DHT read error: {e.args[0]}. Retrying...")
            time.sleep(2.0)
            continue
            
        time.sleep(5)
        
except KeyboardInterrupt:
    print("Program dihentikan.")
```

## 📊 Visualisasi Data

### Dashboard Grafana
- **Gauge Chart Suhu**:
  - 0-10°C: Biru
  - 10-20°C: Biru Muda
  - 20-25°C: Hijau
  - 25-30°C: Kuning
  - 30-38°C: Jingga
  - >40°C: Merah

- **Gauge Chart Kelembapan**:
  - <25%: Hijau (kondisi kering)
  - 25-50%: Merah (kondisi optimal)
  - 50-75%: Kuning (kondisi lembap)
  - 75-100%: Biru (kondisi sangat lembap)

- **Time-Series Graph**: Menampilkan tren perubahan suhu dan kelembapan terhadap waktu

## 📚 Referensi

- [Random Nerd Tutorials - DHT11/DHT22 with Raspberry Pi](https://randomnerdtutorials.com/raspberry-pi-dht11-dht22-python/)
- [InfluxDB Cloud Documentation](https://cloud.2.influxdata.com/signup)
- [Grafana Cloud Documentation](https://grafana.com/)
- Dwi, A., Suaidah, & Prayogo, A. (2021). Teknologi Pengendali Perangkat Elektronik Menggunakan Sensor Suara. *Jurnal Teknologi dan Sistem Tertanam*, 2(2), 46-59.
- Friadi, R., Junadhi, J. (2019). Sistem Kontrol Intensitas Cahaya, Suhu dan Kelembaban Udara Pada Greenhouse Berbasis Raspberry PI. *JTIS*, 2(1), 30-36.

## 👥 Kontributor

- Gede Agung Okantara (2315344011)
- I Made Wiranata Kusuma Putra (2315344005)

## 🏫 Institusi

Program Studi Teknik Otomasi  
Jurusan Teknik Elektro  
Politeknik Negeri Bali
