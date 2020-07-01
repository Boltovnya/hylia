---
layout: layouts/page.njk
title: Tardis - FreeIPA Migration and Deployment
permalink: /tardis-freeipa/index.html
---
## Introduction

One of the actions identified after the Tardis meeting on June 28th was to revisit and revise the way of managing users and identities across the system. Tardis management hasn't evolved much in at least 5 years and therefore, an opportunity to modernise has arisen.

## Current state of affairs

Tardis authentication currently relies on an outdated version of OpenLDAP, with most administration being performed through Apache Directory Studio and a series of Python scripts.

## Plan of action
### tl;dr
```
1 - Deploy FreeIPA w/o Integrated DNS
2 - Update tardis.ed.ac.uk zone with IPA SRVs
    - # ipa dns-update-system-records --dry-run > ipa_bind.db
3 - Enable FreeIPA migration mode
    - # ipa config-mod --enable-migration=True
    - # ipa-compat-manage disable
    - # systemctl restart dirsrv.target
4 - Migrate OpenLDAP to FreeIPA
    - # ipa migrate-ds ldap://{{ ldap }}:389 --bind-dn="{{ fqdn }}"
5 - Disable migration mode
    - # ipa config-mod --enable-migration=False
    - # ipa-compate-manage enable
    - # systemctl restart dirsrv.target
6 - Install/enable IPA Client (Recommend doing this with Ansible)
    Per-host:
    - # apt install freeipa-client
    - # ipa-client-install
7 - ...
8 - Proffit
(Note: DNS management will be done in the same way as before)
```

### Full plan
(This is a living document (I know, clicheed), but will be updated as progress is made)
