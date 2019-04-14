---
Title: Automated Wordpress Upgrades
Date: 2015-12-31
---

One of my biggest pain points over the past few months has been keeping up with WordPress updates and performing upgrades/changes in maintenance windows which usually occur early in the morning (I’m not a morning person). Typically the solution here is for me to script out what I need to do, perform it on a test box, and then schedule it with [at](http://linux.die.net/man/1/at). Unfortunately WordPress would usually need some interactions on the web interface (click to finish a database upgrade or click to upgrade the network) after just simply replacing the files which I couldn’t script away, at least not with the tools I had at my disposal. This led me on a hunt for a better way.

## The Better Way

It turns out that my hunt didn’t take long at all. I came across a project called [WP-CLI](http://wp-cli.org/) which did everything I could have ever wanted. Once I had that setup it was just a simple matter of writing an extremely simple bash script to update the tool, update WordPress, and then update all of it’s pieces.

## The Script

The final product ended up at a whopping twenty lines with whitespace and comments:

```bash
#!/bin/bash

#Write Output to Syslog for consumption by logging server
exec 1> >(logger -s -t $(basename $0)) 2>&1

#Update everything via the wp-cli tool (http://wp-cli.org/)
/usr/local/sbin/wp cli update --yes #Update the tool its self first
/usr/local/sbin/wp core upgrade --path='/var/www/html' --quiet
/usr/local/sbin/wp core update-db --path='/var/www/html' --quiet
/usr/local/sbin/wp core update-db --network --path='/var/www/html' --quiet
/usr/local/sbin/wp plugin update --all --path='/var/www/html' --quiet
/usr/local/sbin/wp theme update --all --path='/var/www/html' --quiet

#Remove unwanted files from upgrade
rm /var/www/html/readme.html
rm /var/www/html/wp-config-sample.php

#Ensure that the ownership and SELinux are happy
chown -R nginx:nginx /var/www/html/
restorecon -Rv /var/www/html/
```

## The Best Way

Yes, I know this isn’t the best way, the best way would be to make the box stateless and version control WordPress in Git or something else. We’re working towards that but this is a “quick and dirty” solution that will hold my team and I over until we get there. Something simple like this also works great for small, personal installations outside of the enterprise.
