---
layout: post
title:  "Installing Central Authentication(413-11)"
categories: Linux
tags: 413 
---

### Installing an Identify Management(IdM) Server

Red Hat Identify Management(IdM) is a solution to centrally manage authentication and authorization policies from a Linux server for enrolled Linux clients using native Linux tools.

*    server package: ipa-server
*    client package: ipa-client ipa-admintools

###### Demonstration: Installing an Identify Management(IdM) Server

1. DNS NTP IPTABLES
2. RHEL6.3(idM 3.0)
3. NetworkManager off
4. ifcfg-eth0 static no ip netmask dns
5. install ipa-server

```
# ipa --help
Usage: ipa [options]

Options:
  -h, --help         show this help message and exit
  -e KEY=VAL         Set environment variable KEY to VAL
  -c FILE            Load configuration from FILE
  -d, --debug        Produce full debuging output
  --delegate         Delegate the TGT to the IPA server
  -v, --verbose      Produce more verbose output. A second -v displays the
                     XML-RPC request
  -a, --prompt-all   Prompt for ALL values (even if optional)
  -n, --no-prompt    Prompt for NO values (even if required)
  -f, --no-fallback  Only use the server configured in /etc/ipa/default.conf

  Available help topics:
    ipa help topics

  Available commands:
    ipa help commands


# ipa help commands
automember-add                   Add an automember rule.
automember-add-condition         Add conditions to an automember rule.
automember-default-group-remove  Remove default (fallback) group for all unmatched entries.
automember-default-group-set     Set default (fallback) group for all unmatched entries.
automember-default-group-show    Display information about the default (fallback) automember groups.
automember-del                   Delete an automember rule.
automember-find                  Search for automember rules.
automember-mod                   Modify an automember rule.
automember-remove-condition      Remove conditions from an automember rule.
automember-show                  Display information about an automember rule.
automountkey-add                 Create a new automount key.
automountkey-del                 Delete an automount key.
automountkey-find                Search for an automount key.
automountkey-mod                 Modify an automount key.
automountkey-show                Display an automount key.
automountlocation-add            Create a new automount location.
automountlocation-del            Delete an automount location.
automountlocation-find           Search for an automount location.
automountlocation-import         Import automount files for a specific location.
automountlocation-show           Display an automount location.
automountlocation-tofiles        Generate automount files for a specific location.
automountmap-add                 Create a new automount map.
automountmap-add-indirect        Create a new indirect mount point.
automountmap-del                 Delete an automount map.
automountmap-find                Search for an automount map.
automountmap-mod                 Modify an automount map.
automountmap-show                Display an automount map.
cert-remove-hold                 Take a revoked certificate off hold.
cert-request                     Submit a certificate signing request.
cert-revoke                      Revoke a certificate.
cert-show                        Retrieve an existing certificate.
cert-status                      Check the status of a certificate signing request.
config-mod                       Modify configuration options.
config-show                      Show the current configuration.
console                          Start the IPA interactive Python console.
delegation-add                   Add a new delegation.
delegation-del                   Delete a delegation.
delegation-find                  Search for delegations.
delegation-mod                   Modify a delegation.
delegation-show                  Display information about a delegation.
dns-resolve                      Resolve a host name in DNS.
dnsconfig-mod                    Modify global DNS configuration.
dnsconfig-show                   Show the current global DNS configuration.
dnsrecord-add                    Add new DNS resource record.
dnsrecord-del                    Delete DNS resource record.
dnsrecord-find                   Search for DNS resources.
dnsrecord-mod                    Modify a DNS resource record.
dnsrecord-show                   Display DNS resource.
dnszone-add                      Create new DNS zone (SOA record).
dnszone-add-permission           Add a permission for per-zone access delegation.
dnszone-del                      Delete DNS zone (SOA record).
dnszone-disable                  Disable DNS Zone.
dnszone-enable                   Enable DNS Zone.
dnszone-find                     Search for DNS zones (SOA records).
dnszone-mod                      Modify DNS zone (SOA record).
dnszone-remove-permission        Remove a permission for per-zone access delegation.
dnszone-show                     Display information about a DNS zone (SOA record).
env                              Show environment variables.
group-add                        Create a new group.
group-add-member                 Add members to a group.
group-del                        Delete group.
group-detach                     Detach a managed group from a user.
group-find                       Search for groups.
group-mod                        Modify a group.
group-remove-member              Remove members from a group.
group-show                       Display information about a named group.
hbacrule-add                     Create a new HBAC rule.
hbacrule-add-host                Add target hosts and hostgroups to an HBAC rule.
hbacrule-add-service             Add services to an HBAC rule.
hbacrule-add-user                Add users and groups to an HBAC rule.
hbacrule-del                     Delete an HBAC rule.
hbacrule-disable                 Disable an HBAC rule.
hbacrule-enable                  Enable an HBAC rule.
hbacrule-find                    Search for HBAC rules.
hbacrule-mod                     Modify an HBAC rule.
hbacrule-remove-host             Remove target hosts and hostgroups from an HBAC rule.
hbacrule-remove-service          Remove service and service groups from an HBAC rule.
hbacrule-remove-user             Remove users and groups from an HBAC rule.
hbacrule-show                    Display the properties of an HBAC rule.
hbacsvc-add                      Add a new HBAC service.
hbacsvc-del                      Delete an existing HBAC service.
hbacsvc-find                     Search for HBAC services.
hbacsvc-mod                      Modify an HBAC service.
hbacsvc-show                     Display information about an HBAC service.
hbacsvcgroup-add                 Add a new HBAC service group.
hbacsvcgroup-add-member          Add members to an HBAC service group.
hbacsvcgroup-del                 Delete an HBAC service group.
hbacsvcgroup-find                Search for an HBAC service group.
hbacsvcgroup-mod                 Modify an HBAC service group.
hbacsvcgroup-remove-member       Remove members from an HBAC service group.
hbacsvcgroup-show                Display information about an HBAC service group.
hbactest                         Simulate use of Host-based access controls
help                             Display help for a command or topic.
host-add                         Add a new host.
host-add-managedby               Add hosts that can manage this host.
host-del                         Delete a host.
host-disable                     Disable the Kerberos key, SSL certificate and all services of a host.
host-find                        Search for hosts.
host-mod                         Modify information about a host.
host-remove-managedby            Remove hosts that can manage this host.
host-show                        Display information about a host.
hostgroup-add                    Add a new hostgroup.
hostgroup-add-member             Add members to a hostgroup.
hostgroup-del                    Delete a hostgroup.
hostgroup-find                   Search for hostgroups.
hostgroup-mod                    Modify a hostgroup.
hostgroup-remove-member          Remove members from a hostgroup.
hostgroup-show                   Display information about a hostgroup.
idrange-add                      Add new ID range.
idrange-del                      Delete an ID range.
idrange-find                     Search for ranges.
idrange-mod                      Modify ID range.
idrange-show                     Display information about a range.
krbtpolicy-mod                   Modify Kerberos ticket policy.
krbtpolicy-reset                 Reset Kerberos ticket policy to the default values.
krbtpolicy-show                  Display the current Kerberos ticket policy.
migrate-ds                       Migrate users and groups from DS to IPA.
netgroup-add                     Add a new netgroup.
netgroup-add-member              Add members to a netgroup.
netgroup-del                     Delete a netgroup.
netgroup-find                    Search for a netgroup.
netgroup-mod                     Modify a netgroup.
netgroup-remove-member           Remove members from a netgroup.
netgroup-show                    Display information about a netgroup.
passwd                           Set a user's password.
permission-add                   Add a new permission.
permission-del                   Delete a permission.
permission-find                  Search for permissions.
permission-mod                   Modify a permission.
permission-show                  Display information about a permission.
ping                             Ping a remote server.
plugins                          Show all loaded plugins.
privilege-add                    Add a new privilege.
privilege-add-permission         Add permissions to a privilege.
privilege-del                    Delete a privilege.
privilege-find                   Search for privileges.
privilege-mod                    Modify a privilege.
privilege-remove-permission      Remove permissions from a privilege.
privilege-show                   Display information about a privilege.
pwpolicy-add                     Add a new group password policy.
pwpolicy-del                     Delete a group password policy.
pwpolicy-find                    Search for group password policies.
pwpolicy-mod                     Modify a group password policy.
pwpolicy-show                    Display information about password policy.
role-add                         Add a new role.
role-add-member                  Add members to a role.
role-add-privilege               Add privileges to a role.
role-del                         Delete a role.
role-find                        Search for roles.
role-mod                         Modify a role.
role-remove-member               Remove members from a role.
role-remove-privilege            Remove privileges from a role.
role-show                        Display information about a role.
selfservice-add                  Add a new self-service permission.
selfservice-del                  Delete a self-service permission.
selfservice-find                 Search for a self-service permission.
selfservice-mod                  Modify a self-service permission.
selfservice-show                 Display information about a self-service permission.
selinuxusermap-add               Create a new SELinux User Map.
selinuxusermap-add-host          Add target hosts and hostgroups to an SELinux User Map rule.
selinuxusermap-add-user          Add users and groups to an SELinux User Map rule.
selinuxusermap-del               Delete a SELinux User Map.
selinuxusermap-disable           Disable an SELinux User Map rule.
selinuxusermap-enable            Enable an SELinux User Map rule.
selinuxusermap-find              Search for SELinux User Maps.
selinuxusermap-mod               Modify a SELinux User Map.
selinuxusermap-remove-host       Remove target hosts and hostgroups from an SELinux User Map rule.
selinuxusermap-remove-user       Remove users and groups from an SELinux User Map rule.
selinuxusermap-show              Display the properties of a SELinux User Map rule.
service-add                      Add a new IPA new service.
service-add-host                 Add hosts that can manage this service.
service-del                      Delete an IPA service.
service-disable                  Disable the Kerberos key and SSL certificate of a service.
service-find                     Search for IPA services.
service-mod                      Modify an existing IPA service.
service-remove-host              Remove hosts that can manage this service.
service-show                     Display information about an IPA service.
show-mappings                    Show mapping of LDAP attributes to command-line option.
sudocmd-add                      Create new Sudo Command.
sudocmd-del                      Delete Sudo Command.
sudocmd-find                     Search for Sudo Commands.
sudocmd-mod                      Modify Sudo Command.
sudocmd-show                     Display Sudo Command.
sudocmdgroup-add                 Create new Sudo Command Group.
sudocmdgroup-add-member          Add members to Sudo Command Group.
sudocmdgroup-del                 Delete Sudo Command Group.
sudocmdgroup-find                Search for Sudo Command Groups.
sudocmdgroup-mod                 Modify Sudo Command Group.
sudocmdgroup-remove-member       Remove members from Sudo Command Group.
sudocmdgroup-show                Display Sudo Command Group.
sudorule-add                     Create new Sudo Rule.
sudorule-add-allow-command       Add commands and sudo command groups affected by Sudo Rule.
sudorule-add-deny-command        Add commands and sudo command groups affected by Sudo Rule.
sudorule-add-host                Add hosts and hostgroups affected by Sudo Rule.
sudorule-add-option              Add an option to the Sudo Rule.
sudorule-add-runasgroup          Add group for Sudo to execute as.
sudorule-add-runasuser           Add users and groups for Sudo to execute as.
sudorule-add-user                Add users and groups affected by Sudo Rule.
sudorule-del                     Delete Sudo Rule.
sudorule-disable                 Disable a Sudo Rule.
sudorule-enable                  Enable a Sudo Rule.
sudorule-find                    Search for Sudo Rule.
sudorule-mod                     Modify Sudo Rule.
sudorule-remove-allow-command    Remove commands and sudo command groups affected by Sudo Rule.
sudorule-remove-deny-command     Remove commands and sudo command groups affected by Sudo Rule.
sudorule-remove-host             Remove hosts and hostgroups affected by Sudo Rule.
sudorule-remove-option           Remove an option from Sudo Rule.
sudorule-remove-runasgroup       Remove group for Sudo to execute as.
sudorule-remove-runasuser        Remove users and groups for Sudo to execute as.
sudorule-remove-user             Remove users and groups affected by Sudo Rule.
sudorule-show                    Display Sudo Rule.
trust-add                        Add new trust to use.
trust-del                        Delete a trust.
trust-find                       Search for trusts.
trust-mod                        Modify a trust (for future use).
trust-show                       Display information about a trust.
user-add                         Add a new user.
user-del                         Delete a user.
user-disable                     Disable a user account.
user-enable                      Enable a user account.
user-find                        Search for users.
user-mod                         Modify a user.
user-show                        Display information about a user.
user-status                      Lockout status of a user account
user-unlock                      Unlock a user account


```

