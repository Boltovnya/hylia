---
title: Unbinding the knots of DNS
date: 2020-06-08T13:39:00.000Z
tags:
  - homelab
  - DNS
  - Knot
  - Knot-Resolver
  - Unbound
  - ISC DHCP
---
## F@#K DNS!!1!

Before I kick this post off properly, I just want to take a moment to send forward my appreciation and thanks to those out in the field, those who look after this monstrosity of a service on a day-to-day basis - keeping it tame and well-kept so that we, the unworthy, can resolve our hostnames safely and happily.

> Seriously, I wouldn't wish this fate on my worst enemy

- - -

## The plan

I recently purchased a Unifi Security Gateway to complete the home network (having already had a Unifi AP), and after the rigmarole of setting that up, I wanted an even greater level of control of what's happening within the network boundary. Partly fueled by a constant urge to do better, partly fueled by the constant frustration of my old router's tendancy to inject horrific, non-specific hostnames into DNS queries which had an absurdly long TTL, I just had to fix this...

```
                                      +--------------+
                  +------------------>| RapsberryPi? |
                  |                   +--------------+
                  |
                  |                   +----------+
                  |           /------>| Hue Hub? |
          +-------+----------+        +----------+
          | Computer.connect |
          +-------+----------+        +----------+
                  |           \------>| Macbook? |
                  |                   +----------+
                  |
                  |                   +-------------+
                  +------------------>| Thermostat? |
                                      +-------------+
```

