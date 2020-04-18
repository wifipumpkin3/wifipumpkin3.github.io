---
title: Logger settings
description: logger file structure 
---

## Overview

the wp3 have a system log manager that save log in format `.json` or not if you set on manually in `config.ini`, see this page [config](configuration-file), to save in simple log like wp2. the output `.json` have many information of session like date, timestamp, session id, extra information the packets and etc. if you not save `.json` only sove the log of process generate on wp3 interface.

### Session ID

the session id can be restored if you want to continue the attack in next day or whatever you want. It very powerful restore the session for great log report that will be implemented coming soon.

### Log localization

All log generate from wp3 will be a unique folder in `logs`,  ~ is your home directory, usually /home/username. A file or folder name starting with a . is the Linux version of a hidden file/folder. So ~/.config is a hidden folder within your home directory. On ~/.config exist `~/.config/wifipumpkin3` when wp3 is installed. the file structure is like:

```bash
$ cd ~/.config/wifipumpkin3/
$ ls 
 config  exceptions  helps  logs  scripts
```

```bash
$ cd ~/.config/wifipumpkin3/logs/ap
$ ls
captiveportal.log  pumpkin_proxy.log  pydns_server.log  responder3.log  sniffkin3.log
```

the extension is .log, but all data into file by default is `json object`. let`s take a look on file example sniffkin3.log that contain all data capture http traffic packet.

```json

  "text": "2020-04-09 at 19:23:29 INFO - [ 10.0.0.21 > 172.217.30.99 ] b'GET' b'connectivitycheck.gstatic.com'b'/generate_204'\n",
  "record": {
    "elapsed": {
      "repr": "0:00:13.495115",
      "seconds": 13.495115
    },
    "exception": null,
    "extra": {
      "dns": 144,
      "session": "b52e2300-7ab0-11ea-95ed-94e979fa917b",
      "packet": {
        "urlsCap": {
          "IP": {
            "options": [
            ],
            "version": 4,
            "ihl": 5,
            "tos": 0,
            "len": 284,
            "id": 44754,
            "flags": "",
            "frag": 0,
            "ttl": 64,
            "proto": 6,
            "chksum": 62904,
            "src": "10.0.0.21",
            "dst": "172.217.30.99"
          },
          "Headers": {
            "Headers": "b'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.32 Safari/537.36\\r\\nHost: connectivitycheck.gstatic.com\\r\\nConnection: Keep-Alive\\r\\nAccept-Encoding: gzip'",
            "Host": "b'connectivitycheck.gstatic.com'",
            "User-Agent": "b'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.32 Safari/537.36'",
            "Accept-Encoding": "b'gzip'",
            "Connection": "b'Keep-Alive'",
            "Method": "b'GET'",
            "Path": "b'/generate_204'",
            "Http-Version": "b'HTTP/1.1'"
          }
        }
      },
      "name": "sniffkin3",
      "specific": true
    },
    "file": {
      "name": "logger_manager.py",
      "path": "/usr/local/lib/python3.7/dist-packages/wifipumpkin3-1.0.0-py3.7.egg/wifipumpkin3/core/widgets/default/logger_manager.py"
    },
    "function": "info",
    "level": {
      "icon": "ℹ️",
      "name": "INFO",
      "no": 20
    },
    "line": 149,
    "message": "[ 10.0.0.21 > 172.217.30.99 ] b'GET' b'connectivitycheck.gstatic.com'b'/generate_204'",
    "module": "logger_manager",
    "name": "wifipumpkin3.core.widgets.default.logger_manager",
    "process": {
      "id": 10809,
      "name": "MainProcess"
    },
    "thread": {
      "id": 140676164286272,
      "name": "MainThread"
    },
    "time": {
      "repr": "2020-04-09 19:23:29.459485-03:00",
      "timestamp": 1586471009.459485
    }
  }
}
```

The format file is each line contain a json file data struture as you can see above. 

### Json Format

the wp3 use the python module `loguru`, this a great and powerful module for system logger, in `config.ini` file have a key for change to normal log report. this format json should be great to be readed by API, that will be implemented in thre future updates. 

```ini
# get file on ~/.config/wifipumpkin3/config/app/config.ini
[settings]
log_colorize=true # for get colors when print output on CLI interface
log_serialize=true # for diasble json format only change for false
```




