---
layout: post
title:  "Tracking Security Updates(413-1)"
categories: Linux
tags: 413
---

### The Red Hat security response

1. Security Response Team(secalert@redhat.com)

2. Investigates and Verifies issues:

*    which products are affected
*    determine the impact
*    determine the remedial action
*    security updates is produced

3. Publishing Red Hat Security Advisories

4. Subscribing to any of the following mailing lists:

*    rhsa-announce
*    enterprise-watch-list
*    jboss-watch-list
*    rhev-watch-list

https://access.redhat.com/security/updates/advisory

https://access.redhat.com/security/team  


### Red Hat severity scoring

The Red Hat Security Response Team rates the impact of security issues found in Red Hat products using a four point scale:

*    Low impact
*    Moderate impact
*    Important impact
*    Critical impact


### What are CVEs and Red Hat Errata?

Common Vulnerabilities and Exposures(CVE) is a standardized format for reporting and tracking security related software issues.    
MITRE Corporation, Community web presence    
National Vulnerability Database(NVD), U.S. National institutes of standards and Technology(NIST)    
Red Hat will issue an errata announcement:

*    RHSA: fix a security related issue
*    RHBA: fix a non-security bug
*    RHEA: add an additional enhancement

https://cve.mitre.org
https://nvd.nist.gov
https://rhn.redhat.com/errata

