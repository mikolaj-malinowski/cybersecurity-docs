# Set up

Using my virtual machine with Kali Linux, and a VPN connection to TryHackMe.

# Goal

Pentest the application, escalate the access, achieve RCE. There are 2 flags to find, first one appears after logging as `admin`, second is inside `/home/ubuntu/user.txt`.

# Reconnaissance

### Website

The IP address opens a website called `Support Operations Panel`. I looked inside the source code, network, cookies but couldn't find anything solid.

### Scanning

Nmap scan didn't reveal too much.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Scanning with Gobuster had more success and I now have a few interesting locations.

```
api.php              (Status: 302)
config.php           (Status: 200)
dashboard.php        (Status: 302)
footer.php           (Status: 200)
includes             (Status: 301)
index.php            (Status: 200)
info.php             (Status: 200)
js                   (Status: 301)
layout               (Status: 301)
logout.php           (Status: 302)
server-status        (Status: 403)
skins                (Status: 301)
```

There is `api.php` which should be exploitable, but most interesting is `info.php`. It prints out the whole server configuration, including for an example `allow_url_fopen	On`.

# Access

### Dictionaries

I tried to see if I can get different errors when trying different users and passwords, tried some SQL injections, none of them had any luck, and the error message was always the same.

I can safely assume the user will be `help@support.thm`, so what I need is a password. I tried using Burp Intruder with some basic password lists I have, without luck. As a last resort before a full brute force, I'm going to try a `rockyou` list using `ffuf`. I collected the size of the invalid response with `curl`.

>curl -d "email=help@support.thm&password=x" http://10.114.146.238 | wc -m

Then prepared a `ffuf` command.

>ffuf -w rockyou-thm.txt -u http://10.114.146.238 -X POST -d "email=help@support.thm&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -fs 2678

I got the password, I probably would have gotten it sooner, if I would figure out that `ffuf` will fail without adding `http://` to the address.

### Login

I'm greeted with a dashboard. Let's see what access I have and how far can I elevate it.

I noticed that cookie now has a new field `isITUser`. It has a value, that turned out to be an MD5 hash. I used a rainbow table and it read `false`, so I edited the cookie to have it say `true`, which is `b326b5062b2f0e69046810717534cb09` in MD5.

# Escalation

### API

After setting the value to `true`, I now have access to the API. Site allows us to query ourselves with `/user/3`, but because of Insecure Direct Object Reference I can query others too. That let me find the email address of the admin account under `/user/1`.

### Admin

I tried to trick the API into letting me update the record, with no avail. But then I noticed something, the website is pretty bare, but they did give us a button to change the color of the theme. It means it probably hides something.

From the directory scan I know there is a skins folder, inside I can see this:

```
blue.php	2026-01-20 08:16	56	 
default.php	2026-01-20 08:15	56	 
green.php	2026-01-20 08:15	56	 
red.php	2026-01-20 08:15	56	 
```

The URL to change the theme is `?skin=default`, this means that files are probably loaded directly. If correct, that would give us unlimited access.

It does, now we can preview everything. Most interesting is the configuration. Using `?skin=../config` and checking the page source we can find something called `$MASTER_PASSWORD`. Combining it with the admin email is probably the highest privilege account.

Well, password doesn't work. I was previewing `index.php` before, it showed some mechanism for logging, maybe it has a clue.

It didn't have any clues, I did find however a clue for RCE in one of the files.

### First flag

I managed to log in by not using `@` in the password. I don't know why, maybe the hint was below the form, in the way the dev used `@` in `Operations @ help@support.thm` instead of `at`.

### Second flag

Remember how I wrote that I found a clue for RCE? It even appears right next to the theme after you login as admin.

If you view the `footer.php` file, it will show you this:

```
if ($isAdmin && $_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['sys'])) {

    $selectedSys = $_POST['sys'];
    $sys = $_POST['sys'];


    if (strpos($sys, 'date') === 0) {
        $output = shell_exec($sys); 
    } else {
        $error = 'Only date command is allowed.';
    }
}
```

So, I used `curl`, passed my cookie data, prepared a command like below and grabbed the last flag.

>curl -H "Cookie: PHPSESSID=cbt95gj0lq776hc2epshffqt73; isITUser=b326b5062b2f0e69046810717534cb09" -X POST -d "sys=date | cat /home/ubuntu/user.txt" "http://10.112.170.49/dashboard.php"

# Last words

It's my second challenge on TryHackMe, I think I did better than previously. Overall it was pretty easy, should've done it much quicker. Once again, I was expecting it to be much more sophisticated.

At least I didn't try to exploit OpenSSH this time.

Cheers,
Mikołaj.