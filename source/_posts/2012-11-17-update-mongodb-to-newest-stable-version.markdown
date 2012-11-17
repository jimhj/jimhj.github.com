---
layout: post
title: "Update MongoDB to newest stable version"
date: 2012-11-17 18:57
comments: true
categories: 
---

如果你是在Ubuntu下 sudo apt-get install mongodb, 很不幸，你很有可能会得到一堆错误，具体原因和解决办法如下:

```
  The Ubuntu repositories have a very old version of mongoDB - the current stable branch is 2.0 at the time of writing and yours is 1.6, 
  you could try fixing the error but it's not really worth it. 
  You should get an up to date version from the official 10gen repos as described here and uninstall the one you have now:
```

解决办法参考下面：
[http://www.mongodb.org/display/DOCS/Ubuntu+and+Debian+packages](http://www.mongodb.org/display/DOCS/Ubuntu+and+Debian+packages)

#####General procedure would be:

```
  Remove Ubuntu version (apt-get remove mongodb)
  Add new repo line and gpg key as described above
  Install 10gen version (apt-get update; apt-get install mongodb-10gen)
```