[Red Hat Vulnerabilities by CVE name](https://access.redhat.com/security/cve)

###### Performance Checklist: Using yum updateinfo

```
1. Install yum security plugin
# yum list yum-plugin-security
# yum install -y yum-plugin-security

2. Generate a report containing RHSA RHBA RHEA
# yum updateinfo
# yum updateinfo list
# yum updateinfo list bugfix
# yum updateinfo list enhancement
# yum updateinfo list security

3. View the contents of RHSA RHSA-2016:0820 and CVE-2016-0775
# yum updateinfo list RHSA-2016:0820
# yum updateinfo RHSA-2016:0820

# yum updateinfo list cve
# yum updateinfo list cve --cve=CVE-2016-0775

```

### Discuss package maintenance through backporting

###### An Example of Why We Backport Security Fixes

### Engaging the Red Hat Security Response Team

You should contact Security Response Team if:

Who Reads Email Sent to secalert@redhat.com?

### Unit Test: Examining Red Hat Updates

1. How many security notices, bugfix and enhancement notices are available for your machine?
2. How many security related packages are available for your machine?
3. How many unique, ciritical severity RHSA are available for your machine?
4. yum updateinfo list xulrunner(which update rpm need to be installed on this machine to ensure all RHSA have been resolved?)

```
1.
# yum updateinfo
  29 Security notice(s)
     4 Critical Security notice(s)
     8 Important Security notice(s)
    17 Moderate Security notice(s)
  26 Bugfix notice(s)
   3 Enhancement notice(s)

2. 
# yum list updates --security
54 package(s) needed for security, out of 82 available

3. 
# yum updateinfo
  29 Security notice(s)
     4 Critical Security notice(s)
     8 Important Security notice(s)
    17 Moderate Security notice(s)
  26 Bugfix notice(s)
   3 Enhancement notice(s)

4.
# yum updateinfo list xulrunner
RHSA-2013:0271 Critical/Sec. xulrunner-17.0.3-1.el6_3.x86_64
RHSA-2013:0614 Critical/Sec. xulrunner-17.0.3-2.el6_4.x86_64
RHSA-2013:0696 Critical/Sec. xulrunner-17.0.5-1.el6_4.x86_64
RHSA-2013:0820 Critical/Sec. xulrunner-17.0.6-2.el6_4.x86_64
```

### Examples

```
1. How many security notices, bugfix notices, and enhancement notices are available for your machine?
There are 29 security notices, 26 bugfix notices, and 3 enhancement notices available for this machine.
# yum updateinfo
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Updates Information Summary: available
    29 Security notice(s)
         4 Critical Security notice(s)
         8 Important Security notice(s)
        17 Moderate Security notice(s)
    26 Bugfix notice(s)
     3 Enhancement notice(s)
updateinfo summary done

2. How many security related packages are available for your machine?
There are 54 security updates available.
# yum --security list updates
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Limiting package lists to security relevant ones
54 package(s) needed for security, out of 82 available
Updated Packages
bind-libs.x86_64                          32:9.8.2-0.17.rc1.el6_4.4      Updates
bind-utils.x86_64                         32:9.8.2-0.17.rc1.el6_4.4      Updates
... Output Truncated ...

3. How many unique, critical severity Red Hat Security Advisories (RHSAs) are available for your machine?
There are 4 unique, critical severity RHSAs available.
# yum updateinfo
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Updates Information Summary: available
    29 Security notice(s)
         4 Critical Security notice(s)
         8 Important Security notice(s)
        17 Moderate Security notice(s)
    26 Bugfix notice(s)
     3 Enhancement notice(s)
updateinfo summary done

Alternatively...
# yum updateinfo list | grep 'Critical' | cut -f1 -d' ' | sort -u | wc -l
4

4. Given the following example updateinfo output, which updated RPM(s) need to be installed on this machine to ensure all Red Hat Security Advisories have been resolved?
# yum updateinfo list qemu-kvm
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Updating certificate-based repositories.
RHBA-2012:1121 bugfix         qemu-kvm-2:0.12.1.2-2.295.el6_3.1.x86_64
RHSA-2012:1234 Important/Sec. qemu-kvm-2:0.12.1.2-2.295.el6_3.2.x86_64
RHBA-2012:1585 bugfix         qemu-kvm-2:0.12.1.2-2.295.el6_3.5.x86_64
RHBA-2012:1519 bugfix         qemu-kvm-2:0.12.1.2-2.295.el6_3.8.x86_64
RHBA-2012:1582 bugfix         qemu-kvm-2:0.12.1.2-2.295.el6_3.10.x86_64
RHBA-2013:0527 bugfix         qemu-kvm-2:0.12.1.2-2.355.el6.x86_64
RHBA-2013:0539 bugfix         qemu-kvm-2:0.12.1.2-2.355.el6_4.1.x86_64
RHSA-2013:0609 Important/Sec. qemu-kvm-2:0.12.1.2-2.355.el6_4.2.x86_64
updateinfo list done

This machine would need qemu-kvm-2:0.12.1.2-2.355.el6_4.2.x86_64 applied to it to resolve all outstanding Red Hat Security Advisories. Later update RPMs include all previously released enhancements, bugfixes, and security updates, which means administrators need only apply the latest available security related update to satisfy all currently outstanding previous Security Advisories for the package.

5. Given the following example updateinfo output, which updated RPM(s) need to be installed to ensure all Red Hat Security Advisories have been resolved?
# yum updateinfo list openldap
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
Updating certificate-based repositories.
RHSA-2012:1151 Low/Sec. openldap-2.4.23-26.el6_3.2.x86_64
RHBA-2013:0364 bugfix   openldap-2.4.23-31.el6.x86_64
RHBA-2013:0598 bugfix   openldap-2.4.23-32.el6_4.x86_64
updateinfo list done

This machine would need openldap-2.4.23-26.el6_3.2.x86_64 applied to it to resolve all outstanding Red Hat Security Advisories have been resolved. The later updates are bugfix, not security related.

6. How would you view a synopsis of Red Hat Security Advisory RHSA-2013:0689?
# yum updateinfo RHSA-2013:0689
===============================================================================
  Important: bind security and bug fix update
===============================================================================
  Update ID : RHSA-2013:0689
    Release : 
       Type : security
     Status : final
     Issued : 2013-03-28 00:00:00
... Output Truncated ...

7. How would you generate a list of all packages that would need to be updated to resolve RHSA-2013:0689?
# yum updateinfo list --advisory=RHSA-2013:0689
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
RHSA-2013:0689 Important/Sec. bind-libs-32:9.8.2-0.17.rc1.el6_4.4.x86_64
RHSA-2013:0689 Important/Sec. bind-utils-32:9.8.2-0.17.rc1.el6_4.4.x86_64
updateinfo list done

8. How many packages would need to be updated to resolve Common Vulnerability and Exposures number CVE-2012-6329?
The machine would need 12 package updates to resolve CVE-2012-6329.
# yum updateinfo list --cve=CVE-2012-6329
Loaded plugins: product-id, refresh-packagekit, security, subscription-manager
RHSA-2013:0685 Moderate/Sec. perl-4:5.10.1-130.el6_4.x86_64
RHSA-2013:0685 Moderate/Sec. perl-CGI-3.51-130.el6_4.x86_64
RHSA-2013:0685 Moderate/Sec. perl-ExtUtils-MakeMaker-6.55-130.el6_4.x86_64
RHSA-2013:0685 Moderate/Sec. perl-ExtUtils-ParseXS-1:2.2003.0-130.el6_4.x86_64
RHSA-2013:0685 Moderate/Sec. perl-Module-Pluggable-1:3.90-130.el6_4.x86_64
RHSA-2013:0685 Moderate/Sec. perl-Pod-Escapes-1:1.04-130.el6_4.x86_64
... Output Truncated ...
```
