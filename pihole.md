bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/pihole.sh)"

![alt text](img/pihole_nets.png)


ufw default deny incoming
ufw deny in on eth1
ufw allow in on eth1 to any port 53 proto udp
ufw allow in on eth1 to any port 53 proto tcp
ufw allow in on eth0
ufw enable
