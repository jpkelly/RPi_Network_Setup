# RPI NETWORKING SET UP 
from https://www.raspberrypi.org/forums/viewtopic.php?t=191453
_______________________________________________________________________

## INSTALL dnsmasq hostapd
```
sudo apt-get install dnsmasq hostapd
```

## ADD TO /etc/dhcpcd.conf

```
# It is possible to fall back to a static IP if DHCP fails:
# define static profile
profile static_eth0
static ip_address=192.168.0.199/24
static routers=192.168.0.1
static domain_name_servers=8.8.8.8


# fallback to static profile on eth0
interface eth0
fallback static_eth0

interface wlan0
static ip_address=192.168.4.1/24
static routers=192.168.4.1
static domain_name_servers=8.8.8.8
```


## PUT IN /etc/hostapd/hostapd.conf
```
# This is the name of the WiFi interface
interface=wlan0

# Use the nl80211 driver with the brcmfmac driver
driver=nl80211

# This is the name of the network
ssid=ssidname-AP

# Use the 2.4GHz band
hw_mode=g

# Use channel 4
channel=4

# Enable 802.11n
ieee80211n=1

# Enable WMM
wmm_enabled=1

# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]

# Accept all MAC addresses
macaddr_acl=0

# Use WPA authentication
auth_algs=1

# Require clients to know the network name
ignore_broadcast_ssid=0

# Use WPA2
wpa=2

# Use a pre-shared key
wpa_key_mgmt=WPA-PSK

# The network passphrase
wpa_passphrase=myverysecurepassphrase

# Use AES, instead of TKIP
rsn_pairwise=CCMP
```


## PUT THE FOLLOWING IN  /etc/default/hostapd
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## PUT (ONLY) THE FOLLOWING IN  /etc/dnsmasq.conf
```
interface=wlan0      # Use the require wireless interface - usually wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```


## RUN
```
sudo systemctl unmask hostapd.service
sudo systemctl enable hostapd.service
```


## RUN THE  FOLLOWING
```
sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

sudo rfkill unblock 0
```


## UNCOMMENT THE FOLLOWING IN /etc/sysctl.conf
```
#net.ipv4.ip_forward=1
```

## RUN
```
iptables -t nat -A POSTROUTING -j MASQUERADE

sudo service dnsmasq restart
sudo service hostapd restart
```


## INSTALL
```
sudo apt install hostapd bridge-utils
```


## CHECK IF WORKING
```
sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf

sudo service dnsmasq status
sudo service hostapd status
```

_______________________________________________________________________

# OTHER SETUP

## Put following in .vimrc:
```
let &t_SI .= "\<Esc>[?2004h"
let &t_EI .= "\<Esc>[?2004l"

inoremap <special> <expr> <Esc>[200~ XTermPasteBegin()

function! XTermPasteBegin()
  set pastetoggle=<Esc>[201~
  set paste
  return ""
endfunction
```
From
https://coderwall.com/p/if9mda/automatically-set-paste-mode-in-vim-when-pasting-in-insert-mode
