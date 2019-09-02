# Reversing MD5 hashes

This repo provides a couple simple examples of using password cracking tools like [John the Ripper](https://www.openwall.com/john/) and [Hashcat](https://hashcat.net/hashcat/) to reverse MD5 hashes of common passwords.

Both tools can be used for other hash types and to reverse more complex password-hashing. These are just some simple examples for demonstration purposes.

You can likely just search for common MD5 hashes, e.g. on Google. For example, I searched for the following MD5 hash:

    e10adc3949ba59abbe56e057f20f883e

Google returned https://hashtoolkit.com/reverse-hash/e10adc3949ba59abbe56e057f20f883e for the second result. The site indicates the search term is the result of hashing the string '123456'.

## Examples

I used 2 username and password pairs:

* username sfalken, password JOSHUA
* username mjr, password the0toky

One note, the `echo` command automatically includes a newline character, which would also be hashed by `md5sum`. Use `echo -n` to suppress the trailing newline:

    echo -n JOSHUA | md5sum >hashes
    echo -n the0toky | md5sum >>hashes

John the Ripper expects a `passwd` file, which requires the username string and the password string, separated by a colon (":"):

    sfalken:c9f41e6d2b503216e772b8e5fd00adfe
    mjr:5b6d08dea554d4bef693a1593dec6300

You can create this file using the `paste` command:

    paste -d: usernames hashes >passwd

### John the Ripper

We need to specify that the password format is MD5:

    john --format=Raw-md5 passwd

It shouldn't take long to crack `JOSHUA`, which is part of John the Ripper's default word list:

    /usr/share/john/password.lst

(Note that `joshua` is part of the default word list, not `JOSHUA`, but apparently `john` tries uppercase and possibly mixed-case variations as well.)

`the0toky`, on the other hand, is not in the default word list. After exhausting the word list, John the Ripper will attempt to brute-force the password. Depending on your machine, it could take a long time to reverse. (I waited several hours and eventually I gave up.) You can stop processing with ctrl-c or `q`.

To display the cracked password, you'll need the following command:

    john --show --format=Raw-MD5 passwd

### Hashcat

You can re-use John the Ripper's list of common passwords with Hashcat. There are many other password lists available online:

    hashcat -m 0 --show hashes /usr/share/john/password.lst

* `-m 0` specifies MD5 as the hash type (see `man hashcat`)
* `--show` indicates that the cracked passwords should be displayed

Hashcat can also use the `passwd` file, if we tell it to ignore the username:

    hashcat -m 0 --username --show passwd /usr/share/john/password.lst

## Why?

1. Demonstrates that MD5 hashes are not sufficient for storing passwords. Use PBKDF2 or bcrypt instead (newer KDFs, such as scrypt, may become standard in time).
2. Demonstrates how to use tools like `john` and `hashcat` with wordlists to crack common passwords.