###### Demonstration: Set Up an IdM Server

1. ipa-server-install 
2. service ssh restart
3. kinit admin(87654321)
4. klist
5. ipa user-find
6. ipa user-mod --password
7. https://server.example.com
9. ssh tmunsom@server.example.com

```
# netstat -antlup | grep 123
udp        0      0 10.16.98.103:123            0.0.0.0:*                               10097/ntpd 
udp        0      0 127.0.0.1:123               0.0.0.0:*                               10097/ntpd 
udp        0      0 0.0.0.0:123                 0.0.0.0:*                               10097/ntpd 
udp        0      0 2620:52:0:1060:5054:ff:f:123 :::*                                    10097/ntpd 
udp        0      0 fec0:0:a10:6000:5054:ff::123 :::*                                    10097/ntpd 
udp        0      0 fe80::5054:ff:fe40:5804:123 :::*                                    10097/ntpd
udp        0      0 ::1:123                     :::*                                    10097/ntpd
udp        0      0 :::123                      :::*                                    10097/ntpd

# ipa-server-install --help
Usage: ipa-server-install [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit

  basic options:
    -r REALM_NAME, --realm=REALM_NAME
                        realm name
    -n DOMAIN_NAME, --domain=DOMAIN_NAME
                        domain name
    -p DM_PASSWORD, --ds-password=DM_PASSWORD
                        admin password
    -P MASTER_PASSWORD, --master-password=MASTER_PASSWORD
                        kerberos master password (normally autogenerated)
    -a ADMIN_PASSWORD, --admin-password=ADMIN_PASSWORD
                        admin user kerberos password
    --hostname=HOST_NAME
                        fully qualified name of server
    --ip-address=IP_ADDRESS
                        Master Server IP Address
    -N, --no-ntp        do not configure ntp
    --idstart=IDSTART   The starting value for the IDs range (default random)
    --idmax=IDMAX       The max value value for the IDs range (default:
                        idstart+199999)
    --no_hbac_allow     Don't install allow_all HBAC rule
    --no-ui-redirect    Do not automatically redirect to the Web UI
    --ssh-trust-dns     configure OpenSSH client to trust DNS SSHFP records
    --no-ssh            do not configure OpenSSH client
    --no-sshd           do not configure OpenSSH server
    -d, --debug         print debugging information
    -U, --unattended    unattended (un)installation never prompts the user

  certificate system options:
    --external-ca       Generate a CSR to be signed by an external CA
    --external_cert_file=EXTERNAL_CERT_FILE
                        File containing PKCS#10 certificate
    --external_ca_file=EXTERNAL_CA_FILE
                        File containing PKCS#10 of the external CA chain
    --dirsrv_pkcs12=DIRSRV_PKCS12
                        PKCS#12 file containing the Directory Server SSL
                        certificate
    --http_pkcs12=HTTP_PKCS12
                        PKCS#12 file containing the Apache Server SSL
                        certificate
    --dirsrv_pin=DIRSRV_PIN
                        The password of the Directory Server PKCS#12 file
    --http_pin=HTTP_PIN
                        The password of the Apache Server PKCS#12 file
    --subject=SUBJECT   The certificate subject base (default O=<realm-name>)
    --selfsign          Configure a self-signed CA instance rather than a
                        dogtag CA. WARNING: Certificate management
                        capabilities will be limited

  DNS options:
    --setup-dns         configure bind with our zone
    --forwarder=FORWARDERS
                        Add a DNS forwarder
    --no-forwarders     Do not add any DNS forwarders, use root servers
                        instead
    --reverse-zone=REVERSE_ZONE
                        The reverse DNS zone to use
    --no-reverse        Do not create reverse DNS zone
    --zonemgr=ZONEMGR   DNS zone manager e-mail address. Defaults to
                        hostmaster@DOMAIN
    --no-persistent-search
                        Do not enable persistent search feature in the name
                        server
    --zone-refresh=ZONE_REFRESH
                        When set to non-zero the name server will use DNS zone
                        detection based on polling instead of a persistent
                        search
    --no-host-dns       Do not use DNS for hostname lookup during installation
    --no-dns-sshfp      Do not automatically create DNS SSHFP records
    --no-serial-autoincrement
                        Do not enable SOA serial autoincrement

  uninstall options:
    --uninstall         uninstall an existing installation. The uninstall can
                        be run with --unattended option

# ipa-server-install
...
The IPA Master Server will be configured with:
Hostname:      cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com
IP address:    10.16.98.103
Domain name:   idmqe.lab.eng.bos.redhat.com
Realm name:    IDMQE.LAB.ENG.BOS.REDHAT.COM

Continue to configure the system with these values? [no]: yes

The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Configuring NTP daemon (ntpd)
  [1/4]: stopping ntpd
  [2/4]: writing configuration
  [3/4]: configuring ntpd to start on boot
  [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server for the CA (pkids): Estimated time 30 seconds
  [1/3]: creating directory server user
  [2/3]: creating directory server instance
  [3/3]: restarting directory server
Done configuring directory server for the CA (pkids).
Configuring certificate server (pki-cad): Estimated time 3 minutes 30 seconds
  [1/21]: creating certificate server user
  [2/21]: creating pki-ca instance
  [3/21]: configuring certificate server instance
  [4/21]: disabling nonces
  [5/21]: creating CA agent PKCS#12 file in /root
  [6/21]: creating RA agent certificate database
  [7/21]: importing CA chain to RA certificate database
  [8/21]: fixing RA database permissions
  [9/21]: setting up signing cert profile
  [10/21]: set up CRL publishing
  [11/21]: set certificate subject base
  [12/21]: enabling Subject Key Identifier
  [13/21]: setting audit signing renewal to 2 years
  [14/21]: configuring certificate server to start on boot
  [15/21]: restarting certificate server
  [16/21]: requesting RA certificate from CA
  [17/21]: issuing RA agent certificate
  [18/21]: adding RA agent as a trusted user
  [19/21]: configure certificate renewals
  [20/21]: configure Server-Cert certificate renewal
  [21/21]: Configure HTTP to proxy connections
Done configuring certificate server (pki-cad).
Configuring directory server (dirsrv): Estimated time 1 minute
  [1/38]: creating directory server user
  [2/38]: creating directory server instance
  [3/38]: adding default schema
  [4/38]: enabling memberof plugin
  [5/38]: enabling winsync plugin
  [6/38]: configuring replication version plugin
  [7/38]: enabling IPA enrollment plugin
  [8/38]: enabling ldapi
  [9/38]: disabling betxn plugins
  [10/38]: configuring uniqueness plugin
  [11/38]: configuring uuid plugin
  [12/38]: configuring modrdn plugin
  [13/38]: enabling entryUSN plugin
  [14/38]: configuring lockout plugin
  [15/38]: creating indices
  [16/38]: enabling referential integrity plugin
  [17/38]: configuring ssl for ds instance
  [18/38]: configuring certmap.conf
  [19/38]: configure autobind for root
  [20/38]: configure new location for managed entries
  [21/38]: restarting directory server
  [22/38]: adding default layout
  [23/38]: adding delegation layout
  [24/38]: adding replication acis
  [25/38]: creating container for managed entries
  [26/38]: configuring user private groups
  [27/38]: configuring netgroups from hostgroups
  [28/38]: creating default Sudo bind user
  [29/38]: creating default Auto Member layout
  [30/38]: adding range check plugin
  [31/38]: creating default HBAC rule allow_all
  [32/38]: Upload CA cert to the directory
  [33/38]: initializing group membership
  [34/38]: adding master entry
  [35/38]: configuring Posix uid/gid generation
  [36/38]: enabling compatibility plugin
  [37/38]: tuning directory server
  [38/38]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc): Estimated time 30 seconds
  [1/10]: adding sasl mappings to the directory
  [2/10]: adding kerberos container to the directory
  [3/10]: configuring KDC
  [4/10]: initialize kerberos container
  [5/10]: adding default ACIs
  [6/10]: creating a keytab for the directory
  [7/10]: creating a keytab for the machine
  [8/10]: adding the password extension to the directory
  [9/10]: starting the KDC
  [10/10]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin 
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa_memcached
  [1/2]: starting ipa_memcached 
  [2/2]: configuring ipa_memcached to start on boot
Done configuring ipa_memcached.
Configuring the web interface (httpd): Estimated time 1 minute
  [1/14]: setting mod_nss port to 443
  [2/14]: setting mod_nss protocol list to TLSv1.0 - TLSv1.2
  [3/14]: setting mod_nss password file
  [4/14]: enabling mod_nss renegotiate
  [5/14]: adding URL rewriting rules
  [6/14]: configuring httpd
  [7/14]: setting up ssl
  [8/14]: setting up browser autoconfig
  [9/14]: publish CA cert
  [10/14]: creating a keytab for httpd
  [11/14]: clean up any existing httpd ccache
  [12/14]: configuring SELinux for httpd
  [13/14]: restarting httpd
  [14/14]: configuring httpd to start on boot
Done configuring the web interface (httpd).
Applying LDAP updates
Restarting the directory server
Restarting the KDC

Sample zone file for bind has been created in /tmp/sample.zone.2KnoNH.db
Restarting the web server
==============================================================================
Setup complete

Next steps:
	1. You must make sure these network ports are open:
		TCP Ports:
		  * 80, 443: HTTP/HTTPS
		  * 389, 636: LDAP/LDAPS
		  * 88, 464: kerberos
		UDP Ports:
		  * 88, 464: kerberos
		  * 123: ntp

	2. You can now obtain a kerberos ticket using the command: 'kinit admin'
	   This ticket will allow you to use the IPA tools (e.g., ipa user-add)
	   and the web user interface.

Be sure to back up the CA certificate stored in /root/cacert.p12
This file is required to create replicas. The password for this
file is the Directory Manager password

# ipa-server-install --hostname server.example.com -n example.com -r EXAMPLE.COM -p 12345678 -a 87654321 --no-ntp -U --idstart=5000 --idmax=10000
```

