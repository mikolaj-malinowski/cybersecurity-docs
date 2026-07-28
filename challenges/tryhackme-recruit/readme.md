# Disclaimer

This write up is intended to document my first no-help challenge. I'm only using a search engine and various documentations. To make things harder, I'm not using AI for help or searching.

Initially, I wanted to simply record my steps. But with all of the going back and forth, this became more of a proof that it's okay to get lost, fail, and change your mind 23 times. Nobody starts full of experience and the beginnings won't look pretty.

Hopefully, someone will find encouragement, after seeing how poorly I dealt with this challenge.

# Set up

Using my own VM hosted Kali Linux, a VPN connection to TryHackMe, coffee, glasses, and patience.

# Reconnaissance

### Opening the page

After opening the page I saw a form, so I checked if the form is susceptible to SQL injections, used different types of apostrophes, no errors were given back. Probably typical injections won't work. Just to make it easier when testing, I removed `required` for the password via the browser inspector.

Looking around what else is there, found a page about API. Says you can fetch CVs with `file.php?cv=<URL>`. Sounds incredibly exposed. Tried with a random input, error says only local files are allowed.

### Deciding on an approach

Here I was wondering if I can break through the form, or can I install a shell via the file upload. Decided to gather more info. 

A scan with Nmap showed this:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Added a Gobuster scan for common directories to run in the background. The result was:

```
.                    (Status: 200) [Size: 1417]
assets               (Status: 301) [Size: 317] [--> http://10.112.189.245/assets/]                                        
javascript           (Status: 301) [Size: 321] [--> http://10.112.189.245/javascript/]                                    
mail                 (Status: 301) [Size: 315] [--> http://10.112.189.245/mail/]                                          
phpmyadmin           (Status: 301) [Size: 321] [--> http://10.112.189.245/phpmyadmin/]     
```

### Checking for exploits

While Gobuster was running I started checking when was Apache httpd 2.4.41 released - it was 13-08-2019, based on https://www.apachelounge.com/viewtopic.php?p=38419 from https://www.apachelounge.com/Changelog-2.4.html. It's probably riddled with holes. As expected, the list is long, focusing on most critical issues, unfortunately I couldn't find anything fitting.

### Check on discovered paths from the scan

Folder `assets` is fully readable, `mail` folder as well. There's a log file in there with a copy of a mail. It has an interesting part:

>- HR login credentials (username: XXX) are currently stored in the application configuration file (config.php) for ease of access during the initial rollout phase.
>- Administrator credentials are NOT stored in the application files and are securely maintained within the backend database."

Goal now is to read the `config.php` file, this will grant some level of access.

Doing an extra check if OpenSSH is easily exploitable. According to https://www.openssh.org/txt/release-8.2 it was released on 14-02-2020. Which means both Apache and OpenSSH are going to be most likely highly exploitable. Both are at least 6 years old.

Checking one more folder - `phpMyAdmin`, most likely it will be old too. Logins `admin:admin` or `root:root` didn't work. Just `root` won't work because of a set policy to not allow logins without a password. Front page doesn't tell the version, but page source references 4.9.5deb2. Site https://www.phpmyadmin.net/files/4.9.5/ says it's from 21-03-2020, which fits with other versions. I know that admin credentials are stored in a database, so this will be the entry point later.

# Planning

### What I know so far

I know that OpenSSH, Apache and phpMyAdmin are all at least 6 years old.

Also know that there is an `hr` and an `admin` account, `hr` has the password stored in `config.php`, and `admin` inside a database.

Lastly, there is an API that can access locally stored files.

I had a few questions:

1. Can I access `config.php` via the API?
2. Can Nmap or Metasploit find something more?
3. Can I exploit these versions of Apache or OpenSSH?

### Finding the answers

1. Tried a few approaches with `file.php`, none worked. I checked where the file is and seems to be in the top folder next to `file.php`, I know that because just replacing it with `config.php` worked and showed an empty page, wrong paths give an error.
2. I decided to run Nmap again but this time with `--script vulners`, looking for exploits with highest score. Ran `msfconsole` and looked what's available. Tried a few exploits but none worked.
3. They have to be hackable. Looked at a long list of exploits for OpenSSH and Apache again, most required some kind of initial access first. Did one more check with network inspector in the browser, just in case if there is some type of a redirect before reaching the API but there wasn't. Then I remembered to do first things first again and decided to grab a banner, or rather MOTD, from SSH. Only extra information was that `This session may be vulnerable to "store now, decrypt later" attacks.`

### Going back to scanning

