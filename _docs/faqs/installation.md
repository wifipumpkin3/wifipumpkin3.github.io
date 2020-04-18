---
title: FAQ installation
tags: 
 - FAQ
 - installation
description: FAQ Installation
---

### FAQ installation 

1. when I try to start the **Wp3** i got this error:
   
    `ModuleNotFoundError: No module named 'PyQt5.sip'`

After some investigation, I found that the problem can be fixed, if you run on your system os:
```bash 
sudo python3.7 -m pip install --upgrade PyQt5
```

now, on folder wifipumpkin3 you can run this for check if work `sudo make test`. if the log not show any error the bug has been solved. ;)