###### Performance Checklist: Install and Set Up an IdM Server

1. Perform prerequisite steps by reviewing the hardware and software requirements, opening necessary ports in the firewall, and confirming network and system configuration.

The IdM Server is supported on both i386 and x86_64 platforms of Red Hat Enterprise Linux6. Properly sizing RAM is imperative given the caching philosophy of IdM, i.e. at least 2GB
RAM for 10,000 users and at least 16GB of RAM for 100,000 users and 50,000 groups. The installation script makes certain assumptions about the system: DNS and /etc/hosts properly resolves the fully-qualified domain name, there is no preexisting 389 Directory Server or Kerberos configuration, and the nscd service is disabled.

System ports for IdM services include HTTP/HTTPS (80/tcp and 443/tcp), LDAP/LDAPS(389/tcp and 636/tcp), Kerberos (88 and 464, both tcp and udp), DNS (53, both tcp and udp), NTP (123/udp), and Certificate System (7389/tcp).

```
# uname -a
Linux cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com 2.6.32-642.el6.x86_64 #1 SMP Wed Apr 13 00:51:26 EDT 2016 x86_64 x86_64 x86_64 GNU/Linux
# head /proc/meminfo
MemTotal:        1922088 kB
MemFree:          280044 kB
Buffers:           72356 kB
Cached:          1271448 kB
SwapCached:            0 kB
Active:           720916 kB
Inactive:         708064 kB
Active(anon):       1812 kB
Inactive(anon):    83572 kB
Active(file):     719104 kB
# yum list nscd
Loaded plugins: product-id, search-disabled-repos, security, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Available Packages
nscd.x86_64                                                                         2.12-1.192.el6
# iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
# chkconfig NetworkManager off
# service NetworkManager stop
# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
BOOTPROTO="static"
HWADDR="52:54:00:00:00:FA"
IPV6INIT="yes"
MTU="1500"
NM_CONTROLLED="no"
ONBOOT="yes"
TYPE="Ethernet"
UUID="383d9eaa-ac8a-43ff-96d9-fe76e9200877"
IPADDR=192.168.0.250
NETMASK=255.255.255.0
GATEWAY=192.168.0.254
DNS1=192.168.0.254

# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=demo.example.com

# vim /etc/hosts
127.0.0.1 localhost.localdomain localhost
::1 localhost6.localdomain6 localhost6
192.168.0.250 demo.example.com demo

# chkconfig network on; service network restart
```

