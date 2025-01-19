# Visualisasi Data Sensor Suhu dan Kelembapan DHT11 pada Raspberry Pi Menggunakan InfluxDB dan Grafana

Proyek untuk memvisualisasikan data sensor suhu dan kelembapan menggunakan Raspberry Pi, InfluxDB Cloud, dan Grafana Cloud.

## 📝 Deskripsi

Perkembangan teknologi saat ini telah berkembang dengan pesat di berbagai bidang, salah satunya adalah bidang monitoring. Dengan kemajuan teknologi, hambatan jarak dan waktu kini dapat diatasi melalui Internet of Things (IoT). Salah satu perangkat yang mendukung teknologi IoT adalah Raspberry Pi, sebuah single board computer (SBC) berbasis sistem operasi Linux yang sangat populer dan banyak digunakan.

Dalam proyek ini, kami memanfaatkan Raspberry Pi model 4 untuk membaca data suhu dan kelembapan dari sensor DHT11, lalu mengirimkannya ke platform database InfluxDB Cloud, yang selanjutnya akan dikoneksikan dengan platform dashboard Grafana Cloud.

### Komponen Utama:
- **Raspberry Pi**: Single board computer berbasis Linux
- **DHT11**: Sensor gabungan yang mampu mengukur suhu dan kelembapan relatif
- **InfluxDB Cloud**: Database time-series untuk penyimpanan data
- **Grafana Cloud**: Platform visualisasi data interaktif

## 🛠️ Alat dan Bahan

| No | Nama Alat/Bahan | Jumlah |
|----|-----------------|---------|
| 1 | Raspberry Pi 4 | 1 buah |
| 2 | Sensor suhu DHT11 | 1 buah |
| 3 | InfluxDB Cloud | 1 akun |
| 4 | Grafana cloud | 1 akun |
| 5 | Kabel jumper | Secukupnya |

## 🔧 Langkah-Langkah Implementasi

### A. Koneksi Sensor dan Raspberry Pi

#### Tabel Koneksi
| Pin DHT11 | Pin Raspberry Pi |
|-----------|-----------------|
| VCC+ | 5V |
| out | GPIO4 |
| GND- | GND |

1. Pastikan Raspberry Pi dalam keadaan padam (OFF)
2. Rangkai sensor sesuai tabel koneksi di atas
3. Periksa kembali rangkaian sebelum menyalakan Raspberry Pi

### B. Membuat Akun InfluxDB
1. Buka https://cloud.2.influxdata.com/signup
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images1.png)
2. Pilih SIGN UP, kemudian pilih Google

![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images2.png)
3. Masuk dengan alamat email dan password Google
4. Buat nama company dan organization
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images3.png)
5. Checklist Agreement dan pilih CONTINUE
6. Pilih storage provider pada Amazon Web Services
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images4.png)
7. Pilih opsi Free plan dan pilih KEEP

### C. Membuat Akun Grafana Cloud
1. Buka https://grafana.com/auth/sign-up/create-user
2. Pilih ikon akun Google untuk Sign Up
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images5.png)
3. Masuk dengan alamat email dan password Google
4. Buat stack Grafana baru
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images7.png)

### D. Pembuatan Virtual Environment

#### 1. Setup PuTTY
1. Download dan install PuTTY dari https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
2. Jalankan aplikasi PuTTY
3. Masukkan IP Raspberry Pi
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images8.png)
4. Pilih SSH untuk koneksi
5. Masukkan username dan password Raspberry Pi
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images9.png)

#### 2. Update Sistem
```bash
sudo apt update
sudo apt upgrade -y
```

#### 3. Cek dan Install Python
```bash
python3 --version
sudo apt install python3 -y
sudo apt install python3-pip -y
sudo apt install python3-venv -y
```

#### 4. Buat Struktur Folder
```bash
cd Documents
mkdir arsikom2
cd arsikom2
mkdir sensor2
cd sensor2
```

#### 5. Setup Virtual Environment
```bash
python3 -m venv .
source bin/activate
```

### E. Membuat Bucket dan Token InfluxDB

1. Pada laman utama InfluxDB:
   - Pilih menu Buckets
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images10.png)
   - Pilih CREATE BUCKET
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images11.png)
   - Masukkan nama bucket (misal: monitor_suhu)
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images12.png)
   - Set Data Retention (misal: 24 hours)
   - Pilih CREATE

