<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785087675274&w=96&q=75">
</p>

# ROOM 404

Here we have to find Byte Lotus Exposed Source Code. First, we need to identify the clues.
  1. He booked the quiet room. It's not on the floor plan.
  2. But port 8080 is wide open.
  3. First days as a guest who simply notices things: a room that isn't on the floor plan, packets that leave every night at the same hour.
  4. Night-shift developer shipped more than the website.

According to these clues, we need to launch **Directory brute-forcing (Content Delivery Attack)** to find the exposed code.

Target:

```
http://MACHINE_IP:8080
```

## STEP 01

- First, roam around the website and find what happened there.
- Open webrowser:
  ```
  http://MACHINE_IP:8080/
  ```
- Click the **reserve button**
- It returns **404 not found**

> **HINT** : The rooms it **never lists** are the ones worth finding. <br>
> - The hint tells us that there is a hidden path that is not linked anywhere on the website. This means the path exists on the web server, but users cannot reach it by clicking links on the website.<br>
> - You can inspect the website using **Developer Tools** to see if the route for the Reserve button is exposed in the client side code (HTML, JavaScript, or network requests). If the server never sends that information to the browser, you won't be able to find it from Developer Tools alone.

## STEP 02

- Use ```curl``` to identify the server technology by examining the **HTTP response** headers.
  ```bash
  curl -I http://MACHINE_IP:8080/
  ```
  > -I : The *-I* option in *curl* is used to retrieve only the HTTP response headers without downloading the webpage content.
- It returns output like this
  ```
  HTTP/1.1 200 OK
  Server: Werkzeug/3.0.1 Python/3.12.3
  ```
  > **HINT** : The night-shift developer shipped more than the website
  
  > - **200 OK** :  This status code indicates that the web server is active, the connection was successful, and the requested page or resource was delivered without errors. This is correct because we can access the Byte Lotus Hotel website through the link **http://MACHINE_IP:8080/**)
  
  > - **Server** : This shows information about the web server software and technology being used to handle the request. The Byte Lotus website is running on a Python Flask web application, and the web server technology identified is Werkzeug, which is the WSGI library used by Flask to handle HTTP requests.

  > - **Conclusion** : This is not a static website because it uses server-side processing and dynamic functionality. It is a web application.

## STEP 03

- Inspect the Website Source Code
  ```bash
  curl http://MACHINE_IP:8080/
  ```
  > This will fetch the homepage (index.html) source code.
- Then, in the source code, you can find anchor links.
  ```html
  <a href="/booking">Reserve a stay</a>
  ```
  > This is an important finding because it shows that the website has a route called /booking (http://MACHINE_IP:8080/booking). However, when we access it, the page returns "Not Found," which means the route may exist in the application code but the corresponding page or resource is not currently deployed or available on the server.

  > **HINT** : guest experience platform · build staging

  > **GUES** : The hint "build staging" means that a staging version of the application may be publicly accessible. This could be a development/testing environment that contains features or code that have not been deployed to the production website.

  > **Conclution** :
  > Developers often accidentally expose:
  > - source code
  > - backups
  > - Git repositories
  > - configuration files



  
