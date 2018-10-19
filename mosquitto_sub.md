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

## Example
```bash
$ mosquitto_sub \
    --cafile ~/.ssh/mqtt/ca.crt \
    --port 8883 \
    --host emoncms.org -S \
    --topic user/emrys/request/# \
    --id-prefix emoncms_ \
    --username emrys \
    --pw PASSWORD
```