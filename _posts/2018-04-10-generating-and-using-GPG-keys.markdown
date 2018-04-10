---
layout: post
title: Generating and using GPG Keys
date: 2018-04-10 12:30:00 +0500
description: A brief introduction to GPG Keys, how to generate them and what are its practical uses # Add post description (optional)
img: # Add image post (optional)
tags: [gpg, keys] # add tag
---
GPG is a very interesting utility based upon the public private key cryptographic system that lets you encrypt and decrypt files with a digital signature. If you dont know what that is, [then read this](https://en.wikipedia.org/wiki/Public-key_cryptography). 


This utility lets us tackle a variety of use cases. For example, in our project, we used GPG Keys to sign our debian and rpm packages. How that is done exactly will be covered in a later post. It can also be used to sign git commits and tags ( [read this](https://help.github.com/articles/signing-commits-with-gpg/) ) so that anyone looking at your work can verify that the work has indeed been done by you and nobody else. 

In this post, we will cover the basics of GPG keys such as generating and exporting them and at the end, we'll look into a very simple use case for GPG keys. 

# Generating GPG Keys
Type **gpg --gen-key** in the terminal to start

```sh
hussain@LTP-DEV-HUSSAINAKB ~ $ gpg --gen-key
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```
Next, it will ask you to select the kind of key you want. For the purpose of this tutorial, i selected RSA.

```sh
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
```

Next, it'll ask you for a keysize. The bigger the keysize, the stronger it will be against brute force attacks. However, a bigger size will result in slower encryption and decryption.

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 2048
Requested keysize is 2048 bits
```
Next, it will ask you the time duration for the key expiry.



```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
```
Next, it will ask you for a name and email address. These two entries are important as they will be used while managing keys and using the keys for signing files.

```
You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Hussain Ali Akbar
Email address: foo@bar.com
Comment: test gpg key
You selected this USER-ID:
    "Hussain Ali Akbar (test gpg key) <foo@bar.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

```
Finally, it will ask for a passphrase. This is again an important field as you will need to enter this passphrase during the encryption/decryption process.

```
You need a Passphrase to protect your secret key.
```

Once you provide all of this information, the utility will take some time in generating random bytes in order the generate the keys. Once the process is complete, you will see something like this:

```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
....+++++
+++++
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 66 more bytes)
..+++++

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 128 more bytes)
.+++++
gpg: key E5124174 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   2048R/E5124174 2018-04-10
      Key fingerprint = B162 6587 7A80 C8FB 4032  7C4A 681E FD1D E512 4174
uid                  Hussain Ali Akbar (test gpg key) <foo@bar.com>
sub   2048R/A7F42694 2018-04-10
```

# Listing the GPG Keys
Listing the GPG Keys can be done by this.
```
hussain@LTP-DEV-HUSSAINAKB ~ $ gpg --list-keys
/home/hussain/.gnupg/pubring.gpg
--------------------------------
pub   2048R/E5124174 2018-04-10
uid                  Hussain Ali Akbar (test gpg key) <foo@bar.com>
sub   2048R/A7F42694 2018-04-10
```

# Exporting the GPG Public keys
Exporting the public key is again very simple. The email address that was provided while creating the GPG Key will be used here.

```
hussain@LTP-DEV-HUSSAINAKB ~/Desktop $ gpg --armor --export foo@bar.com
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1

mQENBFrMYFkBCAC3opeWYJs3UORTLsrCpKZUSI+HCVIhL121se/CIU5TgEGZwHea
....
....
ycKvAahACcHQXmBdFjYpZ6HvwLVJ1sVqqK1cw2mstkTpRPrhHXyG+lljSAQm
=ksI7
-----END PGP PUBLIC KEY BLOCK-----
hussain@LTP-DEV-HUSSAINAKB ~/Desktop $ 
```

If you want to save the key in a file, just redirect the output to a file.

```
gpg --armor --export foo@bar.com > key.gpg
```
Once the key file is created, it can sent to the users who will use it to verify your work and/or files.

# Importing GPG Public keys
The public keys can be imported through the following:

```
gpg --import key.gpg
```
Once this is done, the gpg public key can be used to verify other people's work. 

In the next post, we'll see how dpkg-sig can be used to sign debian packages! 