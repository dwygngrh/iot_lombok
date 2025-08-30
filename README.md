# iot_lombok
# 1. setting software, lakukan sekali saja
sudo apt update  
sudo apt install net-tools  
sudo apt install openssl -y  
sudo apt install -y mosquitto mosquitto-clients  

# Aktifkan Mosquito. 
# Untuk nonaktif bisa mengganti perintah: 
# enable-->disable. start-->stop  
sudo systemctl enable mosquitto  
sudo systemctl start mosquitto  

# Membuat direktory file certifikat dan key buat TLS   
mkdir ~/mqtt_certs && cd ~/mqtt_certs  

## Buat CA (Certificate Authority)
openssl genrsa -out ca.key 2048  
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt  
openssl genrsa -out server.key 2048  

## Buat CSR (Certificate Signing Request)
openssl req -new -key server.key -out server.csr

## Sign server cert with your CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365  

## Cek file sertifikat dan key di ~/mqtt_certs
ca.crt → public CA cert (clients need this)  
server.crt → server TLS certificate  
server.key → server private key  

# Konfigurasi MQTT Mosquito
sudo nano /etc/mosquitto/conf.d/tls.conf  
Paste:  

################################################################
listener 8883  
protocol mqtt  
cafile /home/dnugroho/mqtt_certs/ca.crt  
certfile /home/dnugroho/mqtt_certs/server.crt  
keyfile /home/dnugroho/mqtt_certs/server.key  
  
require_certificate true  
################################################################  

## Restart Mosquitto
sudo systemctl restart mosquitto  

## Check it’s listening on secure port:
sudo netstat -tlnp | grep 8883  

# Lakukan transfer data dari IOT  

# berikan perintah ini di server jika menggunakan password
# topic ialah sensor/temp

mosquitto_sub -h localhost -p 8883 \  
 --cafile /etc/mosquitto/certs/ca.crt \  
 -u myiotuser -P mypassword \  
 -t "sensors/#" -v  

 # berikan perintah ini di server jika tanpa password
# topic ialah sensor/temp

mosquitto_sub -h localhost -p 8883 \  
 --cafile /etc/mosquitto/certs/ca.crt \  
 -u myiotuser -P mypassword \  
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

## Store raw data menggunakan script python

