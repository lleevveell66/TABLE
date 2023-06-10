## TABLE 1.0 by LEVEL6 (https://github.com/lleevveell66)

<link rel="stylesheet" type="text/css" href="css/github.css"> 

<h1 align="center">
  <img src="images/TableLogo.png" alt="TABLE Logo" width="100%" align="middle">
</h1>

A distributed, ham-fisted, attack sensoring, IPS/IDS/firewall solution.  Take your bandwidth back.<br>


## Table of contents:

* [Table of Contents](#table-of-contents) 
* [Description](#description)
	* [Why?](#why)
* [Theory of Operation](#theory-of-operation) 
	* [Attack Scenarios](#attack-scenarios)
		* [Scenario 1](#scenario-1)
		* [Scenario 2](#scenario-2)
	* [Hardware Components](#hardware-components)
	* [Software Components](#software-components)
	* [Other Ideas](#other-ideas)
* [Requirements](#requirements)
* [Build Your Own](#build-your-own)
	* [Hardware](#hardware)
	* [Software](#software)
* [LIE TABLE_ips](#lie-table_ips)
* [Real World Study](#real-world-study)
* [Caveats and Dangers](#caveats-and-dangers)



## Description:
[Table of Contents](#table-of-contents)

TABLE is a distributed IDS (Intrusion Detection System) and IPS (Intrusion Prevention System) which uses only common linux tools to detect attacks and distribute the protection from further attack.

Implementation is generally straight-forward to implement on your own network of servers, but LIE is also providing an hourly-updated ipset built from their own implementation involving 7 hosts around the world.

All previous generations of solutions which have led to this (DDoP - Distributed Denial of Packets, FIBN - Firewall ipset Blacklist Network, et al) have been retired and will be removed, soon.  I never properly kept up with their repos as they evolved, anyway.  TABLE is what they all became.

### Why?:

Network scanning and attacks have proliferated so much that the traffic generated can signifigantly affect the overall bandwidth you have available for legitimate use.


## Theory of Operation:
[Table of Contents](#table-of-contents)

We will refer to the following diagrams for this explanation:

<h4 align="center"> 
<table cellpadding="3" cellspacing="3" border="0">
<tr><td>
    <img src="images/TableDiagram_Scenario1.png" alt="TABLE Scenario 1" width="100%" align="middle"><br>
    TABLE Scenario 1
</td><td>
    <img src="images/TableDiagram_Scenario2.png" alt="TABLE Scenario 2" width="100%" align="middle"><br>
    TABLE Scenario 2
</td></tr>
</table>
</h3> 

### Attack Scenarios:

An "attack" in these scenarios is defined as being "a completed TCP socket connection to a honey pot port".

You are going to begin to sense that this solution uses a tank to kill a fly, and your sense will be correct.  You may continue to develop this further and tweak it to be less ham-fisted about things, if you like.  The TABLE solution, as it stands, would not be a desirable solution for any network which requires connectivity to many places in the world for commerce, for example.  My network does not.  I would frankly be fine with blocking the entire countries of China, Russia, Iran, and Brazil.  This is indeed what I have tried to do in the past.  It works, for a while.  But, it must be kept in sync with changing CIDR blocks in the world, and that can require subscription fees, etc.

As it stands, all atttacking IP addresses in the US are blocked on their aaa.bbb.ccc.0/24 subnet.  All attacking IP addresses outside of the US are blocked on their aaa.bbb.0.0/16 subnet.  You can certainly change this behavior to be less aggresive, but be aware of ipset's limits.  If you begin only blocking single (/32) IP addresses, you will quickly overwhelm your firewall tables.

#### Scenario 1:

Scenario 1 illustrates an attack on Server 1 by some machine in "The World".

1) The World attacks Server 1
2) Server 1 recognizes this attack
3) Server 1 reports the details about the attack to the TABLE Server  
4) TABLE Server makes some quick decisions about where in the world this attack originated, using IP geolocation
5) TABLE Server initiates an Ansible playbook to reach out to all of the participating servers
6) Each participating server inserts the appropriate subnet into the TABLE_ips ipset
7) All further attacks from this subnet to all participating hosts are stopped

#### Scenario 2:

Scenario 2 illustrates the extra step required for an atack on a remotely-hosted Server 4 by some machine in "The World".

1) The World attacks Server 4 
2) Server 4 recognizes the attack 
3) Server 4 reports the details about the attack to the Web Repeater
4) Web Repeater relays the attack information back into the TABLE Server 
5) TABLE Server makes some quick decisions about where in the world this attack originated, using IP geolocation
6) TABLE Server initiates an Ansible playbook to reach out to all of the participating servers
7) Each participating server inserts the appropriate subnet into the TABLE_ips ipset
8) All further attacks from this subnet to all participating hosts are stopped

### Hardware Components:

TABLE is distributed among as many hosts as possible.  The more hosts that participate, the quicker TABLE will work to minimize sources of attack.

The diagrams show servers in a personal network (IDN + DMZ) as well as virtualized instances in two different clouds (maybe Linode and AWS).  Hosts in your local network are slightly easier to set up, because they will most likely have direct inward connectivity from the DMZ (De-Militarized Zone) to the TABLE Server in the IDN (Internal Data Network).  Servers hosted outside of the network where the TABLE Server lives will require some trickery to reach the TABLE Server.  A "table_relay.cgi" script has been written in Perl and supplied, but feel free to devise your own method.  Port forwarding is another possibility, but not recommended.  You could also chance hosting the TABLE Server on a Web Server in your DMZ, eliminating the need for connectivity to the IDN.

This is the only hardware really required for this to work - *nix servers.  If you have separate hardware firewalls, you will need to develop for that.  


### Software Components:

The following have been developed to help you to get this set-up on your own network:

* table_reporter - This is a bash script that is placed on every participating server to report attacks
* table_block - This is a bash script that is placed on every participating server to finally block the attacker
* table_activity.cgi - This is a Perl CGI script hosted on the TABLE Server which listens for table_reporter reports, makes decisions about the attacking subnet, then runs the table_blocknet script
* table_relay.cgi - This is a Perl CGI script hosted on the Web Repeater which listens for table_reporter reports, then relays those inwardly to the table_activity.cgi script on the TABLE Server
* table_blocknet - This is a bash script that is executed on the TABLE Server to kick off the BlockSubnet.yaml Ansible playbook
* BlockSubnet.yaml - This is an Ansible playbook which will connect to all participating machines and execute their local table_block scripts to finally block the attacking subnet.  This is done via SSH keys in my scenario, but feel free to change that.
* portsentry-leper-1.2.317.005.tgz - A customized release of the old portsentry tool, from before Psionic sold out to Cisco.  This is what is being used to detect the attacks on every server and execute the table_reporter scripts.  There are many other ways, but this is supplied for you.

You will understand, of course, if I need to cleanse all of this a bit to safely share all of this publically.  It will likely require effort on your part to dirty it back up for your own environment.  But, the general procedure will be provided, here.


### Other Ideas:

List some other expansion or customization ideas, here.

* Build a dedicated TABLE network fully in a cloud, dropping off the ipset for retreival and use elsewhere
* More strategic virtual host placement to study in more detail the most likely sources of attack in other locations in the world
* Easy integration with fail2ban for another layer of valid attack detection on legitimate, non-honeypot ports
* World heat maps of attacks through the ages (nothing says you have to actually block them)

## Requirements:
[Table of Contents](#table-of-contents)

You will need the following installed to duplicate this project, although much substitution is possible given a little thought:

* A few *nix servers
* iptables
* httpd
* sudo/sudoers
* Perl
* Ansible
* .. more to come ... 



## Build Your Own:
[Table of Contents](#table-of-contents)

We will walk you through an implementation of TABLE.  All of the scripts are very simple.  Hate Perl?  Port it to Python and bow to your Indentation Overlords.   Big firewalld fan boi?  Easy peasy to customize for your needs.

I only run RHEL variants in my environment, so you are going to get "the RedHat way".  If you are not capable of translating things into apt, pkg, etc., then you maybe should not be trying to implement this solution quite yet.

### Hardware:
[Table of Contents](#table-of-contents)

Again, the more *nix servers participating in this distributed solution, the better.  Basic segregation of the network is always recommended, but not essential for this to function.  We will presume you have stood up at least four (4) servers for this example:

* TABLE Server
* Server 1 on your DMZ
* Server 2 in someone else's cloud
* Web Repeater

### Software:
[Table of Contents](#table-of-contents)

The following assumptions are made for this example:

* You have a web server installed and functioning on TABLES Server and Web Repeater.  Apache is used in the example.
* You have replaced firewalld with iptables, everywhere.  (See below for how to do that, if you want to)
* You have functioning sshd configured everywhere.  We will show how to go the password-less key'ed way.
* You have a working sudo implementation.  Apache user will need to run the script as a different user.

#### Replacing Firewalld with Iptables

With easy customization, TABLE can work with any firewall management system.  We will be using iptables for this example, however.   If you would like to drop that firewalld and get back to iptables, here is the best way in RHEL:

```
  yum -y install iptables-services
  systemctl stop firewalld
  systemctl start iptables
  # systemctl start ip6tables     # If you have implemented IPv6 - I have not
  systemctl disable firewalld
  systemctl mask firewalld
  systemctl enable iptables
  # systemctl enable ip6tables    # If you have implemented IPv6 - I have not
```

#### TABLE Server:

##### Apache:

Enable and start the Apache web server:
```
  systemctl enable httpd --now
```

Configure to allow .cgi scripts to be executed in any directory (you can fine-tune this via htaccess, if you know how):
```
  vi /etc/httpd/conf/httpd.conf
  .
  .
  # Options Indexes FollowSymLinks
  Options Indexes FollowSymLinks ExecCGI
  .
  .
  # AddHandler cgi-script .cgi
  AddHandler cgi-script .cgi
  .
  .
```

Restart Apache:
```
  systemctl restart httpd
```

Prepare some log files for writing by apache user:
```
  touch /var/log/table.log /var/log/blocking.log
  chmod 644 /var/log/table.log /var/log/blocking.log
```

Put the table_activity.cgi script into place:
```
  mkdir -p /var/www/html/TABLE/
  # Copy the table_activity.cgi from this repo into /var/www/html/TABLE/
  chmod 755 /var/www/html/TABLE/ /var/www/html/TABLE/table_activity.cgi
  chown apache:apache /var/log/table.log /var/log/blocking.log /var/www/html/TABLE/ /var/www/html/TABLE/table_activity.cgi
```

There is a good chance that the free IP Geolocation API that I found is going to stop being free, after releasing this.  Be prepared to find another and edit this script.

##### Ansible:

Install Ansible with:
```
  yum -y install ansible
```

Create a new user for running the Ansible script and SSH key'ing and switch user to it:
```
  useradd -c "TABLE User" table
  passwd table
  su - table
```

Prepare an Ansible area:
```
  mkdir ansible
  cd ansible
  mkdir hosts

  vi hosts/TABLEhosts
  .
  .
  # insert the IP addresses of all participating hosts, here
  .
  .
```

Either transfer the BlockSubnet.yanl file from this repo to ~table/ansible , or build it now:
```
  cat<<EOF>BlockSubnet.yaml
  ---
  - name: Block IP Subnet
    hosts: TABLEhosts
    vars:
      ansible_ssh_user: 'root'
      ansible_port: '22'
      my_command: "{{ command }}"
      my_subnet: "{{ subnet }}"
    tasks:
    - name: Run the remote table_block script with command line options
      command: table_block {{ command }} {{ subnet }}
      register: output
    - debug: msg="{{ output.stdout_lines }}"
    - debug: msg="{{ output.stderr_lines }}"
  EOF
```


##### SSH:

Switch user to the table user, if you are not already in a table user shell:
```
  su - table
```

Much can be tweaked here, but let us keep this very simple.  Generate a password-less keypair:
```
  ssh-keygen  
  # Just hit <RETURN> for no password, for this example
```

Propagate your public keys to root user of every participating server:
```
  ssh-copy-id -p 22 root@Server1
  ssh-copy-id -p 22 root@Server2
  ssh-copy-id -p 22 root@WebRepeater
```


##### SUDO:



##### Perl:

To use the supplied script, you will need the CGI Perl module:
```
  yum -y install perl-CGI
```


#### Server 1:



#### Server 2:

#### Web Repeater:

```
    This is a code block
```

## LIE TABLE_ips
[Table of Contents](#table-of-contents)

Just want the TABLE_ips ipset?  We got you.  We realize that not everyone will wish to implement such a distributed solution, especially since it requires much customization for one's own environment.  So, we have made the TABLE_ips ipset generated hourly at LIE available to everyone.

https://www.leper.org/TABLE/

Directions for how to utilize this ipset are included on that page.

If you do not use/like ipset or iptables, this file is easily massaged to utilize in your favorite firewall solution.



## Real World Study
[Table of Contents](#table-of-contents)

<h1 align="center">
  <img src="images/LAVA_Page_Reset_10.png" alt="TABLE Study 10" width="100%" align="middle">
</h1>

A real-world study of the efficiency of the TABLE solution is included, here.

Write up my testing, here.   Include all the screenshots of LAVA charts.

Until I get this filled in, just know that during the first 7 days of running TABLE, a 6-hour moving average of attacks dropped from around 20 to 5 attacks per hour.  This was using TABLE on top of previous implemented solutions.  Then, all firewall tables were cleared.  Attacks jumped immediately to over 60 per hour.  Within 3 days, this has dropped by 66% back down near 20 attacks per hour.

All other connectivity on my network is fine. 


## Caveats and Dangers
[Table of Contents](#table-of-contents)

Of course, there is always a bit of risk when taking such drastic pro-active measures.  Here are some points to keep in mind:

* You could just use iptables (or other tools) to watch for all port scans, but beware that SYN packets are much more easily spoofed than completed TCP socket connections.  Still... these connections can also be spoofed.  MAKE A GOOD WHITELIST!  Include uplinks through your provider(s) in it.  
* It is highly recommended that you never take counter-offensive measures.  Do not taunt, do not back-hack nor scan.   It should be enough to know that some automaed bot in some subnet found you.   If you begin to provoke them, you will eventually experience much more trouble than this solution can protect you from.  
* The determination of subnet is "precise enough".  But, beware that sometimes these subnets will contain more than one country. The TABLE scripts do not check the entire subnet.  Contrarily, only one IP address in that subnet is checked, and then the assumption is made that the rest of that subnet also originates in that country.  This is a bad assumption and why this is being called "ham-fisted".  But, it does work well when you care less about sharing services to the world.
* This solution may not compete with a true hardware IPS in speed.  The timing from detection to prevention varies widely based on system loads and network timing.  It has usually come in under a minute from attack to blocking for our implementation, though.

EOT
