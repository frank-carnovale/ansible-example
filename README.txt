"Ansible" Technical Exercise
-----------------------------------

Summary of Required Deliverables:
=================================

- Produce some code to automate the build of a server

	This is provided as an "Ansible" project bundled into a zip file.

- Provide clear instructions for running your code

	1. Unbundle this package on a machine with Ansible installed
	2. Create any "latest Ubuntu" instances on EC2 and list them in "hosts" file if required
	2. Confirm key-pair authentication valid for all "new" and "old" EC2 instances
	3. Run this command: $ ansible-playbook webservers.yml

- State clearly any assumptions made (either in code comments or as a separate README file)

	1. Assumed that the preferred Linux distribution for new instances continues to be "latest Ubuntu".
	2. Assumed that the installation of the required node.js app onto a
           new instance is outside scope of this assignment, hence final "Welcome" page
           test against a NEW server is expected to fail with a 503 but should still work
           fine for the (upgraded) legacy server.
	3. Assumed that pros/cons of an "automatic unattended upgrade" strategy has been discussed with stakeholders,
	   and is considered an appropriate method for applying security patches
	4. Assumed that the web development team will now manage their Apache
	configuration changes in this project, in the directories
	'roles/webserver/files/etc/apache2/{sites,conf}-{enabled,available}'.
	This is not really an ideal solution.. in future, this sub-area should be moved to a
	distinct Git repository and this Ansible job should be modified to clone and
	deploy that repo).

- Execute your solution against the server provided

	A sample transcript is provided showing the output of a run against a
	pool of servers including the given legacy server and one newly created server.
	Summary of URLs:
	1. Legacy server    http://ec2-34-250-227-208.eu-west-1.compute.amazonaws.com
	2. New server       http://ec2-34-240-13-232.eu-west-1.compute.amazonaws.com
	
- Use source control

	This package is a self-contained git repository.
	e.g. A "git log" command will show the history of the project.

User Story Acceptance Criteria:
===============================

- Flaws with the existing server setup are highlighted
	
	1. We provide below a summary of our initial observations and determinations on the existing server.
	2. An Ansible "--check" run against the server's initial state would highlight any actions not-yet-applied.

- Tool produced to restore the servers into a satisfactory state (i.e. fix the issues), and handle future Apache configuration changes

	1. The given Ansible project has been designed to accommodate a
	'files' section in which we manage ONLY the configuration files deemed to be
	under management of the web development team.  The project will ensure that
	these and only these configurations (primarily the virtual host config) are in
	place on each server.  Future config changes will be implemented by adjusting
	the isolated config files in this project, and re-running 'ansible-playbook'
	as instructed above.

Observations
============

Problem:
	The server under examination is well behind in security updates, and does not
	appear to be under any security management scheme, hence presents a growing risk as time progresses.

Action:
	Ensure Ubuntu's 'unattended-upgrades' package installed; and enable by writing following file:
	----------------
	# /etc/apt/apt.conf.d/20auto-upgrades
	APT::Periodic::Update-Package-Lists "1";
	APT::Periodic::Unattended-Upgrade "1";
	-----------------
	(Reference: https://help.ubuntu.com/community/AutomaticSecurityUpdates)
	Note, unattended security upgrade introduces theoretical and low risk that system behaviour may
	change unexpectedly.  Discuss pros/cons with stake-holders and if
	necessary, test security patches on a non-production machine first.

Problem:
	VLE page requires a login but is not HTTPS.  Also, any future pages on this site may
	contain sensitive content.  Clear-text HTTP should be phased out.

Action:
	Implement a loadbalancer-to-server certificate scheme.

Problems:
	- /opt/apache install area has all 777 file modes

	- Apache processes are running as root vulnerability

	- Apache logs do not appear to be under any logrotate scheme

	- 'Classic' apache artifacts such as cgi-bin, manual, server-info etc have
		definitions present but are shadowed by the 'global' root path in the vhosts
		definition (and are broken anyway.. the cgi-bin perl script uses a wrong shebang path).
		Mistakes by developers in vhosts path declarations will expose these 'classic' services.
		These are no longer consistent with best practices and should be permanently retired.

	- /opt/web area is all 777 mode and is owned by root.

	- Apache service as managed by /etc/init.d/httpd has bugs: 'status' command fails (lynx not installed).
		Needs modernising.  Confounds attempts by package configuration tool to monitor service status.

	- Apache service start/stop is being handled by the older init.d scheme
		whereas the proxy app (node.js) has its service handled by newer systemd
		scheme.  For more consistent service administration, move to systemd.

	- The Apache install appears to be a day-1 manual install-and-build from source.
		This is not practical for a future scheme in which we need to install on unprovisioned
		servers.  Also, this leaves the apache install vulnerable to any present and future security
		alerts.   By contrast, an up-to-date and latest stable distribution package will conform to latest
		best-practices, be robust, curated, logs rotated and managed, service managed as per modern sysdtemd,
		processes correctly run as non-root, and will allow vhosts and conf files to
		be managed under the more suitable '[sites|conf]-[available|enabled]' scheme.

	Action:
	Remove the manually-managed Apache build and install the Ubuntu distro-packaged Apache release.
	Configure it for our site.
	Ensure infrastructure settings are fixed by admins, and webserver config settings can be set by web developers.

ToDo:
	- implement node.js (app-server) component on new servers

	- replace 'files' section of Ansible with a procedure to pull the site
	  definition out of a Git repository so that developers can manage
	  this section in isolation and a Continuous Deployment mechanism can be set up


