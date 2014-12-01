_Authored by [Bill Li](https://github.com/billxinli) and [Emerson Lackey](https://github.com/Emerson), feel free to add your thoughts over at [Github](https://github.com/twg/BlogPosts/blob/master/2014-11-13-Securing-WordPress.md)_

_**Note:** The target audience for this article is someone that is comfortable with Linux, WordPress, and does not want to rely on plugins to manage security._

Due to it's popularity and the poor quality of many plugins and themes, WordPress has become a common and easy target for malicious users. We have learned some of these lessons first hand and felt it would be worth sharing our findings with the community. Here are a few of the basic things you can do to start securing your WordPress install:

##### Disable File Modifications
By default, WordPress admin users can edit PHP files and install any plugin they want. Giving users these abilities is often unnecessary and dangerous. One of the first things you can do is edit your wp-config.php file and add `define('DISALLOW_FILE_MODS', true);`. This will prevent users from installing plugins or editing themes, and will also hide the UI elements associated with these actions. Editing themes to add malicious code is a very common attack vector.

##### Remove Unused Themes
There is no reason too keep unused themes around. Remove them to ensure that there is no way for malicious users to take advantage of known exploits.

##### Remove Unused Plugins
The same goes for plugins. Remove anything you are not using and take a few minutes to Google around for known exploits of the plugins you are using. It's worth taking a few minutes to look through the pull request history of the [MetaSploit Github repo](https://github.com/rapid7/metasploit-framework/pulls?q=is%3Apr+wordpress+is%3Aclosed) to see if there are any known security issues with your plugins.

##### Setup Professional Deployments
You should be able to deploy your code from the command line with a single command. We've used [Capistrano](http://capistranorb.com/) with great success, and have more recently followed the techniques outlined in EngineYard's "[WordPress in the Cloud](https://blog.engineyard.com/2014/wordpress-in-the-cloud-part-1)" blog posts.

##### Keep Your Full WordPress Install in Source Control
In the past we used to only keep our themes in version control. After having a few bad experiences, we now version the entire WordPress install, which includes plugins and themes. This way, if a site is ever compromised or you suspect a malicious user has somehow edited core WordPress files, a simple one-line deploy will reset everything back to how it should be. This also gives the developer full control over the plugins used on the site, allowing you to keep them up-to-date as needed.

##### Errbit
Malicious attacks on your WordPress install will often generate errors. Using [Airbrake](https://airbrake.io/) or [Errbit](https://github.com/errbit/errbit) to catch errors will allow you to have better insights into your deployment, and will also alert you to any suspicious errors that occur. The easiest way to get this working is to use [Errbit-PHP](https://github.com/flippa/errbit-php).

##### Restrict Sensitive Folders
Deployments often involve pulling a Github repository down and symlinking sensitive configuration folders into their expected locations. Since WordPress does not have any notion of a public folder, Capistrano or some hosts may end up symlinking these sensitive folders into a publicly accessible location. Ensure that you cannot access these folders and also make sure that any `.git` folders are restricted as well. 

##### Enforce Password Strength
This seems like a basic one, but weak passwords are often the easiest way for malicious users to gain access to your site. Using a plugin like [Force Strong Passwords](https://wordpress.org/plugins/force-strong-passwords/) will help avoid a situation where a user uses a weak password like "_password123_". 

##### Fail2Ban
Another great method to stop brute force attacks on your site is to add [Fail2Ban](https://github.com/fail2ban/fail2ban), which temporarily firewalls IP addresses that make a concurrent number of similar requests. Fail2Ban is often setup to monitor your `access.log` file, which contains important information about the requests being made to your application. For example:

- A successful login occurs when we see a `POST` request to `wp-login.php` with a status code of `302`
- A failed login occurs when we see a `POST` request to `wp-login.php` with a status code of `200`. If we notice the same IP making hundreds of failed login requests, we should probably ban them.

To get started, you'll need to:

1. Install Fail2Ban on your *nix install
2. Install iptables if your version of *nix does not come with it installed by default
3. Configure a Fail2Ban filter to find clusters of failed login requests
4. Configure a jail action to handle the ban
5. Test the setup

In our case, we used [Nginx](http://nginx.org/) and Gentoo _(EngineYard's default linux distro)_ to host our WordPress site and took the following steps:

**1. Install Fail2Ban and iptables via `emerge`**

    # If you're not using Gentoo, you probably install with apt-get or yum
    sudo emerge --ask net-analyzer/fail2ban
    sudo emerge --ask iptables

**2. Add regex that matches failed logins to `/etc/fail2ban/filter.d/wordpress.conf`**

    [Definition]
    failregex = <HOST>.*POST.*(wp-login\.php).* 200
    ignoreregex =

**3. Append a jail configuration to `/etc/fail2ban/jail.conf`**

    [wordpress]
    enabled = true
    filter = wordpress
    action = iptables-multiport[name=NoAuthFailures, port="http,https"]
    logpath = /path/to/wordpress/access.log
    bantime = 120 (This is in seconds)
    maxretry = 10 (Number of retries allowed)

**4. Perform a test with the filter file**

In this step we will check to see if everything is configured correctly using Fail2Ban's built in test:

    fail2ban-regex /path/to/wordpress/access.log /etc/fail2ban/filter.d/wordpress.conf

The output of this command gives you a breakdown of how your Fail2Ban configuration would work. In our example we saw something like this:

    Running tests
    =============

    Use regex file : /etc/fail2ban/filter.d/wordpress.conf
    Use log file   : /path/to/wordpress/access.log


    Results
    =======

    Failregex
    |- Regular expressions:
    |  [1] <HOST>.*POST.*(wp-login\.php).* 200
    |
    `- Number of matches:
       [1] 7 match(es)

    Ignoreregex
    |- Regular expressions:
    |
    `- Number of matches:

    Summary
    =======
    
    Addresses found:
    [1]
        165.x.x.x (Thu Nov 27 03:11:43 2014)
        165.x.x.x (Thu Nov 27 03:20:09 2014)
        192.x.x.x (Thu Nov 27 07:40:41 2014)
        165.x.x.x (Thu Nov 27 10:00:34 2014)
        165.x.x.x (Thu Nov 27 10:00:39 2014)
        165.x.x.x (Thu Nov 27 11:05:46 2014)
        165.x.x.x (Thu Nov 27 11:05:57 2014)

    Date template hits:
    0 hit(s): MONTH Day Hour:Minute:Second
    0 hit(s): WEEKDAY MONTH Day Hour:Minute:Second Year
    0 hit(s): WEEKDAY MONTH Day Hour:Minute:Second
    0 hit(s): Year/Month/Day Hour:Minute:Second
    0 hit(s): Day/Month/Year Hour:Minute:Second
    0 hit(s): Day/Month/Year Hour:Minute:Second
    22681 hit(s): Day/MONTH/Year:Hour:Minute:Second
    0 hit(s): Month/Day/Year:Hour:Minute:Second
    0 hit(s): Year-Month-Day Hour:Minute:Second
    0 hit(s): Year.Month.Day Hour:Minute:Second
    0 hit(s): Day-MONTH-Year Hour:Minute:Second[.Millisecond]
    0 hit(s): Day-Month-Year Hour:Minute:Second
    0 hit(s): TAI64N
    0 hit(s): Epoch
    0 hit(s): ISO 8601
    0 hit(s): Hour:Minute:Second
    0 hit(s): <Month/Day/Year@Hour:Minute:Second>

    Success, the total number of match is 7

**5. Start Fail2Ban**

Once you've confirmed that everything is setup, you need to actually start the Fail2Ban daemon.

    sudo /etc/init.d/fail2ban
    
    or
    
    sudo service fail2ban restart


**6. Monitor Fail2Ban logs and real testing**

Fail2Ban logs all banning and unbanning of IPs in `/var/log/fail2ban.log`. We simulated a brute force attack, by logging in ten times with the wrong username and password.

Our Fail2Ban daemon logged the following results:

    2014-11-26 22:10:01,782 fail2ban.server : INFO   Changed logging target to /var/log/fail2ban.log for Fail2ban v0.8.6
    2014-11-26 22:10:02,373 fail2ban.filter : INFO   Log rotation detected for /path/to/wordpress/access.log
    2014-11-26 22:10:07,379 fail2ban.filter : INFO   Log rotation detected for /path/to/wordpress/access.log
    2014-11-27 12:08:53,992 fail2ban.actions: WARNING [wordpress] Ban our.ip.here.!
    2014-11-27 12:10:54,191 fail2ban.actions: WARNING [wordpress] Unban our.ip.here.! 
    ... snip ...
    
We were banned for a total of two minutes.
