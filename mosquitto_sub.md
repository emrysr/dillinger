# MOSTUITTO MQTT SERVER
Connect to MQTT broker with Mosquitto Client `mosquitto_sub`. Further detail in [mosquitto docs](https://mosquitto.org/man/mosquitto_sub-1.html).

## Command Options
Here are the commands parameters this project uses:
- `--cafile` file containing PEM encoded CA certificates that are trusted
- `--topic` this may be repeated to subscribe to multiple topics
- `--port` port number. Default for `mqtt://` is `1883`. Default for `mqtts://` is `8883`
- `--cert` file containing `PEM` encoded certificate for this client
- `--qos` default is `0`
- `--ciphers` list of supported ciphers
- `--debug` enable debug messages
- `--host` defaults to `localhost`
- `--id` client id. defaults to `mosquitto_sub_<process_id>`
- `--id-prefix` client id prefix. `<prefix><process_id>` will be used as the client id. doesn't work with `--id`
- `--insecure` disable verification of server hostname. **not to be used in production**
- `--username` username
- `--pw` password
- `-S` makes a DNS `SRV` request for `_mqtt._tcp.<host>` to determin the hostname/ip address
- `--tls-version` *[tlsv1.2|tlsv1.1|tlsv1]* defaults to `tlsv1.2` - must match the protocol used the broker
- `--verbose` prints the topic and the payload when message received
- `--protocol-version` *[mqttv311|mqttv31]* defaults to `mqttv311`

## MQTT over TLS Example using mosquitto_sub client
(not confirmed to work)
```bash
$  \
    --cafile /home/emrys/.ssh/mqtt/ca.crt.pem
    --port 8883
    --host localhost
    --topic user/dave/request/#
    --id-prefix emoncms_
    --username dave
    --pw dave
    --insecure
```
## Secure Websocket over TLS example using PAHO nodejs mqtt client
install cli on your system: ```$ npm install mqtt -g```
```bash
mqtt pub -t user/dave/request --ca /home/emrys/.ssh/mqtt/ca.crt.pem --insecure --protocol wss -u dave -P dave -m hi -p 8081
```

### mqtt
```
$ mqtt pub -t user/dave/request  --protocol mqtt -u dave -P dave -m "hello world" -p 1883
```

### using nodejs mqtt client
```
mqtt pub -t user/dave/request --ca /home/emrys/.ssh/mqtt/ca.crt.pem --insecure --protocol mqtts -u dave -P dave -m "http://localhost/emoncms/feed/list.json?apikey=cb9579be83678b89a5eb0faea08ad83"
```
```
mqtt sub -t user/dave/request --ca /home/emrys/.ssh/mqtt/ca.crt.pem --insecure --protocol mqtts -u dave -P dave
```
> supported protocols: 'mqtt', 'mqtts', 'tcp', 'tls', 'ws', 'wss'.


## Keys and SSL Certificates
Each client needs a copy of the signed public key generated form the Broker's private key. The private key will reside on the public MQTT Broker and will be used to verify the encrypted messages from each client.

Before the content is sent to the Broker the client connects to the broker using SSL. The broker will respond with a Server Certificate that contains a signed public key. The client then checks the key's signature with the CA to validate it's origin.

The "trusted" public key is then used to encrypt a session key that is then "agreed" with the broker before content is securly transfered using this session key.

### Certificate Authority
On the production server a Certificate Authority (CA) will be used to sign the keys to verify its origin. The CA does some checks, and sends you back the keys enclosed in a certificate. This signed certificate is what is sent to the clients on connection.

For development purposes we'll "self sign" the certificate and use the `--insecure` option to ignore the certificate signature verification. This will encrypt the data but the Broker's identity is not "trusted". **NOT TO BE USED IN PRODUCTION**

## Create and sign the keys

### Create CA
this is not required in production as you'll use a "real" CA to sign your certificates:
```
openssl req -new -x509 -days 3650 -extensions v3_ca -keyout ca.key.pem -out ca.crt.pem
```
here's the breakdown of the above line:
- `openssl req` create a PKCS#10 X.509 Certificate Signing Request.
- `-new` promp for field values
- `x509` output self signed certificate
- `-days 365` certificate valid for 1 year (default 30 days)
- `-extensions v3_ca` selects `v3_root_cs` from the config settings
- `-keyout ca.key.pem` save private key as `ca.key.pem`
- `-out ca.crt.pem` save the certificate as `ca.crt.pem`

### Generate a server key
```
openssl genrsa -out mqtt.local.key.pem 2048
```
options used:
- `openssl genrsa` generata an RSA private key
- `-out mqtt.local.key.pem` save the file as `mqtt.local.key.pem`
- `2048` size of the private key in bits

### Generate a certificate signing request
This will be done by the "real" CA in production.
```
openssl req -out mqtt.local.csr.pem -key mqtt.local.key.pem -new
```
> see *Create CA* above for the option decriptions.

When prompted for these fields:
- **Common Name** - must match domain used in the 
- **Password** - use the password used to create your CA

### Sign the key with your own CA
```
openssl x509 -req -in mqtt.local.csr.pem -CA ca.crt.pem -CAkey ca.key.pem -CAcreateserial -out mqtt.local.crt.pem -days 365
```
- `openssl x509` sign the certificate
- `-req`
- `-in`

# Websockets (ws://)
the browser can connect to resources starting with `ws://` however you must have a valid CA signed certificate to use a `wss://` connection. For testing you must drop the secure connection option. 

# Web content (http://)
The mosquitto MQTT broker has been setup to respond with the js websockets testing client. This would then connect to the mosquitto broker via a websocket connection

# Ports
you can specify any port you want for the connections but here are the default ones:
- mqtt = 1883
- mqtts = 8883
- mqtt over websockets = 8080 (might change this to default http 80)
- mqtt over secure websockets = 8081 (might change this to default https 443)

# QOS
### MQTT supports three service levels qualities:

- `QoS = 0` - **one delivery at most**.
    The message is delivered according to the capabilities of the underlying network. No response is sent by the receiver and no retry is performed by the sender. The receiver gets the message either once or not at all.
- `QoS = 1` - **one delivery at least**.
    This quality of service ensures that the message arrives at the receiver at least once, but there’s a probability of duplicating messages on the receiver’s side. If the publisher has not received the acknowledgment of delivery from the message broker, it sends the message again. After the duplicated message is received by the broker, the latter sends it again to all subscribers.
- `QoS  = 2` -  **one delivery exactly**.
    This is the highest quality of service. It is used when neither loss nor duplication of messages are acceptable.

I'm using QoS 1 in my initial testing.

# installed mosquitto versions
- `/usr/local/sbin/mosquitto` (v1.5.x) works with `/etc/mosquitto/auth-plugin3.so`
- `/usr/sbin/mosquitto` (v1.4.x) works with `/etc/mosquitto/auth-plugin2.so`

# todo
- authentication
  use a username and password to authenticate
  - password hashes: `$ np` is the command to generate the password hashes
- authorization
  use a access control list (acl) to restrict authourized users to specific topics
 


