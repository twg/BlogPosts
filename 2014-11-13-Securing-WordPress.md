_The target audience for this article is someone that is comfortable with Linux, WordPress, and wants to really lock things down._

Due to it’s popularity and the poor quality of many plugins and themes, WordPress has become a common and easy target for malicious users. We have learned some of these lessons first hand and felt it would be worth sharing our findings with the community.

# LockDown - The Basics

##### Disable File Modifications
By default, WordPress admin users can edit PHP files and install any plugin they want. Giving users these abilities is often unnecessary and dangerous. One of the first things you can do is edit your wp-config.php file and add `define('DISALLOW_FILE_MODS', true);`. This will prevent users from installing plugins or editing themes, and will also hide the UI elements associated with these actions. Editing themes to add malicious code is a very common attack vector.

#####Remove Unused Themes
There is no reason too keep unused themes around. Remove them to ensure that there is no way for malicious users to take advantage of known exploits.

#####Remove Unused Plugins
The same goes for plugins. Remove anything you are not using and take a few minutes to Google around for known exploits of the plugins you are using. It's worth taking a few minutes to look through the pull request history of the [MetaSploit Github repo](https://github.com/rapid7/metasploit-framework/pulls?q=is%3Apr+wordpress+is%3Aclosed).

