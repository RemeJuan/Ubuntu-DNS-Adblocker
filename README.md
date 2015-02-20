# Ubuntu-DNS-Adblocker
Blacklist files for use with Bind9 when setting up a basic ad blocker with support for both Desktop and Mobile

This is based off the [Lituxx.com guide](http://www.lituxx.com/website/en/2014/03/block-ads-on-your-entire-network/). I made a few updates based on a small bug I spotted and have added in the mobile blacklist used by [AdAway](https://sufficientlysecure.org/index.php/adaway/) and modified for use by Bind9.

## Setup

```
sudo apt-get install bind9
```

And modify **/etc/bind/named.conf.options**, adding your favorite DNS servers. Here is mine, which must in no way encourage you to use the same DNS servers. Please note I have been told Google’s DNS is said to track you and lie. I personally stood by namebench’s results.

```
forwarders {
  8.8.4.4; // Google
  8.8.8.8; // Google
};
```

I’d recommend using [namebench](https://code.google.com/p/namebench/) so as to find the best DNS couple for yourself (it is best to avoid repeating the test for the results get skewed after the first go). It could allow you to gain precious milliseconds during your internet browsing. Also you should know that configured as such, the BIND server will cache results from previous requests, effectively giving zero milliseconds DNS queries on websites you visit on a regular basis.

Next we need to create 3 files

```
sudo nano /etc/bind/blacklist-yoyo
sudo nano /etc/bind/blacklist-adaway
sudo nano /etc/bind/blacklist-custom
```
These 3 files are included in this repo, the first 2 have been modified from thier source:
* blacklist-yoyo is sourced from http://pgl.yoyo.org, however the generated files needs a path update, so if you wish to update directly from source, run the following command after:
```
cat /etc/bind/blacklist-yoyo | sed 's/{/IN {/g' > sedtempfile && mv sedtempfile /etc/bind/blacklist-yoyo
cat /etc/bind/blacklist-custom | sed 's:null.zone.file:/etc/bind/null.zone.file' > sedtempfile && mv sedtempfile /etc/bind/blacklist-custom
```
* blacklist-adaway is source from the host files used by the AdAway android app, this is if you wish to use this DNS server on your mobile devices.
* The 3rd file is for you to add your own custom hosts that are not included in the initial 2, to make future updates of the original easier.


**Warning:** please have a quick look at what is in the file we just created, for any domain there will in effect be inaccessible from your network. You could of course choose to simply add a couple of your choosing, just follow the syntax described above.

Add these lines to **/etc/bind/named.conf:**
```
include "/etc/bind/blacklist-yoyo";
include "/etc/bind/blacklist-adaway";
include "/etc/bind/blacklist-custom";
```

Then create a file named **/etc/bind/null.zone.file** and paste this to it, replacing the IP with the server’s on the network.
```
$TTL    86400   ; one day
 
@       IN      SOA     ns.localhost. admin.localhost. (
            2002061000       ; serial number
            28800   ; refresh  8 hours
            7200    ; retry    2 hours
            864000  ; expire  10 days
            86400 ) ; min ttl  1 day
            
                        NS      ns.localhost
                        A       192.168.1.17 ; your ip
@               IN      A       192.168.1.17 ; your ip
*               IN      A       192.168.1.17 ; your ip
```

Lastly, we need to restart the BIND server:
```
service bind9 restart
```

And all you have to do now to enjoy an addless network is add your server’s IP to your router’s settings, as here.

But be careful to input a secondary and third DNS, in case your BIND install fails. I’d recommend adding the first two results from the namebench test.

It might take a while for your devices to register the change, so you might want to do it yourself.