2. Tambahkan sumber data:
   - Pilih ADD DATA
   - Pilih Client Library
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images13.png)
   - Pilih Python
   - Ikuti tahapan Install Dependencies

3. Dapatkan dan simpan token:
   - Pilih tahap Get Token
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images14.png)
   - Salin token dan simpan di text editor dan PuTTY(token hanya muncul satu kali)

### F. Menginstal Library dan Source Code

#### 1. Install Dependencies InfluxDB
```bash
pip install influxdb3-python
pip install pandas
```

#### 2. Install Library Sensor
```bash
python3 -m pip install adafruit-circuitpython-dht
```

#### 3. Setup WinSCP
1. Download dan install WinSCP dari https://winscp.net/eng/download.php
2. Buka WinSCP dan koneksikan ke Raspberry Pi:
   - Masukkan IP Raspberry Pi
   - Masukkan username dan password
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images15.png)
3. Transfer file python ke folder virtual environment
   ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images16.png)

### G. Kode Program

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

### H. Menerima Data pada InfluxDB

1. Buka laman Buckets di InfluxDB
2. Pilih bucket yang telah dibuat
3. Pilih measurement "datasuhu"
   ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images17.png)
5. Checklist field humidity dan temperature
6. Pastikan query SQL tampil tanpa error
   ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images118.png)
8. Pilih RUN untuk melihat data masuk

### I. Membuat Koneksi dan Dashboard pada Grafana Cloud

1. Setup koneksi:
   - Pilih Add new connection
   - Pilih InfluxDB
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images19.png)
   - Pilih Add new data source
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images20.png)

2. Konfigurasi data source:
   - Name: Sesuai keinginan
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images21.png)
   - Query Language: SQL
   - URL: URL InfluxDB Cloud
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images22.png)
   - Database: Nama bucket
   - Token: Token InfluxDB

3. Buat dashboard:
   - Pilih New dashboard
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images23.png)
   - Add visualization
   - Pilih data source yang dibuat
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images24.png)
   - Paste query SQL dari InfluxDB
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images25.png)
   - Set auto refresh 5 detik
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images26.png)
   - Set time range 30 menit

4. Konfigurasi visualisasi:
   - Ubah ke mode gauge
   - Set nilai min 0 dan max 100
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images27.png)
   - Tambahkan judul dan deskripsi panel
   - Save dashboard
     ![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images28.png)

## 📊 Konfigurasi Visualisasi

### Dashboard Grafana
![alt text](https://github.com/okantara10/rasberrypi-influxdb-grafanacloud/blob/main/images/images29.png)
1. Gauge Chart untuk Suhu:
   - 0-10°C: Biru
   - 10-20°C: Biru Muda
   - 20-25°C: Hijau
   - 25-30°C: Kuning
   - 30-38°C: Jingga
   - >40°C: Merah

2. Gauge Chart untuk Kelembapan:
   - <25%: Hijau (kondisi kering)
   - 25-50%: Merah (kondisi optimal)
   - 50-75%: Kuning (kondisi lembap)
   - 75-100%: Biru (kondisi sangat lembap)

3. Time-Series Graph:
   - Menampilkan perubahan suhu dan kelembapan dari waktu ke waktu
   - Memudahkan pengguna untuk melihat tren data
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
- [PuTTY Download](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
- [WinSCP Download](https://winscp.net/eng/download.php)
- Dwi, A., Suaidah, & Prayogo, A. (2021). Teknologi Pengendali Perangkat Elektronik Menggunakan Sensor Suara. *Jurnal Teknologi dan Sistem Tertanam*, 2(2), 46-59.
- Friadi, R., Junadhi, J. (2019). Sistem Kontrol Intensitas Cahaya, Suhu dan Kelembaban Udara Pada Greenhouse Berbasis Raspberry PI. *JTIS*, 2(1), 30-36.
- Putra, D. (2023, 9 Februari). Get to know InfluxDB is? Complete explanation.
- Jagoan Hosting. (2024, 1 Oktober). Apa itu Grafana? Fitur, Fungsi dan Kelebihannya.

## 👥 Kontributor

- Gede Agung Okantara (2315344011)
- I Made Wiranata Kusuma Putra (2315344005)

## 🏫 Institusi

Program Studi Teknik Otomasi  
Jurusan Teknik Elektro  
Politeknik Negeri Bali
