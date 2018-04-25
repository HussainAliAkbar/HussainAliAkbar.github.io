---
layout: post
title: Signing and Verifying RPM Packages
date: 2018-04-11 16:44:00 +0500
description: A comprehensive guide to signing and verifying RPM packages # Add post description (optional)
img: # Add image post (optional)
tags: [rpm, signing, verification, packages] # add tag
---

[The previous post](../digitally-signing-and-verification-of-debian-packages-with-dpkg-sig/) talked about signing and verifying the signatures of Debian Packages. This is a continuation of that post.

1 - The first thing that you need for signing an RPM Package, is of course, an RPM Package. Creating an RPM Package form scratch is not a part of this post, neither is the process of converting a .deb package to an .rpm package (thats what i did!). So, you can check out [this excellent guide](https://www.tecmint.com/convert-from-rpm-to-deb-and-deb-to-rpm-package-using-alien/) in case you dont have one. 

2 - The second thing that you'll need are the GPG Keys. I already have an entire post on [generating and using GPG Keys](../generating-and-using-GPG-keys/) so you can refer to that in case you need any help. Once the GPG Keys are created, they can be imported like this:

```
sudo rpm --import RPM-GPG-KEY-PUBLIC
```
Its better to verify that the key has been imported successfully in the RPM database
```
rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'
```

3 - Install **rpm-sign** if not already installed:

```
sudo yum install rpm-sign
```

4 - Now that we have all the prerequisites, we can finally sign the package!

```
rpm --define "_gpg_name Hussain Ali Akbar <foo@bar.com>" --addsign testpackage-1.1.1.11-1.x86_64.rpm
```
Here, you need to switch the name and email with the values that you defined while creating the GPG keys.

5 - Verify that the package has been signed successfully:
```
rpm --checksig testpackage-1.1.1.11-1.x86_64.rpm
```

A signed package would output something like this:
```
testpackage-1.1.1.11-1.x86_64.rpm: rsa sha1 (md5) pgp md5 OK
```

whereas an unsigned package would output something like this:
```
testpackage-1.1.1.11-1.x86_64.rpm: sha1 md5 OK
```

Another method to verify that the package signature exists or not is by doing this:

```
rpm -qpi testpackage-1.1.1.11-1.x86_64.rpm
```
A signed package would output a lot of items with the signature field as follows:
```
Signature   : RSA/SHA1, Fri 09 Mar 2018 02:40:26 PM PKT, Key ID 155550404e6a93b0
```

Whereas an unsigned package will output :
```
Signature   : (none)
```

Here, we are verifying the package signature on the same system that we signed it on. To verify the package's signature on other systems, you'll need to import the Public key of the GPG Key that was used to sign it and the signature should be verified successfully.

And thats all the information that i have on package signing! Let me know if i missed something or did something incorrect. Any suggesstions and enhancements are appreciated! Cheers!



