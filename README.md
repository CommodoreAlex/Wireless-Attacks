# Wireless Penetration Testing Guide

This repository provides tools and guidance for testing the security of wireless networks, focusing on common Wi-Fi protocols and vulnerabilities. 

## Table of Contents

1. [WPS (Wi-Fi Protected Setup)](wps.md)
2. [WEP (Wired Equivalent Privacy)](wep.md)
3. [WPA-PSK (Wi-Fi Protected Access Pre-Shared Key)](wpa_psk.md)
5. [WPA2/3 AP-less](apless.md)
6.  [WPA2/3 Enterprise](mgt.md)

---

## WEP

**WEP** is an outdated and insecure protocol with major flaws, including weak initialization vectors and weak RC4 encryption.

- **Vulnerabilities**: Weak IVs, RC4 weakness.
- **Pen Testing**: Use **Aircrack-ng** for cracking WEP keys through statistical analysis.

---

## WPA-PSK

**WPA-PSK** uses a shared key for authentication, making it vulnerable to brute-force and dictionary attacks if the passphrase is weak.

- **Vulnerabilities**: Weak passphrase, no robust authentication.
- **Pen Testing**: Capture WPA handshakes using **airodump-ng** and crack with **Hashcat** or **John the Ripper**.

---

## WPS

**WPS** is a protocol for simplifying Wi-Fi connections but is vulnerable to PIN brute-force attacks.

- **Vulnerabilities**: Weak 8-digit PIN.
- **Pen Testing**: Use **Reaver** or **Bully** to brute-force the WPS PIN.

---

## WPA2/3 Enterprise

**WPA2/3 Enterprise** provides stronger security with RADIUS authentication and AES encryption, used in corporate environments.

- **Vulnerabilities**: Misconfigured RADIUS, weak EAP methods.
- **Pen Testing**: Capture and analyze EAP handshakes with **Wireshark**; exploit RADIUS misconfigurations.

---

## WPA2/3 AP-less

**WPA2/3 AP-less** mode allows devices to connect directly without an access point, used in mesh networks or peer-to-peer communications.

- **Pen Testing**: Discover AP-less networks with **Kismet**; exploit session hijacking techniques.

---

## Tools and Resources

- **Aircrack-ng**: For cracking WEP/WPA-PSK.
- **Reaver/Bully**: For exploiting WPS.
- **Wireshark**: For capturing and analyzing network traffic, especially EAP handshakes.
- **Hashcat/John the Ripper**: For cracking passwords.

---

## Contributing

Contributions are welcome! Fork the repo, submit a pull request, or open an issue for suggestions or bug fixes.

**Disclaimer**: Always obtain permission before performing any penetration testing. Unauthorized testing is illegal.

