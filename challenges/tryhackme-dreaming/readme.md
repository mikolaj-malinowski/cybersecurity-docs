# Set up

Using my virtual machine with Kali Linux, and a VPN connection to TryHackMe.

# Goal

Unsure, they ask me to help Sandman restore his kingdom. There are 3 flags to get in total.

# Reconnaissance

The address opens a default Apache2 Ubuntu page. Running a regular nmap didn't show much.

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

My best guess is that they want us to get access through Apache. Going to look for directories with `gobuster`.

# Flags

### Lucien

Scanning for directories revealed a folder named `app`, it's powered by pluck and it seems to be our starting point.

The URL has `file=` format, so most likely they want me to exploit it to open a file. I tinkered a little with it, but all I got was a message about a hacking attempt and that it's been logged.

The page contains a link to an admin console. There we're greeted with a page that only asks for a password.

Running `ffuf` against it, revealed a password, any simple wordlist will do. Just one warning, if you relog too fast, there will be a 5 minute pause before you're allowed back in.

I used `searchsploit` to look for anything related to `pluck 4.7.13` and found one. After downloading with `-m` option I ran it like so:

```
python3 scriptpath ip port password pluckpath
```

Now that I've got a shell installed, I can start looking around. I've taken a look at `/etc/passwd` and `/etc/group`. Nothing really stands out.

After a tone of digging and searching I found two scripts in `/opt`. I'll omit how long it took me, but "I'm in", as Lucien. Went back to the home folder and grabbed the flag.

### Death

I've logged as `lucien` with ssh. Checked what I can do with `sudo -l`, and I can run a script from `death`. Keep in mind that the full path needs to be used, shorting is giving a no access error.

```
sudo -u death /usr/bin/python3 /home/death/getDreams.py
```

This resulted in a few dreams being listed.

>Alice + Flying in the sky
>
>Bob + Exploring ancient ruins
>
>Carol + Becoming a successful entrepreneur
>
>Dave + Becoming a professional musician

The only problem is I can only run it, without any changes or input. However, from reading the code at `/opt`, I know that it takes input from the database.

Once again I was unsure how to proceed further, after a lot of looking around I found out that there was a command hidden in the bash history, in Lucien's home folder.

```
mysql -u lucien -p XXX
```

I know from reading the script that the database is called `library`, and the table is `dreams`. The script takes the value of `dreamer` and `dream` and runs it through `subprocess.check_output` with no sanitization.

```
INSERT INTO dreams (dreamer,dream) VALUES ('Lucien', '$(cp /bin/bash /tmp/death-bd && chmod +s /tmp/death-bd)');
```

Now I was able to run the shell and become `death`.

```
/tmp/death-bd -p
```

Grabbing the flag.

### Morpheus

...ya, I had to get help as well. I will not mention what I think about this part.

There is a recurring process, that keeps running `restore.py` from the `morpheus` folder. You won't find it with `ps -aux`, unless you drank a can of liquid luck. Best way is to copy `pspy`, and be on the lookout for anything running as `morpheus`.

After I've done that, I went back to the `morpheus` home folder and analyzed the `restore.py` file. Important part is the library being imported.

Going to `/usr/lib/python3.8/` I found out that it's owned by `death` and has write rights. Now all I needed to do, was to put a code that will set up a shell, and let me become `morpheus`. Simplest is this:

```
echo 'import os; os.system("cp /bin/bash /tmp/morpheus-bd && chmod +s /tmp/morpheus-bd")' >> /usr/lib/python3.8/XXX
```

From there just load it up and read the flag. Simply use:

```
/tmp/morpheus-bd -p
```

# Last words

I took a part in some CTF contests, and I know they often have some type of gimmicks and puzzles, but do we really need that? As someone with no real world experience with actual live pen testing, how far off are we from the real deal?

On one hand I enjoyed the theme, on the other hand it became very frustrating at times. Specially the last step, which would have me guessing for days.

Cheers,
Mikołaj.