2. Install IdM server packages. Installing ipa-server will typically install a large number of dependencies. If the IdM server will also manage a DNS server, then you must specify
installing two additional packages: bind and bind-dyndb-ldap.

```
# yum install -y ipa-server
```

3. Create IdM server instance interactively. The ipa-server-install script will configure all of the required services for the IdM domain: Network time daemon (ntpd), 389 directory server (dirsrv), Kerberos key distribution center (KDC), Apache (httpd), Certificate authority, and an updated SELinux targeted policy. Optionally, it can integrate a DNS service. You can monitor the installation from the /var/log/ipaserver-install.log log file. Each of the component services (Apache, 389 Directory Server, Kerberos, and Certificate System) all have their own log files generally found under /var/log/ (httpd/, dirsrv/, k*.log, and pki-ca/).

```
# ipa-server-install

The same result could be accomplished by passing the following arguments to the ipa-server-install command.
# ipa-server-install --hostname server.example.com -n example.com -r EXAMPLE.COM -p 12345678 -a 87654321 --no-ntp -U --idstart=5000 --idmax=10000
```

4. Restart ssh service to obtain Kerberos credentials.

```
# service sshd restart
```

5. Verify the IdM instance by verifying Kerberos authentication and ipa command line access. 

```
# kinit admin                   -> Verifying Kerberos authentication
Password for admin@EXAMPLE.COM: 87654321

# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: admin@IDMQE.LAB.ENG.BOS.REDHAT.COM

Valid starting     Expires            Service principal
01/25/17 23:59:15  01/26/17 23:59:09  krbtgt/IDMQE.LAB.ENG.BOS.REDHAT.COM@IDMQE.LAB.ENG.BOS.REDHAT.COM
01/25/17 23:59:28  01/26/17 23:59:09  HTTP/cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com@IDMQE.LAB.ENG.BOS.REDHAT.COM

# ipa user-find admin           -> Verifying IPA access
--------------
1 user matched
--------------
  User login: admin
  Last name: Administrator
  Home directory: /home/admin
  Login shell: /bin/bash
  UID: 934200000
  GID: 934200000
  Account disabled: False
  Password: True
  Kerberos keys available: True
----------------------------
Number of entries returned 1
----------------------------
```


