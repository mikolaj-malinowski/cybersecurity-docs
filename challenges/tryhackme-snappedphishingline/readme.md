# Set up

A browser, nothing else is required. A new tab will open, and all you need is there.

# Goal

Analyze evidence, determine the scope, uncover the method.

# Flags

### Mail

Start the VM, open the `phish-emails` folder on the desktop and start checking the emails. The answer is the name of whom the mail was addressed to. Then check what email address was used as `from`.

Normally I wouldn't just open an attachment from an email like this, but let's assume the VM is safe and even if something would happen, then any lateral movement is blocked. With that in mind, open the attachment, right click the button, and copy the link. Strip to just the domain and that will be your answer. Next is the company, pretty simple, which company offers this product.

### Investigation

Use the link you discovered and go to the `/data` folder. Look for an archive.

>If you're having trouble with opening the site, make sure you're not using `https`, if you are, then switch to `http`.

Download the archive, open the folder in the terminal and run `sha256sum` on it.

Now go to https://www.virustotal.com/gui/home/upload, paste the hash, and look for categories aside from `phishing`. There should be 2 more, but only one is required.

On the same page, switch to `DETAILS`, then scroll down to `Bundle Info`. It will have the information.

### Blue-colored pen testing

I find it a bit funny, that this part is essentially a very basic pen testing, but done as a blue teamer. Go back to the `/data` page, open the folder in it and check the log file. Look for the user that submitted the data more than once.

Go back to your downloaded archive, extract it and analyze the `submit.php` file. It's inside the `Validation` folder. For some reason the address is hard coded twice, first as `$send`, and then in `mail()` at the end.

For the last part, you need to open a phishing link, cut everything after `/office365/` out and add the name of the file you're looking for. Once you copy it, open https://gchq.github.io/CyberChef/, add `From Base64`, and then `Reverse`.

# Last words

The only issue I had was with Firefox, or rather the file. Link within has `https` but the connection isn't actually secure, so the browser blocked it.

Cheers,
Mikołaj.