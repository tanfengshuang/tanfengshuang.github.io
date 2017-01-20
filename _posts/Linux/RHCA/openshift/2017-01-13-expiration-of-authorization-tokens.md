
---
layout: post
title:  "Expiration of authorization tokens"
categories: Linux
tags: openshift2
---

[Expiration of authorization tokens](https://access.redhat.com/solutions/2485951)


### Environment

    OpenShift Enterprise 2.2

### Issue

    In our Openshift 2.2 Production environment, we use automated scripts to access the gears to run checks. However, we are restricted by the authorization tokens which ssh onto the gears and allow the scripts to run.

Essentially, is there any way to make the authorization token expiration to be set to not expire?
Resolution

expires_in Total time in seconds before this authorization expires. Out of range values will be set to the maximum allowed time.
So expires_in = -1 appears to give a non-expiring token.

Check https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/2/html-single/REST_API_Guide/index.html#chap-Authorizations for details.