### Adding User and Group Entries to Identify Management(IdM)

```
# ipa user-add
First name: Thurman
Last name: Munsom
User login [tmunsom]: 
--------------------
Added user "tmunsom"
--------------------
  User login: tmunsom
  First name: Thurman
  Last name: Munsom
  Full name: Thurman Munsom
  Display name: Thurman Munsom
  Initials: TM
  Home directory: /home/tmunsom
  GECOS field: Thurman Munsom
  Login shell: /bin/sh
  Kerberos principal: tmunsom@IDMQE.LAB.ENG.BOS.REDHAT.COM
  Email address: tmunsom@idmqe.lab.eng.bos.redhat.com
  UID: 934200001
  GID: 934200001
  Password: False
  Kerberos keys available: False

# ipa user-mod tmunsom --password
Password: 
Enter Password again to verify: 
-----------------------
Modified user "tmunsom"
-----------------------
  User login: tmunsom
  First name: Thurman
  Last name: Munsom
  Home directory: /home/tmunsom
  Login shell: /bin/sh
  Email address: tmunsom@idmqe.lab.eng.bos.redhat.com
  UID: 934200001
  GID: 934200001
  Account disabled: False
  Password: True
  Member of groups: ipausers
  Kerberos keys available: True

# ipa group-find
----------------
4 groups matched
----------------
  Group name: admins
  Description: Account administrators group
  GID: 934200000
  Member users: admin

  Group name: editors
  Description: Limited admins who can edit other users
  GID: 934200002

  Group name: ipausers
  Description: Default group for all users
  Member users: tmunsom

  Group name: trust admins
  Description: Trusts administrators group
  Member users: admin
----------------------------
Number of entries returned 4
----------------------------

# ipa user-find
---------------
2 users matched
---------------
  User login: admin
  Last name: Administrator
  Home directory: /home/admin
  Login shell: /bin/bash
  UID: 934200000
  GID: 934200000
  Account disabled: False
  Password: True
  Kerberos keys available: True

  User login: tmunsom
  First name: Thurman
  Last name: Munsom
  Home directory: /home/tmunsom
  Login shell: /bin/sh
  Email address: tmunsom@idmqe.lab.eng.bos.redhat.com
  UID: 934200001
  GID: 934200001
  Account disabled: False
  Password: True
  Kerberos keys available: True
----------------------------
Number of entries returned 2
----------------------------

# firefox https://cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com
admin: 87654321
```

