# fluent-plugin-loki

[Fluentd](https://fluentd.org/) output plugin to [Grafana Loki](https://github.com/grafana/loki).

- Can be used to ship docker logs to Loki (using [Fluentd docker logging driver](https://docs.docker.com/config/containers/logging/fluentd/))
- Enable easier transtion to Loki - an alternative to Loki's Promtail 

## Installation

### RubyGems

```
$ gem install fluent-plugin-loki
```

## Configuration
sample configuration:
```
<match *>
  @type loki
  endpoint_url    http://127.0.0.1:3100
  labels          {"env":"prod","farm":"a"} # default: nil
  tenant          abcd           # default: nil
  rate_limit_msec 100            # default: 0 = no rate limiting
  raise_on_error  false          # default: true
  authentication  basic          # default: none
  username        alice          # default: ''
  password        bobpop         # default: '', secret: true
  buffered        true           # default: false. Switch non-buffered/buffered mode
  cacert_file     /etc/ssl/endpoint1.cert # default: ''
  token           tokent         # default: ''
  custom_headers  {"token":"arbitrary"} # default: nil
</match>
```
- **endpoint_url** - Loki's endpoint
- **tenant** - Loki tenant id
- **labels** - Labels for filtering in Grafana (currently they are static)

(generated by running: ```fluent-plugin-config-format output loki -p lib/fluent/plugin```)

## Usage examples for common fluentd inputs:
### syslog
*Setting up rsyslog*
Open ```/etc/rsyslog.d/50-default.conf``` and append the following line:
```*.* @127.0.0.1:5140```
Then restart the rsyslogd service:
```sudo systemctl restart syslog```
*sample fluentd config:*
```
<source>
  @type syslog
  port 5140
  bind 0.0.0.0
  tag system
</source>
```
*simulate data:*
```logger "came from syslog"```
### file
*sample fluentd config:*
```
## File input
## read apache logs with tag=apache.access
#<source>
#  @type tail
#  format apache
#  path /var/log/httpd-access.log
#  tag apache.access
#</source>
```
### HTTP
*sample fluentd config:*
```
# HTTP input
# http://localhost:8888/<tag>?json=<json>
# for ex: http://localhost:8888/baz?json={"src":"http"}
<source>
  @type http
  @id http_input

  port 8888
</source>
```
*simulate data:*
```bash
curl -X POST -d 'json={"src":"http"}' http://localhost:8888
```
### tcp
*sample fluentd config:*
```
## built-in TCP input
## $ echo <json> | fluent-cat <tag>
<source>
  @type forward
  @id forward_input
</source>
```
*simulate data:*
```bash
echo '{"src":"tcp"}' | fluent-cat tcp
```

## Development (using Docker)
- Set up Loki and Grafana (can be done by running their [docker-compose](https://github.com/grafana/loki/blob/master/production/docker-compose.yaml))
- Add loki data source to grafana
- Run Fluentd container with volume mapping to the dummy configuration and to the plugin:
```bash
docker run -d \
  -p 9880:9880 \
  -v $(pwd)/development/conf:/fluentd/etc \
  -v $(pwd)/lib/fluent/plugin:/etc/fluent/plugin -e FLUENTD_CONF=fluentd.conf \
  fluent/fluentd
```
- Execute to send log: ```curl -X POST -d 'json={"foo":"baz"}' http://localhost:9880```


## Copyright

* Copyright(c) 2018- Edan Shahmoon
* License
  * Apache License, Version 2.0

Heavily based on [fluent-plugin-out-http](https://github.com/fluent-plugins-nursery/fluent-plugin-out-http)
