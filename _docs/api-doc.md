---
title: API documentation
tags: 
 - api
 - documentation
description: API documentation
---

# RestAPI Wp3 Documentation 

## Introduction

The **wp3** (version 1.0.8) now has a **restAPIful** with module **flask**, it great, because you can to control all features of the **wp3** sending request HTTP. you can send commands and set all properties like you can do on CLI version.

The default username for the API is 'wp3admin' and password should be passed by user when start the paraments on CLI. for example with --rest --username admin --password 'Passwordwp3'.

```
→ sudo wifipumpkin3 -h                                                                    [46e94fe]
[sudo] password for mh4x0f: 
usage: wifipumpkin3 [-h] [-i INTERFACE] [-s SESSION] [-p PULP] [-x XPULP]
                    [-m WIRELESS_MODE] [--no-colors] [--rest]
                    [--restport RESTPORT] [--username USERNAME]
                    [--password PASSWORD] [-v]

wifipumpkin3 - Powerful framework for rogue access point attack.

optional arguments:
  -h, --help            show this help message and exit
  -i INTERFACE          set interface for create AP
  -s SESSION            set session for continue attack
  -p PULP, --pulp PULP  interactive sessions can be scripted with .pulp file
  -x XPULP, --xpulp XPULP
                        interactive sessions can be string with ";" as the
                        separator
  -m WIRELESS_MODE, --wireless-mode WIRELESS_MODE
                        set wireless mode settings
  --no-colors           disable terminal colors and effects.
  --rest                Run the Wp3 RESTful API.
  --restport RESTPORT   Port to run the Wp3 RESTful API on. default is 1337
  --username USERNAME   Start the RESTful API with the specified username
                        instead of pulling from wp3.db
  --password PASSWORD   Start the RESTful API with the specified password
                        instead of pulling from wp3.db
  -v, --version         show program's version number and exit
```

## Why?

Why create a Restful API for **Wp3** ? Many users come from of the old version for python2.7 with PyQt5 interface, but this not the real motivation for create the API. the real motivation is do make wp3 run on respberry or any device with support debian OS more easy, because the pyqt5 have many bugs and depend of GUI of OS for run. the restful API, in the future,  allow to create a web interface (frontend) with support of all features.

## Limitations

the API is designed to provide only the essential features like , commands, configs, clients connected, loggers and etc. somes bugs should be happen when the AP has been started, but I think that can be solved with web interface, controling the paraments of initialization.


## RESTAPI with Insomnia