### Install Identity Management Client

1. Perform prerequisite steps by opening necessary ports in the firewall. System ports for IdM services include HTTP/HTTPS (80/tcp and 443/tcp), LDAP/LDAPS(389/tcp and 636/tcp), Kerberos (88 and 464, both tcp and udp), DNS (53, both tcp and udp), NTP (123/udp), and Certificate System (7389/tcp).

If the IdM server was configured as the DNS server, ensure that the client machine points to the IdM server first for DNS by making it the first entry in /etc/resolv.conf.

```
# iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination 

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination 
```

2. Install IdM client packages. Installing ipa-client will typically install a number dependencies. If the IdM client will also function as an administrator machine, then you must install one additional package: ipa-admintools

```
# yum install -y ipa-client
# yum install -y ipa-admintools
```

3. Enroll your desktopX system as a client of the earlier created IdM server serverX.example.com in the domain example.com using the credentials of admin and 87654321 enabling the PAM home directory module. 

```
# ipa-client-install --mkhomedir

The same result could be accomplished by passing the following arguments to the ipa-client-install command.
# ipa-client-install --domain=idmqe.lab.eng.bos.redhat.com --server=cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com --realm=IDMQE.LAB.ENG.BOS.REDHAT.COM -p admin -w 87654321 --mkhomedir -U
Hostname: cloud-qe-16-vm-04.idmqe.lab.eng.bos.redhat.com
Realm: IDMQE.LAB.ENG.BOS.REDHAT.COM
DNS Domain: idmqe.lab.eng.bos.redhat.com
IPA Server: cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com
BaseDN: dc=idmqe,dc=lab,dc=eng,dc=bos,dc=redhat,dc=com

Synchronizing time with KDC...
Unable to sync time with IPA NTP server, assuming the time is in sync. Please check that 123 UDP port is opened.
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=IDMQE.LAB.ENG.BOS.REDHAT.COM
    Issuer:      CN=Certificate Authority,O=IDMQE.LAB.ENG.BOS.REDHAT.COM
    Valid From:  Thu Jan 26 04:41:51 2017 UTC
    Valid Until: Mon Jan 26 04:41:51 2037 UTC

Enrolled in IPA realm IDMQE.LAB.ENG.BOS.REDHAT.COM
Attempting to get host TGT...
Created /etc/ipa/default.conf
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm IDMQE.LAB.ENG.BOS.REDHAT.COM
trying https://cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com/ipa/xml
Forwarding 'env' to server u'https://cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com/ipa/xml'
Adding SSH public key from /etc/ssh/ssh_host_dsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Forwarding 'host_mod' to server u'https://cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com/ipa/xml'
Could not update DNS SSHFP records.
SSSD enabled
Configuring idmqe.lab.eng.bos.redhat.com as NIS domain
Configured /etc/openldap/ldap.conf
NTP enabled
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Client configuration complete.
```

