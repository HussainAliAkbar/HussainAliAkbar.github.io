---
layout: post
title: Add support for FIPS_mode() and FIPS_mode_set() in Python 3.6.0
date: 2018-04-25 16:44:00 +0500
description: A guide to add support for FIPS_mode and FIPS_mode_set in Python which does not exist by default # Add post description (optional)
img: # Add image post (optional)
tags: [python, fips] # add tag
---

While working on FIPS compliance for one of our projects, we came across a scenario where the project was in Python 3.6 that was freezed into an executable with cxfreeze. And our task was to package FIPS validated OpenSSL with the Python project. 

After some research, we found out that there was no native functionality in Python to enable the FIPS mode of the OpenSSL module packaged with the Python Application. So, our task became twofold:

1.  Add support for FIPS_mode() and FIPS_mode_set() in Python
2.  Set up a build environment that will package FIPS validated OpenSSL with the Python application

#### Add support for FIPS_mode() and FIPS_mode_set() in Python 3.6.0

This post deals with the first part of the problem that we faced. After alot of research, we came across [this patch](https://bugs.python.org/issue27592) which added support for FIPS_mode and FIPS_mode_set in Python 3.4. And this patch helped us in adding the same support in Python 3.6. The source code for Python 3.6.0 can be downloaded from [here](https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz).

The code snippets below details the changes that you need to make in order to introduce support for the FIPS functions.

##### **Python-3.6.0/Lib/ssl.py**:

```diff
     # LibreSSL does not provide RAND_egd
     pass
     
+try:
+    from _ssl import FIPS_mode, FIPS_mode_set
+except ImportError as e:
+    sys.stderr.write('error in importing\n')
+    sys.stderr.write(str(e))
 
 from _ssl import HAS_SNI, HAS_ECDH, HAS_NPN, HAS_ALPN
 from _ssl import _OPENSSL_API_VERSION
```
##### **Python-3.6.0/Modules/Setup.dist**:

```diff
 #_csv _csv.c
 
 # Socket module helper for socket(2)
-#_socket socketmodule.c
+_socket socketmodule.c
 
 # Socket module helper for SSL support; you must comment out the other
 # socket line above, and possibly edit the SSL variable:
-#SSL=/usr/local/ssl
-#_ssl _ssl.c \
-#	-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
-#	-L$(SSL)/lib -lssl -lcrypto
+SSL=/usr/local/ssl
+_ssl _ssl.c \
+	-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
+	-L$(SSL)/lib -lssl -lcrypto
 
 # The crypt module is now disabled by default because it breaks builds
 # on many systems (where -lcrypt is needed), e.g. Linux (I believe).
 ```


##### **Python-3.6.0/Modules/_ssl.c**:
```diff
     return PyLong_FromLong(RAND_status());
 }
 
+static PyObject *
+_ssl_FIPS_mode_impl(PyObject *module) {
+    return PyLong_FromLong(FIPS_mode());
+}
+
+static PyObject *
+_ssl_FIPS_mode_set_impl(PyObject *module, int n) {
+    if (FIPS_mode_set(n) == 0) {
+        _setSSLError(ERR_error_string(ERR_get_error(), NULL) , 0, __FILE__, __LINE__);
+        return NULL;
+    }
+    Py_RETURN_NONE;
+}
+
 #ifndef OPENSSL_NO_EGD
 /* LCOV_EXCL_START */
 /*[clinic input]
 ```
 
 ```diff
 
     _SSL_ENUM_CRLS_METHODDEF
     _SSL_TXT2OBJ_METHODDEF
     _SSL_NID2OBJ_METHODDEF
+    _SSL_FIPS_MODE_METHODDEF
+    _SSL_FIPS_MODE_SET_METHODDEF
     {NULL,                  NULL}            /* Sentinel */
 };
 
 ```

##### **Python-3.6.0/Modules/clinic/_ssl.c.h**:
```diff

     return _ssl_RAND_status_impl(module);
 }
 
+PyDoc_STRVAR(_ssl_FIPS_mode__doc__,
+"FIPS Mode");
+
+#define _SSL_FIPS_MODE_METHODDEF    \
+    {"FIPS_mode", (PyCFunction)_ssl_FIPS_mode, METH_NOARGS, _ssl_FIPS_mode__doc__},
+
+static PyObject *
+_ssl_FIPS_mode_impl(PyObject *module);
+
+static PyObject *
+_ssl_FIPS_mode(PyObject *module, PyObject *Py_UNUSED(ignored))
+{
+    return _ssl_FIPS_mode_impl(module);
+}
+
+PyDoc_STRVAR(_ssl_FIPS_mode_set_doc__,
+"FIPS Mode Set");
+
+#define _SSL_FIPS_MODE_SET_METHODDEF    \
+    {"FIPS_mode_set", (PyCFunction)_ssl_FIPS_mode_set, METH_O, _ssl_FIPS_mode_set_doc__},
+
+static PyObject *
+_ssl_FIPS_mode_set_impl(PyObject *module, int n);
+
+static PyObject *
+_ssl_FIPS_mode_set(PyObject *module, PyObject *arg)
+{
+    PyObject *return_value = NULL;
+    int n;
+
+    if (!PyArg_Parse(arg, "i:FIPS_mode_set", &n)) {
+        goto exit;
+    }
+    return_value = _ssl_FIPS_mode_set_impl(module, n);
+
+exit:
+    return return_value;
+}
+
 #if !defined(OPENSSL_NO_EGD)
 
 PyDoc_STRVAR(_ssl_RAND_egd__doc__,

```

I also have a [gist on github](https://gist.github.com/HussainAliAkbar/37f996f1e009b0ee45b96c0252761d7f) so if you run into any issues, feel free to post a comment there and I'll be happy to help in any way that i can. 

In the next post, i'll list down the steps to set up a build environment that can package a FIPS validated OpenSSL with the Python application. so Stay tuned!





