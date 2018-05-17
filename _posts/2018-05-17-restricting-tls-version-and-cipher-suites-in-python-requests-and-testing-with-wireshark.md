---
layout: post
title: Restricting TLS Version and Cipher Suites in Python's Requests and testing with Wireshark
date: 2018-05-17 10:40:00 +0500
description: Restricting TLS Version and Cipher Suites in Python's Requests and testing with Wireshark
img: # Add image post (optional)
tags: [python, requests, tls, ciphers, wireshark, security, fips, cc] # add tag
---


Python's [Requests](http://docs.python-requests.org/en/master/#) is a very powerful Library that can be used HTTP requests. It's very easy to use and has tons of great features. While working on CC Compliance, I needed to restrict the TLS Version to 1.2 as well as restrict the cipher suites in the Client Hello Packet. And I needed to do this through the request's library.

I found alot of solutions online but most of them contained separate  implementations for restricting the TLS Version and Restricting the Cipher Suites. I could not find a single solution which contained both the functionalities and so i had to come up with it myself.

Before you can get started with the code, make sure that you have the requests library installed:

```
pip install requests
```

The following code (i'll explain the bits and pieces later) achieves the required functionality. Scroll down further after the code for explanation and tests:


```python
import ssl
import requests

from requests.adapters import HTTPAdapter
from requests.packages.urllib3.poolmanager import PoolManager
from requests.packages.urllib3.util import ssl_

CIPHERS = (
    'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:
    ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:AES256-SHA'
)


class TlsAdapter(HTTPAdapter):

    def __init__(self, ssl_options=0, **kwargs):
        self.ssl_options = ssl_options
        super(TlsAdapter, self).__init__(**kwargs)

    def init_poolmanager(self, *pool_args, **pool_kwargs):
        ctx = ssl_.create_urllib3_context(ciphers=CIPHERS, cert_reqs=ssl.CERT_REQUIRED, options=self.ssl_options)
        self.poolmanager = PoolManager(*pool_args,
                                       ssl_context=ctx,
                                       **pool_kwargs)


session = requests.session()
adapter = TlsAdapter(ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1)
session.mount("https://", adapter)

try:
    r = session.request('GET', 'https://google.com')
    print(r)
except Exception as exception:
    print(exception)
```


**Following are the important points to note:**

1. You can control the Cipher List through the **CIPHERS** variable. That variable takes a string parameter of all the cipher suites that you need to allow separated with a ":" . The list of available cipher suites can be found by running "openssl ciphers" in the terminal.

2. You can control which TLS versions to restrict using the parameters of the **TlsAdapter**. It can take the following arguments:

      * ssl.OP_NO_TLSv1 (For restricting TLS V1)
      * ssl.OP_NO_TLSv1_1 (For restricting TLS V1.1)
      * ssl.OP_NO_TLSv1_2 (For restricting TLS V1.2)

3. You can control the HTTP Method and URL through the parameters of **session.request**. Here i have used the GET Method. However, if you are using any other method that requires you to send a request body such as POST or PUT then that can also be added here as an argument. 

Now, in order to test this whether we have indeed managed to restrict the Cipher Suites and the TLS Version, we will need wireshark which is a very popular packet analyzer tool. If you dont already have that, install it by running the following command:


```
sudo apt-get install wireshark
```

Once Wireshark is installed, run it.
```
sudo wireshark
```


![Wireshark Landing Page]({{site.baseurl}}/assets/img/WiresharkLanding.png)

On this screen, you should a list of available network connections of your current device. For example, in the image above you can see enp0s25, wlp3s0 etc. These networks coorespond to your ethernet connection, wireless connection etc. I chose enp0s25 because that is the name of my ethernet connection. Choose the network connection depending on which connection you're currently on.

Once you select the network connection, You should arrive at this screen which lists down all the packets sent and received from your machine.

![Wireshark Packets Page]({{site.baseurl}}/assets/img/WiresharkPackets.png)

However, you should see alot of packets here that are of no use to us. So we need to filter down the traffic captured by wireshark using the ip address of the server that we are going to hit. For the purpose of this tutorial, I used the IP Address of Google. 

Copy the following line in the Display Filter text field and replace the ip with with the ip of the server that you are going to test it on:
```ip.addr == 216.58.213.227```

Now run the Python script:
```
python [filename].py
```
The terminal should output a simple ```<Response [200]>``` which means that the request was received successfully. 

However, there is a chance that you may not have the traffic captured in wireshark due to the fact that Google uses alot of IP addresses and the IP Address that we're using for filtering our packet data might not be the IP on which our request was sent. In order to fix this, we need to add an entry in the hosts file.

open the hosts file in any editor with Sudo privileges:

```
sudo vim /etc/hosts
```
and add the following line: ```216.58.213.227  google.com```. This way, all the requests for "google.com" will be sent to the **216.58.213.227** ip address.

Once this is done, rerun the Python script and you should now see the packet captured in Wireshark.

![Captured Packet]({{site.baseurl}}/assets/img/PacketCaptured.png)

Here, in the client hello packet, we can see that the Protocol is **TLSv1.2**. To check which cipher suites were sent by the python script, Navigate to: **Secure Sockets Layer -> TLSv1.2 Record Layer -> Handshake Protocol -> Cipher Suites**

![Cipher Suites]({{site.baseurl}}/assets/img/CipherSuites.png)

Here, we can see that only those cipher cuites are sent to the server for TLS Negotiation that we allowed in our Python Script. 

To further test this out, lets modify the TLS Version and the Restricted Cipher Suites. 
Change the CIPHER Variable to just this:

```python
CIPHERS = (
    'AES256-SHA'
)
```

And change the TLS Version arguments to restrict TLSv1 and TLSv1.2 instead of TLSv1.1:

```
adapter = TlsAdapter(ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_2)
```

Execute the Python script again and view the results in Wireshark.

![Modified TLS Version]({{site.baseurl}}/assets/img/modifiedTLSVersion.png)

We can now see that the Protocol being used to communicate is TLSv1.1. Also, only one cipher suite is sent by the client for TLS negotiation and thats the one that we have specified. 

I hope that this was helpful! Please feel free to contact me in case you run into any issues. Cheers!