4. Confirm successful configuration by testing lookup information of the standard user admin and group admins.

```
# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
# getent passwd admin
admin:*:934200000:934200000:Administrator:/home/admin:/bin/bash
# id admin
uid=934200000(admin) gid=934200000(admins) groups=934200000(admins)
# getent group admins
admins:*:934200000:admin
```

5. Cleanly remove this client system's enrollment in the domain.

```
# ipa-client-install --uninstall

After the machine completes the reboot, remove the CA certificate from desktopX.
# rm -f /etc/ipa/ca.crt
```

### Registering a Client System with Identify Management(IdM)

```
# yum install -y ipa-client ipa-admintools
# ipa-client-install --mkhomedir
# ipa-client-install --domain=example.com --server=server.example.com --realm=EXAMPLE.COM -p admin -w 87654321 --mkhomedir

# klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)

# kinit admin
Password for admin@IDMQE.LAB.ENG.BOS.REDHAT.COM: 

# getent passwd tmunsom
tmunsom:*:934200001:934200001:Thurman Munsom:/home/tmunsom:/bin/sh

# klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: admin@IDMQE.LAB.ENG.BOS.REDHAT.COM

Valid starting     Expires            Service principal
01/26/17 01:37:27  01/27/17 01:37:23  krbtgt/IDMQE.LAB.ENG.BOS.REDHAT.COM@IDMQE.LAB.ENG.BOS.REDHAT.COM

# ipa user-find
---------------
2 users matched
---------------
  User login: admin
  Last name: Administrator
  Home directory: /home/admin
  Login shell: /bin/bash
  UID: 934200000
  GID: 934200000
  Account disabled: False
  Password: True
  Kerberos keys available: True

  User login: tmunsom
  First name: Thurman
  Last name: Munsom
  Home directory: /home/tmunsom
  Login shell: /bin/sh
  Email address: tmunsom@idmqe.lab.eng.bos.redhat.com
  UID: 934200001
  GID: 934200001
  Account disabled: False
  Password: True
  Kerberos keys available: True
----------------------------
Number of entries returned 2
----------------------------

# ssh tmunsom@desktop.example.com
# ssh tmunsom@cloud-qe-16-vm-04.idmqe.lab.eng.bos.redhat.com
tmunsom@cloud-qe-16-vm-04.idmqe.lab.eng.bos.redhat.com's password: 
Password expired. Change your password now.

WARNING: Your password has expired.
You must change your password now and login again!
Changing password for user tmunsom.
Current Password: 
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
Connection to cloud-qe-16-vm-04.idmqe.lab.eng.bos.redhat.com closed.

# ssh tmunsom@cloud-qe-16-vm-04.idmqe.lab.eng.bos.redhat.com
-sh-4.1$ pwd
/home/tmunsom
```

