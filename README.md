# iot_lombok, Gerupuk 30 Agustus 2025    
Langkah yang harus dilakukan:
1. Install MQTT server di server. Bisa memakai laptop sebagai server. Harus memakai IP Public.  
2. Install grafana  
3. Untuk tampilan advance bisa menggunakan apache superset. tapi installnya susaah
4. Setelah MQTT server siap. Ubah code python di raspberry 3b+ (raspi) dari yang awalnya hanya membaca data, diubah menjadi membaca data dan mengirimkan ke server MQTT format JSON menggunakan modem GSM

## 1. Install MQTT server menggunakan software mqtt server mosquito
#### setting software, lakukan sekali saja  
sudo apt update  
sudo apt install net-tools  
sudo apt install openssl -y  
sudo apt install -y mosquitto mosquitto-clients  
sudo apt install -y python3-pip python3-venv build-essential     libpq-dev libsasl2-dev libldap2-dev libssl-dev  
pip install paho-mqtt  
sudo apt install certbot  
### Aktifkan Mosquito. 
#### Untuk nonaktif bisa mengganti perintah: enable-->disable. start-->stop
sudo systemctl enable mosquitto  
sudo systemctl start mosquitto  
##### Membuat direktory file certifikat dan key buat TLS   
mkdir ~/mqtt_certs && cd ~/mqtt_certs  
### Buat CA (Certificate Authority)
./create_CA.yoga  
./create_certificate.yoga
##### Cek apakah seluruh file sertifikat dan key di tersedia di  ~/mqtt_certs
ca.crt → untuk public CA cert (clients need this)  
server.crt → untuk server TLS certificate  
server.key → untuk server private key  
##### Konfigurasi MQTT Mosquito
sudo nano /etc/mosquitto/conf.d/tls.conf  
Paste ke dalam tls.conf:  
listener 8883  
protocol mqtt  
cafile /home/dnugroho/mqtt_certs/ca.crt  
certfile /home/dnugroho/mqtt_certs/server.crt  
keyfile /home/dnugroho/mqtt_certs/server.key  
require_certificate true  

## file ser
### Restart Mosquitto
sudo systemctl restart mosquitto  
### Check it’s listening on secure port:
sudo netstat -tlnp | grep 8883  

## Install GRAFANA  
ikuti tutorial ini:  
https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/

## 3. Ubah code di raspberry pi
### Gunakan chat gpt dan berikan perintah berikut:
modify python code in raspberry to send the data using GSM modeem over MQTT 
MQTT_broker : 103.56.92.100  
MQTT_port : 8883  
MQTT_topic : "MBKM"  
MQTT_CERT : "direktory/ca.crt"  
MQTT_KEY : "direktory/client.key"  
MQTT_CRT : "certificate/client.crt"
ketiga file certificate di dapatkan saat pembuatan Certificate authority di server. 


## Lakukan transfer data dari IOT  
### berikan perintah ini di server jika menggunakan password
### topic contoh :  sensor/temp

mosquitto_sub -h localhost -p 8883 \  
 --cafile /etc/mosquitto/certs/ca.crt \  
 -u myiotuser -P mypassword \  
 -t "sensors/#" -v  

# berikan perintah ini di server jika tanpa password
# topic ialah sensor/temp  
mosquitto_sub -h localhost -p 8883 \  
 --cafile /etc/mosquitto/certs/ca.crt \  
 -t "sensors/#" -v  

## Tampilkan data sensor
### bisa Store data dalam database → e.g., Python script writing to InfluxDB, PostgreSQL, or MongoDB.
### bisa langsung tampil di Dashboard → use Grafana atau Apache Superset to visualize.

import paho.mqtt.client as mqtt  

def on_message(client, userdata, msg):  
    print(f"Topic: {msg.topic}, Message: {msg.payload.decode()}")  
  
client = mqtt.Client()  
client.tls_set(ca_certs="ca.crt")  
#client.username_pw_set("myiotuser", "mypassword")  
client.connect("103.56.92.100", 8883)  
client.subscribe("sensors/#")  
client.on_message = on_message  
client.loop_forever() 

## save data sensor ke dalam file. running file: 

## Store raw data menggunakan script python

