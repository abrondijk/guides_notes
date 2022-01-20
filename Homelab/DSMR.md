
With ser2net 3:

```conf
# /etc/ser2net.conf
23:raw:600:/dev/ttyUSB0:115200 8DATABITS NONE 1STOPBIT XONXOFF LOCAL -RTSCTS
```

With ser2net 4:

```yaml
# /etc/ser2net.yaml
connection: &con1192
    accepter: tcp,23
    enable: on
    options:
      banner: *banner
      kickolduser: true
      telnet-brk-on-sync: true
    connector: serialdev,
              /dev/ttyUSB0,
              115200n81,local
```