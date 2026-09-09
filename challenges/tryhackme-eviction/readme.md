# Set up

A browser, nothing else is required. A new tab will open, and all you need is there.

# Goal

Protect an organization from a possible APT28 group attack.

# Walkthrough

### Technique

Either use the link provided, or go to MITRE ATT&CK, find the group, and open the view of the Navigator Layer. There take a look at what's used for initial access, only one technique fits into the answer field.

### Reconnaissance

Might I just say that the spelling of the word reconnaissance is overly complicated? I get that it's French in origin, but simplify it already. Anyway...

Consult the Navigator and look at resource development. That will answer the question, what kind of accounts the group might compromise.

### Initial access

There are 2 techniques of user execution to find. Look at the execution section, and find what type of actions a user could perform to help them get access. Format should be `a and b`.

### Execution

We're looking for interpreters. Think which ones are selected as most likely to be used.

### Persistence

Basic knowledge of how to make sure, that the script stays active every time. Pick what part of registry would normally be used.

### Defence evasion

Look at the stealth section, APT28 seems to like to use a specific binary for proxy execution. It's not an uncommon one either.

### Snooping

An application has been left behind, it's tcpdump, clue is very much in the name of what it does.

### Pivoting

Probably one of the most exploited remote services in Windows. Look at the lateral movement section.

### Collection

In the collection section look for repositories, many companies use it to share data.

### Control

Look into command and control, what proxy are they most likely to use?

# Last words

Very straight forward challenge, just read the Navigator and you'll have all the answers.

Cheers,
Mikołaj.