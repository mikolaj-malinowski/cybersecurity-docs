# Set up

A browser, nothing else is required. A new tab will open, and all you need is there.

# Goal

Analyze an email and determine, if it's legitimate or not.

# Flags

### Basic header

Arm yourself in patience and start the machine. The file you need is on the desktop, just open it. Number is in the subject.

Transfer Reference Number, display name, email address, and reply to, are all in the basic header. They should be immediately visible.

### Source

Find `View` in the top menu, choose `Message Source`. To find the IP look into the received header, where it mentions `helo=mutawamarine.com`.

For the owner of the IP, I ran it through https://talosintelligence.com/reputation_center.

For the SPF I used https://mxtoolbox.com/SuperTool.aspx and picked `SPF Record Lookup`. Then changed it to `DMARC Lookup`.

### Attachment

Check the source again, look for `Content-Disposition: attachment;`.

Then download the file, open your console, find the file, and run `sha256sum` on it to get the hash.

Open https://www.virustotal.com/gui/home/search, input the hash. You should see the size and analysis.

# Last words

Pretty simple analysis, nothing more I can say about it. Teaches you the basics of investigating a suspicious email.

Cheers,
Mikołaj.