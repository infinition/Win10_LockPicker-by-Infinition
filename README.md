# Win10 LockPicker

A P4wnP1 A.L.O.A. payload that unlocks a locked Windows 10 session without
prior knowledge of the user's password. It does this by poisoning LLMNR/NBT-NS
traffic over a spoofed USB network adapter, capturing the NTLMv2 hash, cracking
it offline on the device itself, then replaying the recovered password through a
HID keyboard, all automatically, in one plug-in sequence.

A second script (`smbrute.sh`) provides an alternative path using SMB brute
force via Metasploit rather than hash capture.

For educational and authorised penetration testing purposes only.

---

## How it works

### Hash capture path (`Win10_LockPicker.sh`)

1. The P4wnP1 presents itself as a USB RNDIS network adapter and a HID
   keyboard simultaneously.
2. Windows automatically assigns the adapter a default route. Combined with
   WPAD poisoning and static route spoofing, this routes a portion of the
   target's network traffic through the device.
3. Responder intercepts LLMNR/NBT-NS/WPAD requests and captures NTLMv2 hashes,
   which Windows sends automatically when probing network resources.
4. The captured hash is stored in the Responder SQLite database. Once a hash
   is detected, Responder is stopped and John the Ripper runs against the hash
   file.
5. On a successful crack, the recovered plaintext password is typed into the
   locked session via the HID keyboard emulator. The script sends
   `CTRL+ALT+DEL` first to bring the credential prompt forward, then types
   the password and presses Enter.

### SMB brute force path (`smbrute.sh`)

1. Same USB gadget setup: RNDIS + HID keyboard.
2. Once the target receives a DHCP lease, the script resolves its IP and checks
   whether port 445 is open using Nmap.
3. If SMB is reachable, Metasploit's `smb_login` auxiliary module runs a
   credential brute force using the provided wordlists.
4. On a successful login, the found password is replayed via HID in the same
   way as the hash capture path.

---

## Requirements

- P4wnP1 A.L.O.A. (Raspberry Pi Zero W)
- Responder (included in P4wnP1)
- John the Ripper (for the hash capture path)
- Nmap and Metasploit (for the SMB brute force path)
- Wordlists placed at `/usr/local/P4wnP1/scripts/wordlists/`
  - `users.txt`
  - `passwords.txt`

---

## Installation

Copy the scripts to the P4wnP1 scripts directory:

```bash
cp Win10_LockPicker.sh /usr/local/P4wnP1/scripts/
cp smbrute.sh /usr/local/P4wnP1/scripts/
chmod +x /usr/local/P4wnP1/scripts/Win10_LockPicker.sh
chmod +x /usr/local/P4wnP1/scripts/smbrute.sh
```

In the P4wnP1 web interface, configure the USB gadget settings to expose RNDIS
and HID keyboard, then create a trigger that launches the chosen script when a
DHCP lease is issued.

The keyboard layout is set to French (`fr`) in both scripts. Change the `lang`
variable and the `layout()` call in the generated HID script if your target
uses a different layout.

---

## Configuration

Both scripts expose a set of variables at the top that control behaviour.

### `Win10_LockPicker.sh`

| Variable         | Default              | Description                                      |
|------------------|----------------------|--------------------------------------------------|
| `USE_RNDIS`      | `true`               | Enable RNDIS network adapter emulation           |
| `USE_HID`        | `true`               | Enable HID keyboard emulation                    |
| `lang`           | `fr`                 | Keyboard layout for the target                   |
| `IF_IP`          | `172.16.0.1`         | IP address of the P4wnP1 interface               |
| `IF_MASK`        | `255.255.255.252`    | Subnet mask                                      |
| `IF_DHCP_RANGE`  | `172.16.0.2,...`     | DHCP range offered to the target                 |
| `ROUTE_SPOOF`    | `true`               | Inject static routes to maximise hash capture    |
| `WPAD_ENTRY`     | `true`               | Advertise a WPAD proxy via DHCP                  |
| `CRACK`          | `true`               | Run John the Ripper after hash capture           |
| `LOGIN`          | `true`               | Type the recovered password via HID              |

### `smbrute.sh`

| Variable        | Default                                        | Description                       |
|-----------------|------------------------------------------------|-----------------------------------|
| `WORDLIST_DIR`  | `/usr/local/P4wnP1/scripts/wordlists`          | Directory containing wordlists    |
| `LOOTBASE`      | `/usr/local/P4wnP1/www/loot/smbrute/`          | Directory where results are saved |

---

## File layout

```
Win10_LockPicker-by-Infinition/
  Win10_LockPicker.sh    # Hash capture + crack + HID replay
  smbrute.sh             # SMB brute force + HID replay
  README.md
```

Wordlists, loot directories, and the temporary HID scripts generated at runtime
are excluded from version control via `.gitignore`.

---

## Based on

- [SMB Brute Force mit P4wnP1 A.L.O.A.](https://pentestit.de/smb-brute-force-mit-p4wnp1-a-l-o-a/)
- [Jackalope](https://github.com/hak5/bashbunny-payloads/tree/master/payloads/library/credentials/Jackalope) by Hak5

---

## Star History

<a href="https://www.star-history.com/?repos=infinition%2FWin10_LockPicker-by-Infinition&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=infinition/Win10_LockPicker-by-Infinition&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=infinition/Win10_LockPicker-by-Infinition&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=infinition/Win10_LockPicker-by-Infinition&type=date&legend=top-left" />
 </picture>
</a>


## Legal notice

This tool is intended for use on systems you own or have explicit written
authorisation to test. Plugging this into a machine without permission is
illegal in most jurisdictions. The author takes no responsibility for misuse.

---

## License

MIT
