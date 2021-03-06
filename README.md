zabbix-agent-extensions
=======================

# Overview

A set of zabbix UserParameter scripts and a monitoring template for linux systems.

 * this project ist originated on: https://github.com/scoopex/zabbix-agent-extensions
 * this project is presented at zabbix share: https://share.zabbix.com/operating-systems/zabbix-agent-extensions
 * the ci is located at: https://travis-ci.org/scoopex/zabbix-agent-extensions

# Monitoring details

This package provides the following capabilities:

 * linux standard monitoring using the standard items provided by the zabbix-agent
   * monitor memory behavior
   * monitor important servcies: smtp, ssh, cron
   * swap usage
   * 5min system load
   * monitor dmesg for bad behavior of the system
   * monitor the maximum and minimum of processes
   * automatic discovery
      * filesystems: inode and space measures<BR>
        (the amount of discovered devices can be limited by a configuration file on the monitored host)
      * network interfaces: packets and transferrates per second
      * storage devices: operations per second
   * number of processes
 * monitor ICMP ping 
 * monitor the MTA mailqueue
 * monitor NFS operations/retransmits
 * apache monitoring:
   (enable /server-status available on localhost, https://httpd.apache.org/docs/2.4/mod/mod_status.html)
   * Check Apache Server-Status (Readers, Writers - alert if to many slots are in use)
   * Loadbalancer check
   * Monitor Mod JK backend status
 * elasticsearch node and cluster monitoring (needs elasticsearch and es_stats_zabbix Python modules)
 * redis Monitoring (needs redis Python module)
 * NGINX Monitoring
   (enable /basic_status available on localhost, https://nginx.org/en/docs/http/ngx_http_stub_status_module.html)
 * generic discovery
   (put json snippets to /var/run/zabbix-generic-discovery/ITEMNAME.json i.e. with puppet and get a combined discovery value)
 * monitor puppet state
 * linux software raid state
 * check quality of ntp sync behavior
 * check for required reboot
 * Configurable autodiscovery of block devices
 * Monitor performance havior of discovered block devices
 * Configurable autodisovery of real disk devices
 * Check smart state of disks and gather statistics
 * Discover network devices and monitor performance and error behavior
 * Zabbix agent version 


Almost all measures are integrated in graphs which are displayed on the various host screens.

A quick overview is provided by the following files:

 * [custom-os-linux](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-os-linux.html)
 * [custom-os-linux-hardware](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-os-linux-hardware.html)
 * [custom-os-puppet](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-os-puppet.html)
 * [custom-service-apache](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-apache.html)
 * [custom-service-cups](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-cups.html)
 * [custom-service-elasticsearch-cluster](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-elasticsearch-cluster.html)
 * [custom-service-elasticsearch-node](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-elasticsearch-node.html)
 * [custom-service-nginx](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-nginx.html)
 * [custom-service-nginx-logstats](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-nginx-logstats.html)
 * [custom-service-redis](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-redis.html)
 * [custom-service-varnish](http://htmlpreview.github.io/?https://github.com/scoopex/zabbix-agent-extensions/blob/master/zabbix_templates/3.4/documentation/custom-service-varnish.html)

# How to test and debug

 * Install the packages on the zabbix agent host
 * Test the agent on the agent machine
 
   ```
   zabbix_agentd  -t linux.dmesg
   zabbix_agentd --print
   ```
 * Test the agent on the server machine
 
   ```
   apt-get install zabbix-get
   zabbix_get -s 127.0.0.1 -k linux.dmesg
   ```

# How to release and/or install the userparameter scripts

   Adpoted from: https://www.digitalocean.com/community/tutorials/how-to-use-fpm-to-easily-create-packages-in-multiple-formats

 * Install FPM
   ```
   sudo apt-get install ruby-dev build-essential debhelper devscripts rpm
   gem install fpm --user
   ```
 * Get the repo 
   ```
   git clone https://github.com/scoopex/zabbix-agent-extensions
   cd zabbix-agent-extensions
   ```
 * Release only: Edit this file (README.md) and describe the new feature
 * Create packages
   ```
   git commit ....
   ./create_packages
   ```
 * Install the rpm or debian archive on as an addition to your zabbix-agent:
 
   ```
   rpm -Uvh noarch/zabbix-agent-extensions-<version>.noarch.rpm
   dpkg -i zabbix-agent-extensions_<version>_all.deb
   ```
 * Push release
   ```
   git tag <release-number>
   ./create_packages
   git push
   git push --tags
   ```
 * TravisCI builds and tests the release and uploads it to github

# How to configure discovery for zabbix agent
 
 * Configure disk device discovery
    * Create file: /etc/zabbix/item_zabbix_device_discovery.json
    * Add content to include/exclude devices
      ```
{
  "regex_includes_name": [
    ".*"
  ],
  "regex_includes_model": [
    ".*"
  ],
  "regex_excludes_name": [
    "loop.*", "fd.*", "sr.*", "dm.*", "ram.*"
  ]
  "regex_excludes_model": [
  ]
}
      ```
    * Test
      ```
      zabbix_agentd -t "vfs.dev.discovery"
      ```
 * Configure generic discovery
   * Decide to use a appname consiting of the following characters: a-zA-Z0-9. (Alphanumeric and dot characters)
   * Add file snippes to /var/run/zabbix-generic-discovery/ which look like <appname>-<anything>.json
     ```
     {
        "{#FSNAME}":"\/",
        "{#FSTYPE}":"rootfs"
     },
     ```
   * Test the disovery by
     ```
      zabbix_agentd -t "generic.discovery[appname]"
     ```

# How to configure the zabbix server

The templates will work on zabbix 2.2 and above.

 * Open Zabbix web frontend
 * Open "Configuration" => "Templates"
 * Press button "Import"
 * Activate Linux template
   * Load "zabbix_templates/3.4/Custom - OS - Linux.xml"
   * Open template "Custom - OS - Linux" and modify the default values defined in the macros
     * {$DISK_USAGE_PERCENT_ALARM}: percentage of storage usage to send alarms 
     * {$MAXIMUM_NUMBER_RETRANSMISSIONS}: alert if that number of nfs retransmits appears in one monitoring cycle
     * {$MONITOR_LOAD_WARNING_MULT} : multiplying factor
     * {$MONITOR_TIMEOUT} : amount of time to complain if hosts does not provide values anymore
   * Assign template "Custom - OS - Linux" to the desired hosts and modify the default values to host specific settings
 * Activate Apache template
   * Load "zabbix_templates/Custom - Service - Apache.xml"
   * Open template "Custom - Service - Apache" and modify the default values defined in the macros
   * Assign template "Custom - Service - Apache" to the desired hosts and modify the default values to host specific settings
 * ...
   
# Licence and Authors

Additional authors are very welcome - just submit your patches as pull requests.

  * Marc Schoechlin <ms@256bit.org>
  * Marc Schoechlin <marc.schoechlin@breuninger.de> (not active anymore)
 
This software is licensed by GPLv2 - review file "LICENSE"
