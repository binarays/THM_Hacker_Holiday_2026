<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251914578&w=96&q=75"> 
</p>

# After Hours

Target:
```bash
/root/Rooms/hacker-holidays-2026/after-hours
```

## STEP 01
- go to the file directory
  ```bash
  cd /root/Rooms/hacker-holidays-2026/after-hours
  ```

- Unzip the `file` 
  ```bash
  7z x filename.7z
  ```
- Enter the Password
  ```
  Aft3rH0ursAtt4chm3ntP4ss
  ```
- Find the `forensic` folder
- Then go there
- If you use your own attack box, skip STEP01 


## STEP 02
- Lets 
  ```bash
  strings -a OBJECTS.DATA | grep -oE '[A-Za-z0-9+/]{200,}={0,2}' > b64.txt
  ```
  > This tries to find large encoded/Base64 data inside OBJECT.DATA and puts the results into b64.txt for further analysis.
  > - strings -a OBJECT.DATA → extracts readable text from the file.
  > - grep -oE '[A-Za-z0-9+/]{200,}={0,2}' → finds strings that look like Base64, at least 200 characters long.
  > - ">" → redirects the results into a file.
  > - /temp/b64.txt → stores the extracted strings there.

- Now use the `payload`
  ```bash
  while read -r x; do
      echo "$x" | base64 -d 2>/dev/null | python3 -c 'import sys,zlib; d=sys.stdin.buffer.read(); 
  try:
      print(zlib.decompress(d,-15).decode(errors="ignore"))
  except:
      pass'
  done < b64.txt
  ```
  > Base64 → decode → try DEFLATE decompression → print readable text if successful.

- Then find the `user patch`
  ```bash
  echo "<hash>" | base64 -d
  ```
  
## You will find the  **FLAGs** 🚩

# Congratulations on our Exploration 🎉
