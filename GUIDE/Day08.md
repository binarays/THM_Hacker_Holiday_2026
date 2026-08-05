<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251667401&w=96&q=75">
</p>

# Towel on the Sunbed

In this room, you will use Burp Suite to intercept and modify HTTP requests, allowing you to manipulate the server into increasing your credits and ultimately retrieve the flag.

Target application:
```URL
http://MACHINE_IP:3000
```

## STEP 01

- Open [BURP SUITE](https://portswigger.net/burp/downloads)
- Open the Burp Suite browser
- Open the web app in the Burp Suite browser
  ```URL
  http://MACHINE_IP:3000
  ```

## STEP 02
- Then press `F12` and open the Developer Tools
- Then Go to the `Request  Tab`
- Check the `js`
- Then, you can gain an understanding of how the web application works and how the **Claim** button functions.
- Then open the `valut` route in a new tab
  ```URL
  http://MACHINE_IP:3000/valut
  ```
- Then go back to BurpSuite
- Select the `Proxy` tab
- Turn on `Intercept`
- Next, switch back to the Burp Suite browser, open the dashboard, and click the **Claim** button to generate the corresponding HTTP request.
- Then, send the request to **Repeater** in Burp Suite.
- Then, go to Burp Suite, send the request to **Repeater**, group the request, and create **30 duplicates**. Each request increases the balance by 50. After that, change the sending mode to **Parallel (Synchronized)**, send the requests, and check the updated balance.
- Then turn off the intercept in the Proxy tab
- Navigate to the Burp Suite browser, then access the **vault** route.
  ```URL
  http://MACHINE_IP:3000/valut
  ```
  
## You have found the  **FLAGs** 🚩

# Congratulations on our Exploration 🎉
