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

