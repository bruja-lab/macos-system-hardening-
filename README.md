### Captain’s Log: Bashful System Hardening

**Date:** 2026-05-23

**Objective:** Threat hunting, perimeter hardening, and removal of unauthorized/background surveillance.

|**Command**|**Objective**|**Result**|
|---|---|---|
|`launchctl list \| grep -v com.apple`|Identify 3rd-party startup agents.|Clean (No malicious persistent agents).|
|`lsof -iTCP -sTCP:ESTABLISHED`|List active network connections.|Verified legitimate system traffic only.|
|`sudo lsof -iTCP -sTCP:LISTEN`|Audit open listening "backdoor" ports.|No unauthorized remote listeners found.|
|`top -u`|Monitor live CPU/Process heartbeat.|Identified high-load system processes.|
|`sudo networksetup -setairportpower en0 off`|Force hardware disable of Wi-Fi antenna.|**SUCCESS:** Antenna powered down.|
|`networksetup -listallhardwareports`|Map all physical/logical network interfaces.|Identified all internal/virtual ports.|
|`ifconfig \| grep -E "status\|flags"`|Verify active network interface status.|**SUCCESS:** Confirmed `en4` as sole gateway.|
|`networksetup -getairportpower en0`|Verify power state of Wi-Fi.|**Confirmed:** "Off".|

### Command Meanings & Forensic Definitions

- **`launchctl`**: The master service manager for macOS. It controls all background daemons and processes.
    
- **`lsof` (List Open Files)**: In Unix, "everything is a file." This command allows you to see every program that has a network "file" (port) open.
    
- **`grep`**: A filtering tool. We used it to search for specific strings (like `LISTEN` or `status`) inside the massive output of other commands.
    
- **`sudo`**: "SuperUser DO." This grants you administrative powers to interact with hardware and system files. **Always use this with caution.**
    
- **`networksetup`**: The command-line equivalent of the Network settings in your System Preferences. It is the most powerful tool for controlling how your Mac talks to the outside world.
    
- **`ifconfig`**: A classic networking tool used to configure and view the status of network interfaces (the "doors" we mapped).
    
- **`mtu`**: Maximum Transmission Unit. This is the size of the largest data packet a port can handle. 1500 is the standard for Ethernet.
    
- **`en0`, `en4`, etc.**: These are the device labels for your network interfaces. `en` stands for "Ethernet" (even for Wi-Fi), and the number is the specific index assigned by the kernel.
    
- **`utun`**: User Tunnel. These are virtual interfaces created by software like Proton VPN to manage encrypted traffic flows.
    

### Next Tactical Step: LuLu

You have a clean, audited, and hardened baseline. Your system has no active Wi-Fi, no listening backdoors, and you know exactly which port (`en4`) is your only path to the internet.

--

Captain’s Log: LuLu Reverse-Firewall Deployment

**Date:** 2026-05-23 **Objective:** Establish a Zero-Trust Host Intrusion Prevention System (HIPS) to monitor, intercept, and authorize all outgoing network traffic.

|Phase|Action / Command|Forensic Result & Meaning|
|---|---|---|
|**1. Procurement**|Downloaded `LuLu` via official Objective-See source.|Acquired cryptographically signed, open-source firewall binary.|
|**2. Kernel Authorization**|Bypassed macOS restrictions in **Privacy & Security**.|Granted LuLu `System Extension` privileges to intercept packets at the OS network layer.|
|**3. Baseline Config**|Checked: "Allow Apple Programs".|Auto-whitelisted native macOS daemons to prevent system crashes and extreme alert fatigue.|
|**4. Zero-Trust Config**|**Unchecked:** "Allow Already Installed Applications".|Revoked implicit trust for all currently installed software. Forced manual authorization for every existing app.|
|**5. DNS Config**|Checked: "Allow DNS Traffic (Port 53)".|Permitted raw IP lookups. _Note: Prevents firewall gridlock while still blocking unauthorized data exfiltration to those IPs._|
|**6. Scope Definition**|Authorized Proton VPN via **"Process"** scope (not Remote Endpoint).|Allowed the verified VPN application to dynamically rotate through global secure servers without breaking the tunnel.|
|**7. Stress Test**|Executed full system reboot.|Purged all existing network sockets; forced all background agents to rebuild connections into LuLu's active net. **Result:** Clean boot, zero unauthorized tracking daemons detected.|

