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

## 🔧 Langkah-Langkah Implementasi

### A. Koneksi Sensor dan Raspberry Pi
1. Pastikan Raspberry Pi dalam keadaan OFF
2. Rangkai sensor sesuai tabel koneksi pin di atas
3. Periksa kembali rangkaian sebelum menyalakan Raspberry Pi

### B. Setup InfluxDB Cloud
1. Buka https://cloud.2.influxdata.com/signup
2. Sign up menggunakan akun Google
3. Buat company dan organization
4. Pilih storage provider (Amazon Web Services)
5. Pilih Free Plan
6. Buat bucket baru:
   - Pilih menu Buckets
   - Klik CREATE BUCKET
   - Masukkan nama bucket (misal: monitor_suhu)
   - Set Data Retention (misal: 24 hours)
7. Catat token InfluxDB untuk digunakan nanti

### C. Setup Grafana Cloud
1. Buka https://grafana.com/auth/sign-up/create-user
2. Sign up menggunakan akun Google
3. Buat stack Grafana baru

### D. Persiapan Development Environment
1. Install PuTTY
   ```bash
   # Download dari:
   https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   ```

2. Install WinSCP
   ```bash
   # Download dari:
   https://winscp.net/eng/download.php
   ```

3. Koneksikan ke Raspberry Pi via PuTTY:
   - Masukkan IP Raspberry Pi
   - Pilih SSH
   - Login dengan username dan password Raspberry Pi

### E. Setup Virtual Environment
```bash
# Update sistem
sudo apt update
sudo apt upgrade -y

# Cek dan install Python
python3 --version
sudo apt install python3 python3-pip python3-venv -y

# Buat struktur folder
cd Documents
mkdir arsikom2
cd arsikom2
mkdir sensor2
cd sensor2

# Buat dan aktifkan virtual environment
python3 -m venv .
source bin/activate
```

### F. Instalasi Library dan Dependencies
```bash
# Install library sensor DHT
python3 -m pip install adafruit-circuitpython-dht

# Install InfluxDB client
pip install influxdb3-python

# Install pandas
pip install pandas
```

### G. Transfer dan Konfigurasi Kode
1. Buat file `read_datadht.py` dengan kode program di bawah
2. Transfer file ke Raspberry Pi menggunakan WinSCP:
   - Koneksikan ke Raspberry Pi
   - Navigasi ke folder virtual environment
   - Upload file python

3. Export token InfluxDB:
```bash
export INFLUXDB_TOKEN="your-token-here"
```

### H. Setup Dashboard InfluxDB
1. Di InfluxDB Cloud, buka Data Explorer
2. Pilih bucket yang telah dibuat
3. Pilih measurement "datasuhu"
4. Centang field temperature dan humidity
5. Run query untuk memastikan data masuk

### I. Konfigurasi Grafana Dashboard
1. Di Grafana Cloud:
   - Buka Connections
   - Add new connection
   - Pilih InfluxDB

2. Konfigurasi data source:
   - Name: Sesuai keinginan
   - Query Language: SQL
   - URL: URL InfluxDB Cloud
   - Database: Nama bucket
   - Token: Token InfluxDB
   
3. Buat dashboard baru:
   - Add visualization
   - Pilih data source InfluxDB
   - Paste query SQL dari InfluxDB
   - Set refresh rate ke 5 detik
   - Set time range ke 30 menit

4. Konfigurasi visualisasi:
   - Buat Gauge untuk suhu
   - Buat Gauge untuk kelembapan
   - Buat Time-series graph
   - Sesuaikan threshold warna

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

## 📊 Konfigurasi Visualisasi

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

- **Time-Series Graph**: 
  - Menampilkan tren perubahan suhu dan kelembapan
  - Auto refresh setiap 5 detik
  - Range data 30 menit

## 🔍 Troubleshooting

1. **Sensor tidak terbaca:**
   - Periksa koneksi pin
   - Pastikan library DHT terinstall dengan benar
   - Coba restart Raspberry Pi

2. **Data tidak masuk ke InfluxDB:**
   - Periksa koneksi internet
   - Verifikasi token InfluxDB
   - Pastikan bucket name benar

3. **Grafana tidak menampilkan data:**
   - Periksa konfigurasi data source
   - Verifikasi query SQL
   - Periksa time range dashboard

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
