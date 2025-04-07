## Cracking with Reaver and Bully Demonstration[](#cracking-with-reaver-and-bully-demonstration)

In this section, we’ll walk through the process of cracking WPA/WPA2 PINs using Reaver and the Pixie Dust attack. This technique is useful for CTF challenges involving wireless security and cracking.

You can view a demonstration of the process on [YouTube](https://www.youtube.com/watch?v=_d7EHujPt-U).

### Prerequisites[](#prerequisites)

Before we begin, ensure you have the necessary tools installed and a compatible wireless card. You should also have a wireless network that supports WPA/WPA2 encryption.

### Step-by-Step Process[](#step-by-step-process)

#### 1. **Install Reaver**[](#id-1.-install-reaver)

Reaver is a tool used to crack WPA/WPA2 by exploiting the WPS (Wi-Fi Protected Setup) vulnerability. You can install it by running the following command:

```bash
sudo apt-get install reaver
```

#### 2. **Start Interface in Monitor Mode and Kill Unwanted Services**[](#id-2.-start-interface-in-monitor-mode-and-kill-unwanted-services)

In order to interact with the wireless network and capture packets, you need to switch your wireless card to monitor mode. Additionally, some processes may interfere with wireless scanning, so we’ll stop any unnecessary services.

```bash
airmon-ng check kill && airmon-ng start wlan0
```

- `airmon-ng check kill`: Stops any processes that could interfere with wireless card operations.
- `airmon-ng start wlan0`: Puts your wireless card into monitor mode (replace `wlan0` with the correct interface name for your system).

#### 3. **Enumerate Wireless Access Points Using Wash**[](#id-3.-enumerate-wireless-access-points-using-wash)

Now that the interface is in monitor mode, you can use `wash` to enumerate the wireless points that support WPS. This will help you identify the target BSSID (MAC address) for the attack.

Enumerate Wireless Point using wash:

```bash
wash -i <monitor wireless card>
```

- Replace `<monitor wireless card>` with your actual monitor mode interface name (e.g., `wlan0mon`).
- `wash` will display a list of available wireless networks, showing details such as whether WPS is enabled.

#### 4. **Execute the Pixie Dust Attack**[](#id-4.-execute-the-pixie-dust-attack)

Once you have identified the target BSSID, use Reaver to execute the Pixie Dust attack. The Pixie Dust attack significantly speeds up the WPS PIN cracking process by leveraging public key cryptography.

```bash
reaver -i <monitor wireless card> -b <bssid> -vv -K 1
```

- Replace `<monitor wireless card>` with your interface in monitor mode (e.g., `wlan0mon`).
- Replace `<bssid>` with the target wireless network's BSSID.
- `-vv`: Increases verbosity to show detailed information during the attack.
- `-K 1`: Enables the Pixie Dust attack, which improves cracking speed.
#### Notes:[](#notes)

- **Target Selection**: Ensure that the target wireless access point supports WPS and that the attack is feasible. The Pixie Dust attack is effective on certain networks that have weak or predictable keys.

- **Time Considerations**: This attack can take time depending on the strength of the WPA/WPA2 PIN and the hardware you're using.

### Summary[](#summary)

By following these steps, you'll be able to perform a Pixie Dust attack using Reaver on WPA/WPA2 networks that have WPS enabled. This technique is useful in CTF challenges focused on wireless security, allowing you to crack WPS PINs efficiently.

For more advanced attacks or techniques, check out the [YouTube demonstration](https://www.youtube.com/watch?v=_d7EHujPt-U).
