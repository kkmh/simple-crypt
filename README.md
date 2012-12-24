simple-crypt
============

Simple encryption and decryption for Python 3.

This provides two functions, which encrypt and decrypt data, delegating all
the hard work to the [pycrypto](https://www.dlitz.net/software/pycrypto)
library (which must also be installed).

Example
-------

```python
from binascii import hexlify
from getpass import getpass
from sys import stdin

from simplecrypt import encrypt, decrypt

# read the password from the user (without displaying it)
password = getpass("password: ")

# read the (single line) message we will encrypt
print("message: ")
message = stdin.readline()

# encrypt the message.  we explicitly convert to bytes first (optional)
encrypted = encrypt("somerandomsalt", password, message.encode('utf8'))

# the encrypted message is bytes, so we display it as a hex string
print("encrypted message: %s" % hexlify(encrypted))

# now decrypt the message (using the same salt and password)
decrypted = decrypt("somerandomsalt", password, encrypted)

# the decrypted message is bytes, but we can convert it back to a string
print("decrypted message: %s" % decrypted)
print("decrypted string: %s" % decrypted.decode('utf8'))
```

Which, when run, produces something like (the actual encrypted message will
be different each time, as a random counter - effectively a random IV - is
used for each encryption).

```
password: ******

message:
hello world
encrypted message: b'489b2b5392bb23d4a4314aeffa9156993dad8bd1dd39e2737ad5a3c02f2c84316bb0428cdd78032afe2621ab7782ca5cba7324ca26e51e03d65146a1'
decrypted message: b'hello world\n'
decrypted string: hello world
```

Algorithms
----------

The algorithms used follow the recommendations at
http://www.daemonology.net/blog/2009-06-11-cryptographic-right-answers.html
(plus http://en.wikipedia.org/wiki/PBKDF2), as far as I can tell:

* The "password" is expanded to a 256 bit key, using PBKDF2.  This includes
  a "salt" which should be the same for both encryption and decryption, but
  otherwise "as random as possible" (perhaps user or, at worst, application
  name).

* AES256 CTR mode is used to encrypt the data.  A wraparound counter is used,
  with a random initial offset.  The initial offset is prepended to the
  encrypted message.

* A SHA256 HMAC (of initial offset plus encrypted message) is calculated and
  appended.  This uses the same key as the AES cipher.

* On decryption, the HMAC is validated before decryption.

The [entire implementation is here](https://github.com/andrewcooke/simple-crypt/blob/master/src/simplecrypt/__init__.py).
