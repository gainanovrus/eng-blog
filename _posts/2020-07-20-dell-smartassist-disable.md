---
published: true
layout: single
title: DELL. Disabling SupportAssist on switch
excerpt: >-
  SupportAssist is a daemon for sending technical reports to Dell servers. It enables by default.
  Here we disable this unwanted feature (and may be unsecured).
categories: sysad
tags: dell networks security
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

I am continuing write articles about DELL Networking System (OS9 series). This short note will show how to disable SupportAssist, enabled by default installation.

## About SupportAssist

SupportAssist sends troubleshooting data securely to Dell. SupportAssist does not support automated email notification at the time of hardware fault alert, automatic case creation, automatic part dispatch, or reports. SupportAssist requires Dell EMC Networking OS9.

## About problem

Every time I log in to my Dell switch I see this:
```
 The SupportAssist EULA acceptance option has not been selected. SupportAssist
 can be enabled once the SupportAssist EULA has been accepted. Use the:
 'support-assist activate' command to accept EULA and enable SupportAssist.
```

So I wand to disable this. I would try running this command (in `CONF` mode):
```
Dell# conf
Dell(conf)# eula-consent support-assist reject
```

The result will be:
```
I do not accept the terms of the license agreement. The SupportAssist feature has
been deactivated and can no longer be used.
To enable SupportAssist configurations, accept the terms of the license agreement
by configuring this command 'eula-consent support-assist accept'.
```

## Check the status

If you enter the command `show eula-consent support-assist` you will see a status (`Rejected`) of this feature:

```
Dell# show eula-consent support-assist

SupportAssist EULA has been: Rejected

Additional information about the SupportAssist EULA is as follows:

By installing SupportAssist, you allow Dell EMC to save your contact information
(e.g. name, phone number and/or email address) which would be used to provide
technical support for your Dell EMC products and services.  Dell EMC may use the information
for providing recommendations to improve  your IT infrastructure.

Dell EMC SupportAssist also collects and stores machine diagnostic information, which
may include but is not limited to configuration information, user supplied contact
information, names of data volumes, IP addresses, access control lists, diagnostics &
performance information, network configuration information, host/server configuration
& performance information and related data ("Collected Data") and transmits this
information to Dell EMC. By downloading SupportAssist and agreeing to be bound by these
terms and the Dell EMC end user license agreement, available at: https://i.dell.com
/sites/doccontent/legal/terms-conditions/en/Documents/E-EULA_01June2018.pdf,
you agree to allow Dell EMC to provide remote monitoring services of your IT environment
and you give Dell EMC the right to collect the Collected Data in accordance with Dell EMC's
Privacy Policy, available at: https://www.dell.com/learn/us/en/uscorp1
/policies-privacy-country-specific-privacy-policy, in order to
enable the performance of all of the various functions of SupportAssist during your
entitlement to receive related repair services from Dell EMC. You further agree to
allow Dell EMC to transmit and store the Collected Data from SupportAssist in accordance
with these terms. You agree that the provision of SupportAssist may involve
international transfers of data from you to Dell EMC and/or to Dell EMC's affiliates,
subcontractors or business partners. When making such transfers, Dell EMC shall ensure
appropriate protection is in place to safeguard the Collected Data being transferred
in connection with SupportAssist. If you are downloading SupportAssist on behalf
of a company or other legal entity, you are further certifying to Dell EMC that you
have appropriate authority to provide this consent on behalf of that entity. If you
do not consent to the collection, transmission and/or use of the Collected Data,
you may not download, install or otherwise use SupportAssist.
```


## Conclusion

This short post that could be packed in one command.
But I want to point an importance of security.
Any communication with external system might be used as a back-door attacker.
The SupportAssist has have a vulnerability in the past. Read the [post](https://xakep.ru/2019/06/24/supportassist/) about it.

## Additional information

* [MXL - how to get rid of the nag about Support Assist](https://www.dell.com/community/Networking-General/MXL-how-to-get-rid-of-the-nag-about-Support-Assist/td-p/5037926) - user question on the Dell forum about the same
* [Dell Command Line Reference Guide for the S4048–ON System](https://www.dell.com/support/manuals/us/en/19/force10-s4048-on/s4048-on-9.14.0.0-cli/about-this-guide) - dell man page included full command list
