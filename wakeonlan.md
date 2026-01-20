enable Wake-on-lan enable in BIOS

chack if enable wake-on in OS with 
```bash
sudo ethtool emp7s0
```
if is d - disable to enable 
```bash
sudo ethtool -s enp7s0 wol g
```
but after restar its again disable 

create service to enable it en start and after
```bash
sudo nano /etc/systemd/system/wol.service
```

```bash
[Unit]
Description=Enable Wake-on-LAN
Requires=network.target
After=network.target

[Service]
Type=oneshot
ExecStart=/sbin/ethtool -s enp7s0 wol g
ExecStop=/sbin/ethtool -s enp7s0 wol g

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start wol.service
systemctl enable wol.service
systemctl is-enabled wol.service
```