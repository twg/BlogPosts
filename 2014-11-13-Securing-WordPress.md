_The target audience for this article is someone that is comfortable with Linux, WordPress, and wants to really lock things down._

Due to it’s popularity and the poor quality of many plugins and themes, WordPress has become a common and easy target for malicious users. We have learned some of these lessons first hand and felt it would be worth sharing our findings with the community.

# LockDown - The Basics

##### Disable File Modifications
By default, WordPress admin users can edit PHP files and install any plugin they want. Giving users these abilities is often unnecessary and dangerous. One of the first things you can do is edit your wp-config.php file and add `define('DISALLOW_FILE_MODS', true);`. This will prevent users from installing plugins or editing themes, and will also hide the UI elements associated with these actions. Editing themes to add malicious code is a very common attack vector.

##### Remove Unused Themes
There is no reason too keep unused themes around. Remove them to ensure that there is no way for malicious users to take advantage of known exploits.

##### Remove Unused Plugins
The same goes for plugins. Remove anything you are not using and take a few minutes to Google around for known exploits of the plugins you are using. It's worth taking a few minutes to look through the pull request history of the [MetaSploit Github repo](https://github.com/rapid7/metasploit-framework/pulls?q=is%3Apr+wordpress+is%3Aclosed) to see if there are any known security issues.

##### Setup Professional Deployments
You should be able to deploy your code from the command line with a single command. We've used [Capistrano](http://capistranorb.com/) with great success.

##### Keep Your Full WordPress Install in Source Control
In the past we used to only use source control for versioning our theme. After having a few bad experiences, we now version the entire WordPress install, which includes plugins and themes. This way, if a site is ever comprimised or you suspect a malicious user has somehow edited core WordPress files, a simple one-line deploy will reset everything back to how it should be. This also gives the developer full control over the plugins used on the site, allowing you to keep them up-to-date as needed.

##### Enforce Password Strengh
This seems like a basic one, but weak passwords are often the easiest way for malicious users to gain access to your site. Using a plugin like [Force Strong Passwords](https://wordpress.org/plugins/force-strong-passwords/) will help avoid a situation where a user uses a weak password like "_password123_".

##### Errbit
Malicious attacks on your WordPress install will often generate errors. Using [Airbrake](https://airbrake.io/) or [Errbit](https://github.com/errbit/errbit) to catch errors will allow you to have better insights into your deployment, but will also alert you to any suspicicous errors that occur. The easiest way to get this working is to use [Errbit-PHP](https://github.com/flippa/errbit-php).

##### Restrict Sensitive Folders
Deployments often involve pulling a Github repoistory down and symlinking sensative configuration folders. Most deployments involve symlinking logs and config folders, but since WordPress does not have any notion of a public folder, Capistrano or some hosts may end up symlinking these sensative folders into a publiclially asseccible location. Ensure that you cannot access these folders and also make sure that any `.git` folders are restricted as well.

##### Fail2Ban
The best way to stop brute force attacks on your WordPress install is to add Fail2Ban, which blocks IP's that make clusters of failed requests. To get things setup, you'll need to:

1. Install Fail2Ban on your Linux install
2. Install the simple [WP Fail2Ban Plugin](https://wordpress.org/plugins/wp-fail2ban/)
3. Ensure that Fail2Ban is writing to your syslog when you make failed login requests _(most often in `/var/logs/auth.log`)_
4. Copy over the files mentioned in the [WP Fail2Ban installation instructions](https://wordpress.org/plugins/wp-fail2ban/installation/) and update your `jail.local` file accordingly.