The wp3 API it very simples to use, frist of all the API has been designed with [insomnia](https://insomnia.rest/download/), the insomnia will be useful to test the wp3 routes. 

![insomnia](/assets/img/insomnia_screen.png)

Download export file: [exp_insomnia](/assets/download/insomnia_export_wp3_.json)

## How to RestAPI wp3 work

Endpoint: **/api/v1**
Default port: **5000**
Protocol: **HTTP**

The restAPI wp3 have a **token jwt** that is used for all request on the routes. the jwt token should be added on header of HTTP with a parament **x-access-token** and the value is the token jwt.

Token jwt example:

``` sh
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwdWJsaWNfaWQiOiI2ZDQwYTE2Yi0zMzA2LTQxOTAtOTQ4OS0yN2U4M2Q0ZjZiM2UiLCJleHAiOjE2MDQ5NjI2NzR9.Zy03Q90P6hqoAjgHY7Znl7irhCHguaxM7SBMdwAN8U8
```

this is a token jwt you should be send for all header request, exceptions on the route /authenticate.

## API Authentication

the frist request should be do is the GET /authenticate, because with this rest should be returned the token for access and control the wp3. the credentails should be passed with Basic auth format, for example the password and username into the header of http request converted for base64 **d3AzYWRtaW46MTIzNDU=** == "wp3dmin:12345". as you can see bellow:

```
> GET /api/v1/authenticate HTTP/1.1
> Host: localhost:5000
> Authorization: Basic d3AzYWRtaW46MTIzNDU=
> User-Agent: insomnia/2020.3.3
> Accept: */*

* Mark bundle as not supporting multiuse
* HTTP 1.0, assume close after body

< HTTP/1.0 308 PERMANENT REDIRECT
< Content-Type: text/html; charset=utf-8
< Content-Length: 291
< Location: http://localhost:5000/api/v1/authenticate/
```

if the return is http code 200, the body of request should be like this,

``` json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwdWJsaWNfaWQiOiI2ZDQwYTE2Yi0zMzA2LTQxOTAtOTQ4OS0yN2U4M2Q0ZjZiM2UiLCJleHAiOjE2MDQ5NjI2NzR9.Zy03Q90P6hqoAjgHY7Znl7irhCHguaxM7SBMdwAN8U8"
}
```

## AccessPoint information

Returns json data about all infromation from the AP

- URL
    `/config/accesspoint`
    `/config/accesspoint/:{keyname}`

- Method:
    `GET`

- Header Params:
    x-access-token=`TOKEN jwt`

- URL Params
    - Optional:
        `keyname=[string]`

    - Example:
        `/config/accesspoint/interface`

- Success Response:
    - code: 200
    - Body :
    `
        {
        "ap_max_inactivity": "3600",
        "bssid": "BC:F6:85:03:36:5B",
        "channel": "11",
        "checkConnectionWifi": "true",
        "check_support_ap_mode": "true",
        "current_session": "cccce846-1580-11eb-bebd-4e20afbd5e1d",
        "enable_security": "false",
        "interface": "None",
        "path_pydns_server_zones": "core/config/app/dns_hosts.ini",
        "persistNetwokManager": "true",
        "pydhcp_server": "true",
        "pydns_server": "true",
        "ssid": "WiFi Pumpkin 3",
        "status_ap": "false",
        "timer_update_info": "2000",
        "wpa_algorithms": "TKIP",
        "wpa_sharedkey": "1234567890",
        "wpa_type": "2"
        }
    `

## AccessPoint Setings

change data about accesspoint configuration 

- URL
    `/config/accesspoint`

- Method:
    `POST`

- Header Params:
    x-access-token=`TOKEN jwt`

- Body Params

``` json
{
	"ssid" : "test",
	"interface": "wlan0"
}
```

- Success Response:
    - code: 200
    - Body :
    `
    {
        "ssid" : "test",
        "interface": "wlan0"
    }
    `


## DHCP information

Returns json data about all infromation from the DHCP server

- URL
    `/config/dhcp`

- Method:
    `GET`

- Header Params:
    x-access-token=`TOKEN jwt`

- Success Response:
    - code: 200
    - Body :
    `
        {
        "broadcast": "10.0.0.255",
        "classtype": "A",
        "leasetimeDef": "600",
        "leasetimeMax": "7200",
        "netmask": "255.0.0.0",
        "range": "10.0.0.20/10.0.0.50",
        "router": "10.0.0.1",
        "subnet": "10.0.0.0"
        }
    `

## Settings DHCP information

Returns json data about all infromation from the DHCP server

- URL
    `/config/dhcp`

- Method:
    `POST`

- Header Params:
    x-access-token=`TOKEN jwt`

- Body Params

``` json
{
	"broadcast" : "10.0.0.255"
}
```

- Success Response:
    - code: 200
    - Body :
    `
        {
            "broadcast" : "10.0.0.255"
        }
    `


## Log information

Returns json data about log plugins/proxies 

- URL
    `/logger/:<filename>?page=<id>`
- Method:
    `GET`
- Header Params:
    x-access-token=`TOKEN jwt`
- URL Params
    - Required:
        `:filename [string]`
        `?page=[integer]`
    - Example:
        `/logger/sniffkin3?page=0`
    - Filenames:
        - sniffkin3
        - pumpkinproxy
        - captiveportal
        - responder3

- Success Response:
    - code: 200
    - Body:
    `
    {
    "data": {
        "current_page": 0,
        "items": [],
        "limt_view": 10,
        "total_count": 0,
        "total_pages": 1
    }
    }
    `

## Send CLI command 

execute command line cli  

- URL
    `/commands/:{command;}`
- Method:
    `GET`
- Header Params:
    x-access-token=`TOKEN jwt`
- URL Params
    - Required:
        `command=[string]`
    - Example:
        `/commands/ap; mode`

- Success Response:
    - code: 200
    - Body:
    `
    [
        "command output",
    ]
    `

## Proxies information

Returns json data about status of proxies 

- URL
    `/proxies`
- Method:
    `GET`
- Header Params:
    x-access-token=`TOKEN jwt`

- Success Response:
    - code: 200
    - Body:
    `
    {
    "captiveflask": "false",
    "noproxy": "false",
    "pumpkinproxy": "true",
    "pumpkinproxy_config_port": "8080"
    }
    `

## Proxies information

Returns json data about status of proxies 

- URL
    `/proxies/:{proxies_name}`
- Method:
    `GET`
- Header Params:
    x-access-token=`TOKEN jwt`
- URL Params
    - Optional:
        `proxies_name=[string]`
    - Example:
        `/proxies/pumpkinproxy`

- Success Response:
    - code: 200
    - Body:
    `
    {
    "captiveflask": "false",
    "noproxy": "false",
    "pumpkinproxy": "true",
    "pumpkinproxy_config_port": "8080"
    }
    `

## Proxies settings

Change the value of proxy activated 

- URL
    `/proxies`
- Method:
    `POST`
- Header Params:
    x-access-token=`TOKEN jwt`

- Body Params
    ``` json
    {
        "proxy_name" : "false"
    }
    ```
- Success Response:
    - code: 200
    - Body:
    `
    {
        "proxy_name" : "false"
    }
    `

## Plugins information

Returns json data about status of plugins and config path 

- URL
    `/plugins`

- Method:
    `GET`

- Header Params:
    x-access-token=`TOKEN jwt`

- URL Params
    - Optional:
        `plugin_name=[string]`
    - Example:
        `/plugins/responder3`

- Success Response:
    - code: 200
    - Body:
        ` json
        {
        "responder3": "false",
        "responder3_config": "/.config/wifipumpkin3/config/plugins/responder3/examples/config.py",
        "sniffkin3": "true"
        }
        `

## Plugins settings

Change the value of plugin activated 

- URL
    `/plugins`

- Method:
    `POST`

- Header Params:
    x-access-token=`TOKEN jwt`

- Body Params

``` json
{
    "plugin_name" : "false"
}
```

- Success Response:
    - code: 200
    - Body:
    `
    {
        "plugin_name" : "false"
    }


## Subplugins Information

Returns json data about value  of subplugins of proxies or plugins

- URL
    `/:{name}/plugins`

- Method:
    `GET`

- Header Params:
    x-access-token=`TOKEN jwt`

- URL Params
    - Required:
        `name=[string]` name of plugin or proxy to get config
    - Example:
        `/pumpkinproxy/plugins`

- Success Response:
    - code: 200
    - Body:
        `{
  "beef": "true",
  "downloadspoof": "true",
  "html_inject": "false",
  "js_inject": "true",
  "no-cache": "true",
  "replaceImages": "false",
  "set_beef": [
    {
      "url_hook": "http://192.168.0.103:3000/hook.js"
    }
  ],
  "set_downloadspoof": [
    {
      "backdoorExePath": "plugins/extension/tmp/exe/binary.exe"
    },
    {
      "backdoorPDFpath": "plugins/extension/tmp/pdf/binray_23.pdf"
    },
    {
      "backdoorWORDpath": "plugins/extension/tmp/doc/backdoor.doc"
    },
    {
      "backdoorXLSpath": "plugins/extension/tmp/xls/face.xls"
    }
  ],
  "set_html_inject": [
    {
      "content_path": "file_test.html"
    }
  ],
  "set_js_inject": [
    {
      "url": "http://facebook.com/foo.js"
    }
  ],
  "set_replaceImages": [
    {
      "path": "docs/10000.png"
    }
  ]
} `


## Settings Subplugins

change the value of subplugins of proxies or plugins

- URL
    `/:{name}/plugins`

- Method:
    `POST`

- Header Params:
    x-access-token=`TOKEN jwt`

- URL Params
    - Required:
        `name=[string]` name of plugin or proxy to get config
    - Example:
        `/pumpkinproxy/plugins`

- Body Params

``` json
{
  "beef": "true",
  "downloadspoof": "true",
  "html_inject": "false",
  "js_inject": "true",
  "no-cache": "true",
  "replaceImages": "false",
  "set_beef": [
    {
      "url_hook": "http://192.168.0.103:3000/hook.js"
    }
  ],
  "set_downloadspoof": [
    {
      "backdoorExePath": "plugins/extension/tmp/exe/binary.exe"
    },
    {
      "backdoorPDFpath": "plugins/extension/tmp/pdf/binray_23.pdf"
    },
    {
      "backdoorWORDpath": "plugins/extension/tmp/doc/backdoor.doc"
    },
    {
      "backdoorXLSpath": "plugins/extension/tmp/xls/face.xls"
    }
  ],
  "set_html_inject": [
    {
      "content_path": "file_test.html"
    }
  ],
  "set_js_inject": [
    {
      "url": "http://facebook.com/foo.js"
    }
  ],
  "set_replaceImages": [
    {
      "path": "docs/10000.png"
    }
  ]
}
```

- Success Response:
    - code: 200
    - Body:
        `{
  "beef": "true",
  "downloadspoof": "true",
  "html_inject": "false",
  "js_inject": "true",
  "no-cache": "true",
  "replaceImages": "false",
  "set_beef": [
    {
      "url_hook": "http://192.168.0.103:3000/hook.js"
    }
  ],
  "set_downloadspoof": [
    {
      "backdoorExePath": "plugins/extension/tmp/exe/binary.exe"
    },
    {
      "backdoorPDFpath": "plugins/extension/tmp/pdf/binray_23.pdf"
    },
    {
      "backdoorWORDpath": "plugins/extension/tmp/doc/backdoor.doc"
    },
    {
      "backdoorXLSpath": "plugins/extension/tmp/xls/face.xls"
    }
  ],
  "set_html_inject": [
    {
      "content_path": "file_test.html"
    }
  ],
  "set_js_inject": [
    {
      "url": "http://facebook.com/foo.js"
    }
  ],
  "set_replaceImages": [
    {
      "path": "docs/10000.png"
    }
  ]
} `