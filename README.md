# pump up ur nginx to the MAX!
## intro
### why
So basically nginx is a good webserver and all but when your site gets lots of traffic you'll quickly find during your user-initiated test cycle that, oh no, you're getting error 500s! and bad gateways! and timeouts! maybe not timeouts, nginx will return an internal server error if shit gets fucked.

Out of the box on a system like debian 12 with nginx it does have some limits which you need to increase to prevent this from happening, or you'll find out the hard way, a few tims even, because these limits exist in several places.

This document shows you every limit you should increase if you get lots of traffic and you find shit dying for no reason.

Also install netdata and setup webhook monitoring to send you notifications on something like Matrix or Discord, it will tell you about shit you didn't even know existed. Don't underestimate it, I don't like tying up servers into botnets like Netdata Cloud but it's genuinely good utility to have.

### the rememberence
#### WARNING
you must remember why these limits exist in the first place. in this guide, i'm going to show you how to pump them right up.

these limits exist to stop your entire system as a whole from crashing when there is too much traffic. as much as of an annoyance they are, they exist for a reason.

use different utilities to gague where bottlenecks are and increase your limits gracefully. if you run your backend application on the same machine, you'll need to account for this too, it will run poorly if nginx is allowed to use resources the application needs. in some cases, the backend application can be fairly light so you can go high without issues.

just do your own testing. i'm not responsible for your fuckups and if you have to pay for remote hands don't expect me to get my wallet out, bro!

### how to be stupid
I'm calling this how to be stupid because pumping up your limits to a value you don't understand if it's right for your hardware will work. This works for me and exists so it's easy to find. You do you, as said above.

#### System wide limits
So this isn't strictly required, I do this as a "just in case" because it's supposedly only for user sessions, which nginx is not, but if you start an application in the user space this could be handy.
Open ``/etc/security/limits.conf`` with your editor.
Add the following:
```
*      soft      nofile      250000
*      hard      nofile      500000
```

#### Increase service open file limit
So systemd has this cool default open file limit which we're immediately going to veto because I don't like it. That's why. Run ``systemctl edit nginx`` and between the specified lines, add the following:

```
[Service]
LimitNOFILE=190000
```

and then run:

```
systemctl daemon-reload
systemctl nginx reload
```

#### Increase NGINX limits
Nginx also has it's own limits. Why? I guess it's in case there's no service limit or something, idfk.
Open ``/etc/nginx/nginx.conf`` with your editor. Under ``worker_processes``, add:
```
worker_rlimit_nofile 190000;
```
Set the 190000 to the same as you set on the ``systemctl ...`` command. Ideally you need this set along with the service change as this cannot completely effect open file limits. Why does it exist? God knows.

Within the events block, change worker_processes to 65535, and add the following, within the events block, if it isn't already present:
```
multi_accept on;
use epoll;
```
and then test nginx with ``nginx -t`` and apply changes by running ``systemctl nginx reload``.

#### Something broke!
Running ``nginx -t`` will tell you what's wrong. Don't be an idiot. Understanding the error you see if there is one is important to knowing how to fix it. Also, Google is your best friend, and you can probably just use some AI tool to fix it for you. Win-win!
