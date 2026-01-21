# Auto ssh-keys injector

Start as root
`sudo -i su`

Create file for keys 
```bash
mkdir -p /etc/pve/ssh
touch /etc/pve/ssh/authorized_keys
```

Create injector 
`nano /usr/local/bin/ct-bootstrap.sh`

```bash
#!/bin/bash
set -euo pipefail

CTID="$1"
KEYFILE="/etc/pve/ssh/authorized_keys"
USERNAME="root$(pct exec "$CTID" -- hostname | tr -cd 'a-zA-Z0-9' | head -c 4)"
PASSWORD="toor${CTID}$(pct exec "$CTID" -- hostname | tr -cd 'a-zA-Z0-9' | head -c 5)"

log() {
  echo "$CTID[$(date +'%F %T')] $*" | tee -a /var/log/ct-watcher.log | logger -t ct-watcher
}

log "CTID=$CTID"
log "USERNAME=$USERNAME"

pct exec "$CTID" -- id "$USERNAME" >/dev/null 2>&1 && { log "User exists, skipping"; exit 0; }

log "Creating user"
pct exec "$CTID" -- useradd -m -s /bin/bash "$USERNAME"
pct exec "$CTID" -- bash -lc "echo '$USERNAME:$PASSWORD' | chpasswd"

log "Creating .ssh"
pct exec "$CTID" -- mkdir -p /home/"$USERNAME"/.ssh
pct exec "$CTID" -- chmod 700 /home/"$USERNAME"/.ssh

log "Copying keys"
pct push "$CTID" "$KEYFILE" /home/"$USERNAME"/.ssh/authorized_keys

log "Fix permissions"
pct exec "$CTID" -- chown -R "$USERNAME":"$USERNAME" /home/"$USERNAME"/.ssh
pct exec "$CTID" -- chmod 600 /home/"$USERNAME"/.ssh/authorized_keys

log "Sudo"
pct exec "$CTID" -- bash -lc "echo '$USERNAME ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/$USERNAME"
pct exec "$CTID" -- chmod 440 /etc/sudoers.d/"$USERNAME"

log "Disabling root SSH"
pct exec "$CTID" -- bash -lc "sed -i 's/^#\\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config"
pct exec "$CTID" -- systemctl reload ssh || pct exec "$CTID" -- systemctl reload sshd || true

log "Done"

```

`chmod +x /usr/local/bin/ct-bootstrap.sh`



Create Service scrips to tracke CTS

`mkdir /var/lib/ct-bootstrap`

`nano /usr/local/bin/ct-watcher.sh`

```bash
#!/bin/bash
set -euo pipefail

log() {
  echo "$[$(date +'%F %T')] $*" | tee -a /var/log/ct-watcher.log | logger -t ct-watcher
}

while true; do
  for CONF in /etc/pve/lxc/*.conf; do
    CTID=$(basename "$CONF" .conf)
    MARK="/var/lib/ct-bootstrap/$CTID.done"

    [ -f "$MARK" ] && continue

    if pct status "$CTID" 2>/dev/null | grep -q running; then
      log "CT $CTID running, bootstrap started"
      if /usr/local/bin/ct-bootstrap.sh "$CTID"; then
        touch "$MARK"
        log "CT $CTID bootstrap done"
      else
        log "CT $CTID bootstrap failed"
      fi
    fi
  done

  sleep 20
done

```
`chmod +x /usr/local/bin/ct-watcher.sh`

Write Service

`nano /etc/systemd/system/ct-watcher.service`


```bash
[Unit]
Description=Proxmox CT watcher

[Service]
ExecStart=/usr/local/bin/ct-watcher.sh
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Reload daemon and enable ct-watcher

```bash
systemctl daemon-reload
systemctl enable ct-watcher.service
```