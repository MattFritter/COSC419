# COSC 419: Topics in Computer Science
# Fall 2018 - Lab 5

In this lab, we'll be securing access to our servers and learning how to use monitoring software packages to help deal with bad actors automatically.

## Table of Contents
- [Changing Your SSH Port](#ssh-port)
- [Installing Fail2Ban](#fail2ban)
- [Getting Rid of Root](#no-root)
- [Intro to Mod-Security](#mod-security)

<a name="ssh-port"></a>
## Changing the SSH Port (3 marks)

Port 22 is considered the standard port for a Secure Shell (SSH) connection. Because of this, it is extremely common for automated brute forcing tools to simply default to a port 22 connection when trying to perform an SSH intrusion. By changing the port to a non-standard port number, we can reduce the number of automated intrusion attempts that we have to deal with.

First, edit your ```sshd``` configuration file, which is located at ```etc/ssh/sshd_config```. You should see a line that says ```#Port 22```. Uncomment the line by removing the '#' character, and change 22 to 762. Technically port 762 is dedicated for the quotad service, but we aren't using that service. After you're done, save and close the file.

Now, we just need to inform the SELinux system that we have a new SSH port to manage:

	semanage port -a -t ssh_port_t -p tcp 762
	
Finally, we'll restart the sshd service to enable our changes: ```systemctl restart sshd```. Now, you should be able to exit your terminal session. Try reconnecting via port 22; you should receive an immediate connection closure. Then, try connecting using port 762 - you should be able to log in as previously.

<a name="fail2ban"></a>
## Installing Fail2Ban (6 marks)

While changing our SSH port will cut down on most automated intrusion attempts, it is still possible to identify the SSH port via a port scan, and then perform a bruteforce attack. For this reason, we'll install and configure Fail2Ban, a system that identifies and bans malicious IPs.

First, we'll install and enable the Fail2Ban package:

	yum install fail2ban
	systemctl enable fail2ban
	
If you receive an error saying that the Fail2Ban package is not available, double check that you have properly installed ```epel-release``` - refer back to Lab 1 for details.

Now that Fail2Ban is installed, we can configure it. Referring back to the example slide from Lecture 5 and the configuration basics slide, edit the configuration file at ```/etc/fail2ban/jail.local``` to do the following:

* Ban malignant users for ten minutes
* Use a finding time frame of five minutes
* Allow up to five attempts before banning
* Use the iptables-multiport banaction
* Enable the [sshd] jail
* Use port 762 for the [sshd] jail

Once you're done with the configuration file, save and close it, then restart the Fail2Ban service. Fail2Ban should now enabled and enforcing. Test this by trying to log in to your server via SSH and entering an incorrect password five times. Any further connection attempts should result in being unable to connect to the server, until the 10 minute cooldown is finished.

While the 10 minute cooldown strikes a good balance between punishing an automatic bot while allowing a legitimate, but error-prone user to still access the system after a short while. However, we should also have a long-term banning option for users who prove to be a nuisance.

In order to combat this, we'll create a second Fail2Ban jail. This jail will track number of failed attempts over a longer period of time, and place a longer ban on users who are making excessive failed attempts throughout the day.

First, we'll re-open the ```/etc/fail2ban/jail.local``` file. Then, below the *[sshd]* entry in the file, add an entry called *[ssh-longterm]*. For this jail, assign the following values:

* Ban malignant users for 7 days
* Use a finding time frame of 24 hours
* Allow up to twenty attempts before banning
* Enable the [ssh-longterm] jail
* Use port 762 for the [ssh-longterm] jail

You'll also need to a couple additional entries to your *[ssh-longterm]* jail. First, add an entry: ```filter = sshd-aggressive```. This enables a more aggressive filter for parsing the SSH logs for intrusion attempts. Then, two entries: ```logpath = %(sshd_log)s``` and ```backend = %(sshd_backend)s```. These entries help Fail2Ban identify the location of the SSH log files and backend for retrieving login data.

Now, restart your Fail2Ban service, and run the following command: ```fail2ban-client status```. If everything went correctly, it should return that you have two jails:

<img src="https://i.imgur.com/MpSSYxP.png" width="50%"/>

<a name="no-root"></a>
## Getting Rid of the Root Account (2 marks)

This is fairly straightforward. Using the instructions on slides 43 and 44 from Lecture 5, create a new account, give it *superuser* privileges, and then disable SSH login to the ```root``` account. Switch to your new account using ```su```, and use ```sudo``` when performing actions that require root access.

Close your terminal connection, re-open it, and try to login to the 'root' account. You should find that even if you enter the correct password, you are unable to login to the account.

<a name="mod-security"></a>
## Getting Acquainted with Mod-Security (4 marks)

ModSecurity is an open-source software package that provides real-time web firewall protection. ModSecurity is a rules-based system that checks incoming web requests for potential suspicious activity by matching against a large set of rules.

In this exercise, we'll update our ModSecurity install (installed in Lab 4) with the latest release of the OWASP CRS ruleset, which is considered a standard for the ModSecurity package.

First, we move to our Apache ModSecurity configuration folder:

	cd /etc/httpd/modsecurity.d

Now, we can use Git to clone the latest rules into this folder:

	git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
	
Rename the ```crs-setup.conf.example``` to ```crs-setup.conf```. Then, go and navigate to your base Apache configuration file and edit it: ```nano /etc/httpd/conf/httpd.conf```. At the very bottom of the file: add the following lines:

	<IfModule security2_module>
			Include modsecurity.d/owasp-modsecurity-crs/crs-setup.conf
			Include modsecurity.d/owasp-modsecurity-crs/rules/*.conf
	</IfModule>

This will load the rulesets and the OWASP configuration file into ModSecurity. Save and close the file, the restart Apache.

If you visit your website now, you'll probably notice that every link now takes you to an error page. This is because ModSecurity rules usually err on the side of being too strict, and something about our requests (quite possibly a session token) is triggering the OWASP rulesets. In order to figure out what's going on, we'll go to ```/etc/conf.d/mod_security.conf```.

Find the line that says ```SecRuleEngine On``` near the top of the file. Change this to ```SecRuleEngine DetectionOnly```. Now, ModSecurity will log when rules are broken, but not block or drop the connection.

Now, if we go to our log at ```/etc/httpd/logs/modsec_audit.log```, we should be able to scroll to the bottom of the file and find our rule that is causing issues. In this case, the rule is:

	REQUEST-920-PROTOCOL-ENFORCEMENT
	
Which appears to be tripping because we're asking for an IP address instead of a domain name. We'll disable it for now. Go to ```/etc/httpd/modsecurity.d/owasp-modsecurity-crs/rules```, and find the corresponding rule file:

	REQUEST-920-PROTOCOL-ENFORCEMENT.conf
	
Then move it to a new filename so that it isn't picked up by the Apache config:

	mv REQUEST-920-PROTOCOL-ENFORCEMENT.conf REQUEST-920-PROTOCOL-ENFORCEMENT.conf.disable
	
Now, go back to your ```/etc/conf.d/mod_security.conf```, change the SecRuleEngine back to ```On```, and then restart Apache. You should now be able to visit web pages again without getting an error.

This wraps up Lab 5. ModSecurity is a large tool with many features, but the primary concern is updating your rulesets, and then testing with your website to ensure that you are not getting false positives. Rule sets require a great deal of tweaking, and are an area of constant development.

