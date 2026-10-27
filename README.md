# bandit-writeups
My notes from OverTheWire's Bandit, a wargame that teaches Linux and command-line fundamentals. I'm working through it as part of my path to cloud security engineering.
Each section covers the goal, the commands I used, and what the level taught me.
Progress: Levels 0 to 15

Level 0: Logging in with SSH
Goal: Connect to the game server over SSH.
Commands: ssh -p 2220 bandit0@bandit.labs.overthewire.org
What I learned:
How an SSH command works and how it is used to access a server remotely
SSH required a username, host and port. The -p flag sets a non-standard port.
If you leave out the username, SSH uses your local username, which caused my first "Permission denied".
Read every line the server prints. The banner told me exactly what was wrong.
Level 0 → 1: Reading a file
Goal: Read the password from a file called readme in the home directory.
Commands: ls -la, cat readme
What I learned:
ls -la shows hidden files (-a) and permissions, owner and size (-l).
How to read a permission string like -rw-r-----: file type, then owner, group and everyone else.
Work out which permission slot applies to you by checking the owner and group names.

Level 1 → 2: A file named -
Goal: Read a file whose name is a single dash.
Commands: cat ./-
What I learned:
To many commands, a lone - means "read from the keyboard," so cat - just waits for input.
./ means "this directory," so ./- points to the actual file.

Level 2 → 3: Spaces and dashes in a filename
Goal: Read a file named --spaces in this filename--.
Commands: cat ./--spaces\ in\ this\ filename--
What I learned:
The shell splits words at spaces, so a name with spaces needs quotes or a backslash before each space.
A name starting with - looks like an option, so it needs a ./ path in front.
Tab completion fills in awkward names correctly and avoids typos.

Level 3 → 4: A hidden file
Goal: Find a hidden file in the inhere directory.
Commands: cd inhere, ls -la, cat
What I learned:
Files starting with . are hidden from plain ls. The -a flag shows them.

Level 4 → 5: The only human-readable file
Goal: Find the one human-readable file among several in inhere.
Commands: file ./*
What I learned:
file tells you what kind of data a file contains, regardless of its name.
Don't cat binary files. The output is garbage and can scramble the terminal (reset fixes it).

Level 5 → 6: Searching by size
Goal: Find a file of a specific size hidden among 20 subdirectories.
Commands: find . -size 1033c
What I learned:
Directories show 4096 bytes in ls -la. That's the folder's own size, not its contents.
find searches a directory and everything below it. . means start here.
In find, the c suffix means bytes. Without it, the size is counted in 512-byte blocks.

Level 6 → 7: Searching the whole server
Goal: Find a file somewhere on the server, matching a given owner, group and size.
Commands: find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
What I learned:
find can stack conditions, and a file must match all of them.
/ searches the whole system. Searching everything produces many "Permission denied" errors.
2>/dev/null sends error messages to a black hole, leaving only real results.
Absolute paths start with /; relative paths start from where you are. dev/null and /dev/null are different things.

Level 7 → 8: Searching inside a file
Goal: Find the password next to the word "millionth" in data.txt.
Commands: grep millionth data.txt
What I learned:
grep searches inside files for text. find searches for files.
grep needs both a pattern and a file. Without a file, it waits for keyboard input.

Level 8 → 9: The only unique line
Goal: Find the one line in data.txt that appears only once.
Commands: sort data.txt | uniq -u
What I learned:
The pipe | sends one command's output into the next command.
uniq only compares neighboring lines, so the file has to be sorted first.
uniq -c counts occurrences, and uniq -u shows only lines that appear once.

Level 9 → 10: Readable text in a binary file
Goal: Find the password among the few readable strings in a binary file, next to several = characters.
Commands: strings data.txt | grep "=="
What I learned:
strings pulls readable text out of binary data.
Piping into grep filters a wall of output down to just what matters.

Level 10 → 11: Base64
Goal: Decode a Base64-encoded file.
Commands: base64 -d data.txt
What I learned:
Encoding is not encryption. Base64 hides data from a quick glance, but anyone can reverse it with no key.

Level 11 → 12: ROT13
Goal: Decode text where every letter was rotated 13 places.
Commands: cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
What I learned:
tr swaps each character in the first set for the one in the same position in the second set.
ROT13 is its own reverse, since rotating 13 twice brings you back to the start.
Test a command on a known input first (echo "Uryyb" should become Hello).

Level 12 → 13: Many layers of compression
Goal: Unpack a hex dump of a file that has been compressed many times over.
Commands: mktemp -d, cp ~/data.txt ., xxd -r, file, mv, gzip -d, bzip2 -d, tar -xf
What I learned:
Make a scratch directory in /tmp when the home directory isn't writable.
cp needs a source and a destination.
The loop: run file, rename to match the type, unpack, then ls to see what came out.
gzip -d needs a .gz extension. In tar -xf, the f goes last because the file name follows it.

Level 13 → 14: Logging in with an SSH key
Goal: Use a private SSH key to log in as the next user.
Commands:
scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .
chmod 600 sshkey.private
ssh -i sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org
What I learned:
An SSH key pair has a private key (kept secret) and a public key (stored on the server).
When logging in, the private key proves who you are without ever leaving your machine.
SSH refuses private keys that other users can read, so permissions must be 600.
scp copies files over SSH and uses a capital -P for the port.
A leaked private key means anyone can become that user. Delete keys when you no longer need them.

Level 14 → 15: Talking to a port
Goal: Send the current password to a service on port 30000.
Commands: nc localhost 30000
What I learned:
Programs that accept network connections listen on numbered ports.
localhost means the machine you're on.
nc (netcat) opens a raw connection to a port: what you type is sent, and the reply is printed.

