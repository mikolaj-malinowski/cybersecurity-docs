# Set up

A browser, nothing else is required. A new tab will open, and all you need is there.

# Goal

Protect the workstation from malware by hardening the security.

# Walkthrough

### Sample 1

Open the introduction mail and read it, a sample will be attached, leave it alone and head over to the menu on top left and open `Malware Sandbox`. Run the scan and notice the checksums, it's a very basic way to block malware, but this time it will be enough. Pick one, go to `Manage Hashes` and add accordingly.

Pro tip, go to `Special Offer: Get Rich Quick` and open the totally-not-a-scam link.

### Sample 2

Now that you have a proper soundtrack let's move on. Open the second mail, read and head to sandbox again. This time notice the address file is connecting to, you need to set up a firewall rule.

Go to `Firewall Manager`, choose `Egress` (exit/outbound) and add the details. Think what source or sources could there be, and what was the destination in the sandbox. Then decide if you should deny or allow it.

### Sample 3

Our attacker got himself a load of new IP addresses, but the provider is the same. Meaning the domain will be the same, analyze the sample to find it. Then head over to `DNS Filter` and add a rule.

### Sample 4

Finally it's getting a bit more interesting. Same scenario, read the message and analyze in the sandbox. Scroll down and see the activity, one of the changes is critical here.

Once you decide which one it is, go to `Sigma Rule Builder`. Select `Sysmon Event Logs`, then `Registry Modifications`, and put the changes to look for. There's a link a to `MITRE ATT&CK framework` if you're not sure what ID to pick.

### Sample 5

Even more interesting now, analyze the sample and read the log. Notice, that in the log, there is a set period of time, when the victim's device hails another one.

Head over to `Sigmal Rule Builder`, choose `Sysmon Event Logs`, then `Network Connections`. Think what IP and what port could be the destination, will the size ever change, what about the time period? Then just pick the right ID, and go back to the mailbox.

### Sample 6

Last sample, read the mail, read the log. You need to notice how the code works and what does it do. It creates something.

Once you're ready, go to the rule builder, choose `Sysmon Even Logs`, then `File Creation and Modification`. Add path, file name, the ID, the log will have all you need.

If all is correct, you should receive one more mail, with the last flag.

# Last words

It was a fun little challenge, didn't take me much time.

In fact it went so fast, I forgot to write this write up, so I did it again just to make sure I didn't miss anything in the documentation.

Cheers,
Mikołaj.