The plan was pretty simple, install an authoritative DNS server capable of accepting [RFC2136](https://tools.ietf.org/html/rfc2136) updates, and a recursive server for forwarding and resolving requests not answered by the authoritative server. Easy peasy, right?

> It was not, in fact, easy peasy

- - -

## DNS, Part 1

#### Technologies: [Knot DNS](https://www.knot-dns.cz/), [Knot-Resolver](https://www.knot-resolver.cz/)

Now, I realise that I'm rusty when it comes to network-based infrastructure. It's been over 5 years since my ICND1+2 courses and I've never really been in a position where I've had to deploy or configure DNS since, but nothing could have prepared me for how difficult, frustrating and completely non-sensical this setup was. 

The idea of using Knot and Knot-Resolver first came from a friend through [TARDIS](https://wiki.tardis.ed.ac.uk), who had been recommending it as an easy to deploy and straight forward DNS solution. This ticked the boxes necessary for me to take the plunge in deploying my own DNS.

- - -

![img](/images/unbound-knot-dns-discord.png) 

- - -

### Let's get automatin'

With that, I got to work on planning my deployment. My friend had sent me a copy of their config that I could essentially just copy and paste into my own. As knotd config is YAML based and kresd config only requires one or two lines, I decided that I would try installing and deploying a service from scratch using Ansible.

And all credit to Knot, the config is super simple and easy to template:

```yaml
---
/etc/knot/knot.conf.j2
{% raw %}
server:
  identity:  {{ ansible_fqdn }} 
  user: knot:knot
  listen: [ {{ ansible_default_ipv4["address"] }}@55 ]

log:
  - target: syslog
    any: info

remote:
  - id: resolver
    address: {{ ansible_default_ipv4["address"]@54 }}
    
mod-dnsproxy:
  - id: default
    remote: resolver
    fallback: on

template:
  - id: default
    storage: "/var/lib/knot"
    file: "%s.zone"
    semantics-checks: on
    global-module: mod-dnsproxy/default
      
zone:
  - domain: {{ domain_name }}
    storage: /var/lib/knot/
    file: {{ domain_name }}.zone
{% endraw %} 
```

As is the knot-resolver config:

```lua
--- /etc/knot-resolver/kresd.conf.j2
{% raw %}
net.listen("{{ ansible_default_ipv4["address"] }}", 54)
policy.add(policy.all(policy.FORWARD({"208.67.222.222", "208.67.220.220"})))
{% endraw %}
```

With the config out the way, we need to actually write a playbook to write it, right? The actual install is fairly simple as well, all you need is to enable a few repos here and there and you're all set to install.

```yaml
- hosts: dns_host
  become: yes
  vars:
    - domain_name: fox.den
  tasks:
    - name: Install EPEL
      yum: ...

    - name: Enable codeready builder
      rhsm_repository: ...

    - name: Install Knot-resolver repo
      yum: ...
      
    - name: Update yum
      yum: ...

    - name: Install knot-resolver package
      yum: ...

    - name: Install Knot
      yum: ...
    
    - name: Install Knot config
      template: ...
      
    - name: Install KRESD config
      template: ...
```

So, we run the playbook and we're done? That's it? That's all there is to it?

> No.

- - -

### This is where the fun begins

Something the Knot documentation neglects to tell you is that you need to set up your zone file manually. If you don't, you'll have `knot` screaming at you through systemd that your zone doesn't exist and blah blah blah. tl;dr - you need to build a zone.

And this is really easy with Knot! Knot provides `knotc` to control config and zone changes. No need to wrangle any horrible BIND-based zone syntax, it's all done for you through this utility. I don't understand why it isn't well documented, because I'd want to scream about how useful `knotc` is from the rooftops!

Anyway, we should probably launch `knotc` and make sure our config is okay. If everything is good, we should get an error telling us we're trying to create a duplicate zone.

```bash
[root@master-namer ~]# knotc
knotc> conf-begin
OK
knotc> conf-set zone.domain fox.den
error: (duplicate identifier) zone.domain = fox.den
knotc> conf-commit
OK
knotc> 
```

Cool - we're ready to start building our zone! This couldn't be simpler and can be done in as little as 5 lines.

```bash
knotc> zone-begin fox.den
OK
knotc> zone-set fox.den @ 7200 SOA ns hostmaster 1 86400 900 691200 3600
OK
knotc> zone-set fox.den @ 21600 NS master-namer
OK
knotc> zone-set fox.den master-namer 86400 A 10.0.2.3
OK
knotc> zone-commit
OK
```

And with that out the way, all I need to do is update the DNS and search domain settings on my USG DHCP server and away we go! 

```bash
☁  ~  nslookup google.com
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   google.com
Address: 216.58.210.206

☁  ~  nslookup roke
Server:         10.0.2.3
Address:        10.0.2.3#53
** server can't find roke: NXDOMAIN
```

> Nope. 

So we can resolve to the Internet, but not local addresses. What am I forgetting... I don't know. It's just not working. Is there any point to running an authoritative server if I can't resolve local hostnames? Not really...

- - -

## DNS, Part 2

#### Technologies: [Unbound](https://nlnetlabs.nl/projects/unbound/about/), [ISC DHCP](https://www.isc.org/dhcp/) - [Ansible files](#)

### Down the rabbit hole

On the recommendation of another friend, I decided that I'd given up with Knot/Kresd and focused my attention on just getting a resolver working. Enter: Unbound. 

> Unbound is a validating, recursive, caching DNS resolver. 

Unbound is a recursive DNS server which can host its own zones. A little backwards, but it was enough for what I needed. The setup is also super simple and hands-off (especially with a neat little [Ansible Role](https://galaxy.ansible.com/mrlesmithjr/unbound) on Galaxy to handle it all for me!)

```yaml
---
setup-unbound.yml
roles:
    - role: mrlesmithjr.unbound
      vars:
        config_unbound: true
        unbound_access_control:
          - subnet: 10.0.0.0/8
            action: allow
        unbound_authoritative_zones:
          - zone: 'fox.den'
            a_records:
              - name: "roke"
                ip: 10.0.2.1
              - name: "master-namer"
                ip: 10.0.2.3
              - name: "master-chanter"
                ip: 10.0.2.2
            create_ptr_records: true
        unbound_forward_zones:
          - zone: '.'
            forward_addresses: 
              - 208.67.222.222
              - 208.67.220.220
        unbound_listen_interfaces:
          - 0.0.0.0
        unbound_log_queries: true
```

And that was, in fact, all I needed to configure the DNS server. All that was left to do was enable the DNS service in firewalld to allow traffic and we were good to go.

```bash
☁  ~  nslookup google.com
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   google.com
Address: 216.58.210.206

☁  ~  nslookup roke
Server:         10.0.2.3
Address:        10.0.2.3#53

Name:   roke.fox.den
Address: 10.0.2.1
```

Now, no sensible person wants to manually manage all DNS records on their network - that's just madness. After a quick google search, I figured all I needed was a DHCP server to process hostnames as hosts made requests. Simple.

Of course, there's an [Ansible Role](https://galaxy.ansible.com/adfinis-sygroup/isc_dhcp_server) for that as well, so after disabling DHCP on my USG, I installed the role and configured the firewall again.

```yaml
---
setup-unbound.yml
roles:
  - role: adfinis-sygroup.isc_dhcp_server
    vars:
      isc_dhcp_server_subnet:
        - netaddress: 10.0.0.0
          netmask: 255.0.0.0
          routers: 10.0.0.1
          domain: fox.den
          domain_search: fox.den
          dns: 10.0.2.3
          range: 10.5.0.0 10.6.0.0
...
tasks:
  - name: Enable DHCP
    firewalld:
      service: dhcp
      permanent: yes
      state: enabled
    notify: Enable and Restart Services
```

So we've got a DNS server and a DHCP server. DHCP issues the correct DNS server with each lease. So why can't I resolve my laptop?

Of course, the age-old adage of RTFM still rings true to this day. Unbound is only a caching resolver, though the DHCP server will attempt to update dynamic DNS entries, there's nothing for Unbound to actually update.

Well... I've gone through enough pain with DNS already, I thought. And I depend on a stable internet connection for work. I should probably leave things as they are, I don't need name resolution *that* badly. I can resolve external names and core internal services, I should be happy with that.

- - -

> ### But she wasn't happy with just that

## DNS, Part 3

#### Technologies: Knot, Knot-Resolver, ISC-DHCP - [Ansible Role](https://gitlab.com/Boltovnya/foxden.knot)

At this point, I was at my wit's end. I just wanted to be able to resolve local hostnames ad-hoc and not have to worry about it. Is that too much to ask!? Apparently so, but nonetheless I persevered.

It was then that I conceded that I don't know enough about this sorta stuff to confidently do it by myself, so reached out to the friend who originally pointed me in the direction of Knot for a quick lesson on DNS. They were able to explain it in such terms that I was immediately able to understand it and I am forever grateful to have such a diverse and knowledgable social network that enables that.

> Authoritative servers have the answers for their zones. Recursive servers search far and wide for answers. You need DDNS to dynamically update your local records, so you need an authoritative server to store those records.

I decided to go back to Knot and Kresd after this, with an understanding of how I should proceed. Because I already had all the Ansible files from my first attempt at Knot, I ended up writing a role for this to save me the effort of deploying manually again (Link above). 

So as before, I installed Knot and Kresd and configured my zones. Resolving works. Hurrah! A few config items had to be changed to allow for dynamic address updates, but this wasn't significant - involving only a few extra lines in each config file.

First in Knot.conf to allow for dynamic updates. This is done simply by creating a TSIG key and an ACL associated with the key and assigning it to each zone.

```yaml
---
/etc/knot/knot.conf
{% raw %}
...
key:
  - id: ddns_update
    algorithm: hmac-sha256
    secret: {{ hmac-sha256-key }}
...
acl:
  - id: acl_update
    address: 10.0.0.0/8
    action: update
    key: ddns_update
...
zone:
  - domain: fox.den
  ...
    acl: acl_update

  - domain: 10.in-addr.arpa
  ...
    acl: acl_update
{% endraw %}
```

Then in the `dhcpd` config, enabling dynamic updates and setting the zones and specifying the TSIG key to use.

```bash
---
/etc/dhcp/dhcpd.conf

...
ddns-updates on;
ddns-update-style interim;
ignore client-updates;
update-static-leases on;
...

include "/etc/dhcp/ddns.key";

zone fox.den. {
  primary 10.0.2.3;
  key DDNS_UPDATE;
}

zone 10.in-addr.arpa {
  primary 10.0.2.3;
  key DDNS_UPDATE;
}
```

And lastly creating a TSIG file readable only by `dhcpd`

```bash
---
/etc/dhcp/ddns.key
{% raw %}
key "DDNS_UPDATE" {
        algorithm hmac-sha256;
        secret {{hmac-sha256-secret}} ;
};
{% endraw %}
```

## Fin

The Fox Den now has proper DNS resolution and DDNS. However, this took over 4 days of sleepless nights to get to this point. I learned a lot about each protocol involved and about Ansible, which I think will appear more often than not in this blog.

In closing... fuck DNS.