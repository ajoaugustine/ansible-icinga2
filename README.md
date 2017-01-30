[![Build Status - Master](https://travis-ci.org/juju4/ansible-icinga2.svg?branch=master)](https://travis-ci.org/juju4/ansible-icinga2)
[![Build Status - Devel](https://travis-ci.org/juju4/ansible-icinga2.svg?branch=devel)](https://travis-ci.org/juju4/ansible-icinga2/branches)
# Icinga2 ansible role

Ansible role to setup Icinga2 server with optional plugins like pnp4nagios, graphite or nagvis

See also
https://github.com/Icinga/icinga2-ansible
* missing webui for debian
* missing icinga agent/client mode

## Requirements & Dependencies

### Ansible
It was tested on the following versions:
 * 1.9

### Operating systems

Tested with vagrant only on Ubuntu 14.04 for now but should work on 12.04 and similar debian based systems.
Verified with kitchen against ubuntu14

## Example Playbook

Just include this role in your list.
For example

```
- host: myserver
  roles:
    - { role: icinga2, chart_module: pnp4nagios }
```

[Normally this part is not needed anymore]
Finish install by 
1) icingaweb2
    http://host/icingaweb2/
read token from /root/.icingaweb2_token
copy it in web interface
validate wanted modules (default monitoring, pnp4nagios or graphite, nagvis)
validate requirements
Authentication: can leave default database
 ex: create database 'icingaweb2' with user/pass of your choice
give mysql root access
create first web account
validate application configuration
 backend: icinga, ido
 dbname icinga2, icinga2 / {{ mysqlidouser }}, icinga2 / {{ mysqlidopass }}
 => if credentials is wrong, connect to mysql 
       mysql> SET PASSWORD FOR 'icinga2-ido-mysq'@'localhost' = PASSWORD('icinga2-ido-mysql');
       OR
       mysql> GRANT ALL PRIVILEGES ON icinga2idomysql.* TO 'icinga2-ido-mysq'@'localhost' IDENTIFIED BY 'icinga2';
IDO Credentials in /etc/icinga2/features-available/ido-mysql.conf

2) pnp4nagios
eventually htaccess

3) nagvis
Need to configure url path in icingaweb2: Configuration: Modules: Nagvis: Nagvis
https://IP/nagvis/
(default account admin/admin)
If you want to use automap, you will need to create such a map and ensure your hosts lists have all a parent dependencies but one. Such host will be defined as the root of the map.

As for any services, you are recommended to do hardening.

You can validate system is working correctly either in cli, either web ui.
```
$ icingacli monitoring list
```
or connect on http://IP/icingaweb2/


## Variables

```
tz: America/New_York
scriptsdir: /usr/local/scripts
backupdir: /var/_/backup/{{ inventory_hostname }}

icinga2mode: server
## server conf
icinga2_mysqlrootpw: root
icinga2_mysqlidodb: icinga2
icinga2_mysqlidouser: icinga2
icinga2_mysqlidopass: icinga2
icinga2_apache_httpsonly: true
# Ubuntu
icinga2_apacheconf: /etc/apache2/conf-enabled
# Debian
#icinga2_apacheconf: /etc/apache2/conf.d
## htpasswd for pnp4nagios? BUG/FIXME!
#icinga2_webuser: admini
#icinga2_webpass: admini

icinga2_api: false

icinga2_chart_module: pnp4nagios
#icinga2_chart_module: graphite
icinga2_graphite_backend: sqlite
#icinga2_graphite_backend: postgresql
## graphite base_url (eventually through vagrant forwarding)
icinga2_graphite_base_url: http://127.0.0.1:1588
icinga2_graphite_carbon_localhostonly: true

icinga2_nagvis_removedemo: true
icinga2_nagvis_http_timeout: 3600

## Client conf

#icinga2mode: client
icinga2_server: monserver
icinga2_server_ip: 192.168.0.100
## unixweb, unixserver, clientmac, clientwin, clientlinux
#monclientype: unixweb
icinga2_master_port: 5665
icinga2_pki_dir: /etc/icinga2/pki
icinga2_if: eth0
```

You can also used group 'monitored' inside inventory to apply basic alive monitoring.

## Troubleshooting & Known issues

* memory
Ensure you have enough memory and swap available
* htpasswd generated by ansible or command is not recognized. disabling authentication in /etc/apache2/conf-enabled/pnp4nagios.conf works ok but.
* miss an upstart config file for ubuntu, eventually with 'Restart=on-failure'
* If problem with ido as it seems to have difficulty to respect custom/password format, just reconfigure db
# dpkg-reconfigure icinga2-ido-mysql
* If you go with https only and use vagrant, you will need to change port forward setup
* if setting multiple icinga2 guests on your host, icingaweb2 will logout you from other as cookie name is conflicting. no easy way to change it unless changing name in source code.
* when icingaweb2 session expires, you are logout and redirection url argument seems to be expanded in a crazy way which will create a Forbidden error by modsecurity
* role contains a modsecurity exceptions file which allows to run with most modsecurity without altering icinga2. Still access to icinga2 should be restricted. Don't rely on this small extra for full security.
* nagvis:
"Error: (0) Undefined offset: 16 (/usr/local/nagvis-1.8.5/share/server/core/classes/GlobalBackendmklivestatus.php:997)"
* "There is currently no icinga instance writing to the IDO. Make sure that a icinga instance is configured and able to write to the IDO."
when configuring icingaweb2 the first time on "Monitoring IDO Resource" page

* If you want to include auto restart of some services, can use: monit, upstart/systemd respawn, inittab
https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples
http://www.cyberciti.biz/tips/howto-monitor-and-restart-linux-unix-service.html

* icingaweb2 seems to have issue with # comments in ini files like backends.ini and resources.ini. avoid!

* nagvis not connecting
```
# ncat -U /var/run/icinga2/cmd/livestatus 
Ncat: Connection refused.
# icinga2 feature list
Disabled features: api command compatlog debuglog gelf graphite icingastatus opentsdb statusdata syslog
Enabled features: checker ido-mysql livestatus mainlog notification perfdata
```
try to restart service

* icingaweb2: "Currently there is no dashlet available."
?

* icingaweb2: "Failed to send external Icinga command. None of the configured transports was able to transfer the command"
selinux: No (Ubuntu, only AppArmor)

* role is not idempotent, mostly nagvis part to review
* doing travis test with docker to do multiple distribution brings issues with mysql so for now, sticking to ubuntu trusty test only.

## License

BSD 2-clause