```
# ipa-client-install --help
Usage: ipa-client-install [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit

  basic options:
    --domain=DOMAIN     domain name
    --server=SERVER     IPA server
    --realm=REALM_NAME  realm name
    --fixed-primary     Configure sssd to use fixed server as primary IPA
                        server
    -p PRINCIPAL, --principal=PRINCIPAL
                        principal to use to join the IPA realm
    -w PASSWORD, --password=PASSWORD
                        password to join the IPA realm (assumes bulk password
                        unless principal is also set)
    -W                  Prompt for a password to join the IPA realm
    --mkhomedir         create home directories for users on their first login
    --hostname=HOSTNAME
                        The hostname of this machine (FQDN). If specified, the
                        hostname will be set and the system configuration will
                        be updated to persist over reboot. By default a
                        nodename result from uname(2) is used.
    --ntp-server=NTP_SERVER
                        ntp server to use
    -N, --no-ntp        do not configure ntp
    --ssh-trust-dns     configure OpenSSH client to trust DNS SSHFP records
    --no-ssh            do not configure OpenSSH client
    --no-sshd           do not configure OpenSSH server
    --no-dns-sshfp      do not automatically create DNS SSHFP records
    --noac              do not use Authconfig to modify the nsswitch.conf and
                        PAM configuration
    -f, --force         force setting of LDAP/Kerberos conf
    -d, --debug         print debugging information
    -U, --unattended    unattended (un)installation never prompts the user
    --ca-cert-file=CA_CERT_FILE
                        load the CA certificate from this file

  SSSD options:
    --permit            disable access rules by default, permit all access.
    --enable-dns-updates
                        Configures the machine to attempt dns updates when the
                        ip address changes.
    --no-krb5-offline-passwords
                        Configure SSSD not to store user password when the
                        server is offline
    -S, --no-sssd       Do not configure the client to use SSSD for
                        authentication
    --preserve-sssd     Preserve old SSSD configuration if possible

  uninstall options:
    --uninstall         uninstall an existing installation. The uninstall can
                        be run with --unattended option
```


###### Workshop: Install Identity Management Client

###### Quiz: Identity Management Client Systems

### Unit Test: Prepare Red Hat Identity Management

### nscd

*    nscd（Name Service Cache Daemon,名称服务缓存守护进程）会缓存三种服务passwd group hosts，所以它会记录三个库，分别对应源/etc/passwd, /etc/hosts 和 /etc/resolv.conf每个库保存两份缓存，一份是找到记录的，一份是没有找到记录的。每一种缓存都保存有生存时间（TTL）。其作用就是在本当中增加cache ，加快如DNS的解析等的速度。
*    在系统上安装nscd进行DNS进行缓存时，原则上 效率会大大地提高。不过不建议在DNS服务器上安装nscd，因为缓存地址需要额外的维护，而这对于DNS服务器的功能毫无益处。
*    默认该服务在redhat或centos下是关闭的，可以通过services nscd start开启。缓存DB文件在/var/db/nscd下。可以通过nscd -g查看统计的信息
*    通过编辑/etc/nscd.conf文件，开启本地DNS cache:  enable-cache hosts yes