I'm lost again so decided to gather even more information. Started a scan for common names with focus on `.php` extension. The results were:
   
   ```.htaccess.php        (Status: 403) [Size: 279]
.hta                 (Status: 403) [Size: 279]
.hta.php             (Status: 403) [Size: 279]
.htaccess            (Status: 403) [Size: 279]
.htpasswd            (Status: 403) [Size: 279]
.htpasswd.php        (Status: 403) [Size: 279]
api.php              (Status: 200) [Size: 4151]
assets               (Status: 301) [Size: 317] [--> http://10.112.188.215/assets/]   
config.php           (Status: 200) [Size: 0]
dashboard.php        (Status: 302) [Size: 457] [--> index.php]                       
file.php             (Status: 200) [Size: 20]
footer.php           (Status: 200) [Size: 289]
header.php           (Status: 200) [Size: 457]
index.php            (Status: 200) [Size: 1417]
index.php            (Status: 200) [Size: 1417]
javascript           (Status: 301) [Size: 321] [--> http://10.112.188.215/javascript/]                      
logout.php           (Status: 302) [Size: 0] [--> index.php]                         
mail                 (Status: 301) [Size: 315] [--> http://10.112.188.215/mail/]     
phpmyadmin           (Status: 301) [Size: 321] [--> http://10.112.188.215/phpmyadmin/]                      
server-status        (Status: 403) [Size: 279]
sitemap.xml          (Status: 200) [Size: 1710]
   ```

Only `sitemap.xml` proves useful. Should've checked if it exists in the very beginning.

The API keeps saying that it needs a local file and that it accepts http and https, but keeps refusing everything.

### Taking another step back

Still nothing, reading the actual task again, maybe hint is in the wording.

They ask to me to map, abuse exposed functionality and exploit vulnerabilities. Map would probably refer to Nmap and Gobuster, exposed functionality would be the API, exploiting vulnerabilities could be anything, but since software is over 6 years old they probably mean actual exploits.

Mapping is done, both of the structure and exposed services. I had no luck with API. I must focus on known exploits.

# First flag

### SQL injections

To make sure I didn't miss anything, I ran `sqlmap` with `--data` option to see if the form is actually vulnerable to SQL injections. Also tried it for the `file.php?cv=`. Had no luck with either.

### Metasploit

I ran a couple of scanners and possible exploits from Metasploit but none of them worked.

### API

Tried a few things and when using `cv=file:///` I got a new error, saying `Access denied`. When using `cv=file://` I got no error at all. I now strongly believe that this is what they wanted me to do from the beginning.

Decided to start running everything through Burp to see if there's anything I'm missing, set the scope and started to look at everything again.

I've been trying to make it load a shell from my side, with no luck, but then... something unexpected. I thought to myself - "what if it's reversed?" Meaning that it loads files from its own environment. And it did, it can be used to load `config.php` and other files by using `file.php?cv=file://config.php`. I now have what I needed to log in.

```
/*
|--------------------------------------------------------------------------
| HR Credentials (Temporary – Initial Rollout Phase)
|--------------------------------------------------------------------------
| NOTE:
| These credentials are stored here temporarily for ease of access
| during the initial deployment and will be moved to the database
| in a future release.
*/

$HR_PASSWORD = 'XXX';
```

An added bonus is that now I can see the mechanism of the whole site. I have my foot in the door and should now be able to manipulate much more.

### Logging in

I've used the credentials and got the first flag for logging in. Time for the second one.

# Second flag

### Finding the way in

I know that admin account has the password stored in the database, it's stored as `$dbPassword` and comes from `/var/www/db.php`. The API will limit the access because it's set to `$allowedBase = '/var/www/html';`. I know all of this from viewing the source of `index.php` and `file.php` with the `file.php` exploit.

Judging by the name, it only checks for the base, so maybe an escape with `../` will work. Didn't notice the `realpath` function, it's preventing me from traversal.

I decided to snoop inside `dashboard.php` and saw something interesting. The search is not sanitizing the input at all. I think it can be used to access the database and read any record.

### SQL injection

I was correct, started by taking out the name of the current database. I used `' UNION SELECT 1,2,3,database(); -- `, the result was `recruit_db`.

Next, I have listed the tables inside it. Using the union method `' UNION SELECT 1,2,3,group_concat(table_name) FROM information_schema.tables WHERE table_schema='recruit_db'; -- ` I have gotten 2 tables: `candidates,users`. I can safely assume that I'm looking for `admin` and that it's in `users`. I just need to know the available columns in `users`.

This time used `' UNION SELECT 1,2,3,group_concat(column_name) FROM information_schema.columns WHERE table_name='users'; -- ` and at the end have the columns I needed: `id,password,username`. At the end, because the full list was `CURRENT_CONNECTIONS,MAX_SESSION_CONTROLLED_MEMORY,MAX_SESSION_TOTAL_MEMORY,TOTAL_CONNECTIONS,USER,id,password,username`. Time to read the actual password.

Compared to all my previous scanning and attempts at exploiting OpenSSH, using the last injection felt almost too easy. Using `' UNION SELECT 1,2,3,password FROM users WHERE username='admin'; -- ` I got the password.

Just as a last check I wanted to see if there are some easter eggs in the users table, using `' UNION SELECT 1,2,3,group_concat(username) FROM users; -- `. Alas, no, only that one account.

# Last words

I did many errors, thought this challenge was much more sophisticated than it was, spent much too long reading about different exploits and attempting to break into OpenSSH. It most likely is possible but I've reached both flags so it's time to move on.

Hope this write up made someone not give up and pull through.

Personally, I feel much more confident now using some of the tools. Not using any AI helped too.

Cheers,
Mikołaj.