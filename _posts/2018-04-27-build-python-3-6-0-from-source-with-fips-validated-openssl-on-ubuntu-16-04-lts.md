---
layout: post
title: Build Python 3.6.0 from source with FIPS validated OpenSSL on Ubuntu 16.04 LTS
date: 2018-04-27 10:44:00 +0500
description: Build Python 3.6.0 from source with FIPS validated OpenSSL on Ubuntu 16.04 LTS
img: # Add image post (optional)
tags: [fips, python, openssl, ubuntu] # add tag
---

In the [first part](../add-support-for-fips-mode-and-fips-mode-set-in-python-3-6-0/) of the post, we talked about modifying the source code of Python 3.6.0 to introduce two essential functions - namely FIPS_mode() and FIPS_mode_set() to toggle the FIPS mode of the SSL Module of Python.


Today, We will be looking at building Python 3.6.0 from source with a FIPS validated OpenSSL. 
Since the steps involve moving around and tinkering with alot of system dependent files, i prefer to do this inside a docker container so that my system does not get affected in case of any errors. 
Feel free to do this on any Ubuntu 16.04 LTS system and ignore the steps that i provide for the Docker container setup.

Setting up and using Docker is not a part of this post but there are alot of guides available [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04), [here](https://medium.com/@Grigorkh/how-to-install-docker-on-ubuntu-16-04-3f509070d29c) and [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/) that can help in setting up Docker on your local system.

Once docker is set up on your system, create a working directory and copy the source code of the modified python into that directory. Create another directory called *"output"*. We will mount this directory with our docker container so that we are able to transfer files between our system and the docker container. In case you dont know about Docker volumes, [read here](https://docs.docker.com/storage/volumes/). 

Once this is done, run the following command to start a new container:
```ssh
docker run --name ubuntu-container -it -d -v ${PWD}/output:/output -v ${PWD}/Python-3.6.0:/Python-3.6.0 ubuntu
```
This will set up a new container and copy the modified Python 3.6.0 source code in it. You would also be able to see an *output* folder in the root directory. If you copy any files in that folder, they will be copied outside the docker container into your system's output folder.

Moving on, once the container is set up, you can enter it. Just replace the CONTAINER_ID with the id of your container. 
```ssh
docker exec -it CONTAINER_ID bash
```

From here onwards, The steps should be the same for all.

##### 1 - Install the Dependencies
```ssh
apt-get update && apt-get install tcl-dev tk-dev bzip2 libbz2-dev build-essential \
g++ make zlib1g-dev libffi-dev libc6 fakeroot libcups2-dev libkrb5-dev libyaml-dev \
devscripts debhelper libqt4-dev curl git file cmake libsqlite3-dev wget
```


##### 2 - Check whether OpenSSL exists in the system
```ssh
openssl version
```
This should output something like *OpenSSL 1.0.2g  1 Mar 2016* but the versions may vary depending on the system.

##### 3 - Switch to the home directory
```ssh
cd /home
```



##### 4 - Download the source code for OpenSSL and OpenSSL FIPS Module
```ssh
wget https://www.openssl.org/source/old/fips/openssl-fips-2.0.12.tar.gz && \
wget https://www.openssl.org/source/old/1.0.2/openssl-1.0.2h.tar.gz
```


##### 5 - Extract the tar files
```ssh
tar -xvf openssl-1.0.2h.tar.gz && tar -xvf openssl-fips-2.0.12.tar.gz
```


##### 6 - Build OpenSSL Fips Module and OpenSSL
```ssh
cd openssl-fips-2.0.12 && ./config && make && make install && cd .. && \
cd openssl-1.0.2h && ./config shared fips && make && make install
```

##### 7 - Verify that FIPS validated OpenSSL has been installed properly
```ssh
/usr/local/ssl/bin/./openssl version
```

##### 8 - Replace system OpenSSL with the FIPS validated OpenSSL
```ssh
ln -s -f /usr/local/ssl/bin/openssl /usr/bin/openssl
```

##### 9 - Verify that OpenSSL has been replaced successfully. It should now show "FIPS"
```ssh
openssl version
OpenSSL 1.0.2h-fips  3 May 2016
```

##### 10 - Move the System’s libcrypto and libssl shared objects
```ssh
mv /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 /lib/x86_64-linux-gnu/old_libcrypto.so.1.0.0 \
&& mv /lib/x86_64-linux-gnu/libssl.so.1.0.0 /lib/x86_64-linux-gnu/old_libssl.so.1.0.0
```

##### 11 - Copy new fips-enabled libcrypto and libssl shared objects
```ssh
cp /usr/local/ssl/lib/libcrypto.so.1.0.0 /lib/x86_64-linux-gnu/ && \
cp /usr/local/ssl/lib/libssl.so.1.0.0 /lib/x86_64-linux-gnu/
```

##### 12 - Switch to the Python 3.6.0 source code directory
```ssh
cd /Python-3.6.0/
```


##### 13 - Build Python 3.6.0
```ssh
./configure  --enable-shared --prefix=/usr/local/python3.6 && make && make install
```

##### 14 - Copy libpython3.6m.so.1.0 to system directories
```ssh
cp /usr/local/python3.6/lib/libpython3.6m.so.1.0 /usr/lib/x86_64-linux-gnu/ \
&& cp /usr/local/python3.6/lib/libpython3.6m.so.1.0 /lib/x86_64-linux-gnu/
```

##### 15 - Run Python and confirm that the FIPS Functions are working properly
```ssh
root@cdc0d80758bd:/home/Python-3.6.0# /usr/local/python3.6/bin/./python3.6
Python 3.6.0 (default, Mar 30 2018, 09:52:46) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.

>>> import ssl
successful import

>>> ssl.OPENSSL_VERSION
'OpenSSL 1.0.2h-fips  3 May 2016'

>>> ssl.FIPS_mode()
0

>>> ssl.FIPS_mode_set(1)

>>> ssl.FIPS_mode()  
1
>>> 
```
When you print the OpenSSL Version with ssl.OPENSSL_VERSION, you should see “OpenSSL 1.0.2h-fips 3 May 2016”. 

When you initially check the FIPS mode with ssl.FIPS_mode(), you should see “0” which indicates that the FIPS mode has not been toggled. 

When you execute ssl.FIPS_mode_set(1), you do not see any output which is the expected behavior. 
If any error is thrown such as “FIPS mode not supported” then that means that the FIPS mode has not been toggled properly.

Now, when you again check the FIPS mode with ssl.FIPS_mode(), you should see “1” which indicates that FIPS mode has now been enabled.

If you have reached this point without any issues, then you have compiled Python successfully with a FIPS validated OpenSSL. And you can use this to run your python applications in a FIPS enabled enviroment.

**However**, we can take this a bit further and use [cxFreeze](https://anthony-tuininga.github.io/cx_Freeze/) to create Python applications that can run on any ubuntu systems in FIPS mode regardless of whether a FIPS validated OpenSSL exists on that system or not. Just follow the steps below.


##### 16 - Install cx-Freeze
```ssh
/usr/local/python3.6/bin/./pip3.6 install cx-Freeze==5.0.1
```

##### 17 - Export System flags
```ssh
  export LDFLAGS="-L/usr/local/ssl/lib/"
  export SL_INSTALL_PATH=/usr/local/ssl
  export OPENSSL_FIPS=1
  export LD_LIBRARY_PATH="/usr/local/ssl/lib/"
  export CPPFLAGS="-I/usr/local/ssl/include/ -I/usr/local/ssl/include/openssl/"
```

##### 18 - Install vim if not installed. This will be used to create a test python script.
```ssh
apt-get install vim
```

##### 19 - Create a test script
```ssh
touch /home/testCxFreeze.py && vim /home//testCxFreeze.py
```

Add the following contents to the file:

```python
import ssl
print("OpenSSL Version: ", ssl.OPENSSL_VERSION)
print("Initial FIPS Mode: ", ssl.FIPS_mode())
print("Toggle FIPS Mode: ", ssl.FIPS_mode_set(1))
print("FIPS Mode after toggle: ", ssl.FIPS_mode())
```

##### 20 - Freeze the script
```ssh
/usr/local/python3.6/bin/./cxfreeze /home/testCxFreeze.py --target-dir /home/testCxFreeze
```

##### 21 - Test the freezed script
When you run the script, you should see the following output.
```ssh
root@f47cb09d42ab:/home/testCxFreeze# /home/testCxFreeze/./testCxFreeze
OpenSSL Version:  OpenSSL 1.0.2h-fips  3 May 2016
Initial FIPS Mode:  0
Toggle FIPS Mode:  None
FIPS Mode after toggle:  1
```

Of course, since we're still in the environment where we built Python. So to have a final test, extract this script out of the docker container (or outside the system in case you are not using docker) so that we can run it on another system.

##### 22  - Extract the test script folder from Docker
```ssh
scp -r testCxFreeze /output/
```
If you mounted the output folder correctly like i mentioned earlier in the post, the **testCxFreeze** folder should now appear in your system's output folder.

##### 23 - Run the testCxFreeze executable
```ssh
hussain@LTP-HUSSAIN ~/Desktop/projects/personal/pythonFips/output/testCxFreeze $ ./testCxFreeze 
OpenSSL Version:  OpenSSL 1.0.2h-fips  3 May 2016
Initial FIPS Mode:  0
Toggle FIPS Mode:  None
FIPS Mode after toggle:  1
```
And is how we can export a Python application running in FIPS mode to other systems.

I hope this was helpful because it certainly made my life easier. If you run into any issues during any of the steps above, please feel free to contact me. Cheers!

