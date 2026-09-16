# Set up

A browser, nothing else is required. A new tab will open, and all you need is there.

# Goal

Analyze the logs and stop an attack.

# Flags

### Connection

Start the machine, open the browser, and open Splunk. You can go to `127.0.0.1:8000`, or start your own VPN, and open it outside in your own browser. Then go to search, type in `index=network_logs` and pick full possible time range.

Once you see the events, best way is to filter by `log_type=ids`. You'll have just one event and the IP is there. It will provide you also with the outbound connection.

### Impersonation

Now switch the `log_type` and look for `arp`. Event with `sender_ip` set to the IP you want, will also have the `sender_mac`.

For the user agent, you can add `extracted_host=jira` to the search, and then look at `event.agent` from the fields. One has a very suspicious name.

### Deep waters

Open the `.pcap` file from the `network_traffic` folder on the virtual machine's desktop. Filter for `arp` and count lines one by one, or read them from the status bar on the bottom.

Filter for `http.request.method == "POST"`, it should find just one event. Find the encoded form data and copy as a C string.

For the domain, just remove the filter, stay on the same packet, and start scrolling down. Soon you'll see calls to a suspicious looking domain. The type of calls made, will also answer what was used for exfiltration.

# Last words

A chance to work with Splunk and Wireshark is always welcomed.

It's an easy room, and I don't think it'll pose too much of a challenge for anyone.

Cheers,
Mikołaj.