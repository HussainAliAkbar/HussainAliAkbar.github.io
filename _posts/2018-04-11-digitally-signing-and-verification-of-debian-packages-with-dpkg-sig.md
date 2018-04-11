---
layout: post
title: Digitally Signing and Verification of Debian packages with dpkg-sig
date: 2018-04-11 16:44:00 +0500
description: A comprehensive guide to signing and verifying debian packages # Add post description (optional)
img: # Add image post (optional)
tags: [deb, signing, verification, packages] # add tag
---

Security Standards such as Common Criteria make it a requirement for all the Debian packages and installers to be Digitally signed so that they can be verified by the users in order to achieve compliance. This can be done very easily with dpkg-sig - a tool for generating and verifying signatures of debian packages. 

1 - First thing's first, we will need a sample package that we can sign and then verify. For this purpose, i chose the **aribas** package which can be [downloaded](https://packages.debian.org/stable/math/aribas) from the official debian repository.

2 - Once we have the package that we need to sign and verify, we will need to create **GPG Keys**. I already have an entire post on [generating and using GPG Keys](../generating-and-using-GPG-keys/) so you can refer to that in case you need any help.

3 - The next step is to install the **dpkg-sig** utility. That can be downloaded from the apt repository.

```
sudo apt-get install dpkg-sig
```
4 - Next we will need the GPG ID of the key that we will be using to sign the package. This id can be found by listing the gpg keys.

```
hussain@LTP-DEV-HUSSAINAKB ~/Downloads $ gpg --list-keys
/home/hussain/.gnupg/pubring.gpg
--------------------------------
pub   2048R/E5124174 2018-04-10
uid                  Hussain Ali Akbar (test gpg key) <foo@bar.com>
sub   2048R/A7F42694 2018-04-10
```

Here, **E5124174** is the GPG ID.

5 - The package can now be signed.
```
dpkg-sig -k <GPG ID> --sign builder aribas_1.64-6_amd64.deb 
```
change the GPG ID with the ID of your GPG Key. 

6 - Verify the signature just to ensure that the package has been signed successfully.
```
hussain@LTP-DEV-HUSSAINAKB ~/Downloads $ dpkg-sig --verify aribas_1.64-6_amd64.deb Processing aribas_1.64-6_amd64.deb...
GOODSIG _gpgbuilder B16265877A80C8FB40327C4A681EFD1DE5124174 1523440651
hussain@LTP-DEV-HUSSAINAKB ~/Downloads $ 
```
You should see **GOODSIG** in the output of the verify command. Here i have verified the signature on the same machine on which i signed it so i did not run into any issues. On other user machines, the GPG Keys will first need to be to be imported into the system and then the package's signature can be verified. The export and import process of the GPG Keys is defined in the [previous post](../generating-and-using-GPG-keys/) as well. 

There is a bug in older versions of dpkg-sig where it fails to verify the signature of debian packages compressed with xz. When you try to verify such a package it will give you **"BADSIG _gpgbuilder** error. This bug has been reported [here](https://bugs.launchpad.net/ubuntu/+source/dpkg-sig/+bug/1342938). This will not cause any issues on newer Debian based OSes like Ubuntu 16.04 LTS or Linux Mint 18.3, however, if you download dpkg-sig utility from the apt repository on older OSes like Ubuntu 14.04 LTS, it will download the older version of dpkg-sig which can cause issues while verifying the package.

If you stuck with older dpkg-sig versions, you can get around this by simply downloading the newer package from [the official repository](http://ftp.us.debian.org/debian/pool/main/d/dpkg-sig/dpkg-sig_0.13.1+nmu4_all.deb) and extract it like this:
```
ar vx dpkg-sig_0.13.1+nmu4_all.deb
```
Next, extract the contents from the **data.tar.xz**.
```
tar -xvf data.tar.xz
```
You should now have a usr folder which will contain the executable script for dpkg-sig. You can use it just like any other script.
```
usr/bin/dpkg-sig --verify aribas_1.64-6_amd64.deb
```
This should give you the correct output! 
I hope this was helpful. In the next post, we'll look at signing RPM packages!
