定时启动（每天5点）：
0  5    * * *   root    /config/scripts/post-config.d/gfwlist2dnsmasq.sh

也可通过以下命令设置策略路由：

configure

set firewall group address-group gfwlist

set protocols static table 5 route 0.0.0.0/0 next-hop 192.168.1.254

set firewall modify AUTO_GW rule 2000 action modify

set firewall modify AUTO_GW rule 2000 destination group address-group gfwlist

set firewall modify AUTO_GW rule 2000 modify table 5

set firewall modify AUTO_GW rule 2000 protocol all

set interfaces ethernet eth1 firewall in modify AUTO_GW

commit;save;exit


service dnsmasq restart

输出配置：
mca-ctrl -t dump-cfg


# gfwlist2dnsmasq
A shell script which convert gfwlist into dnsmasq rules.

Working on both Linux-based (Debian/Ubuntu/Cent OS/OpenWrt/LEDE/Cygwin/Bash on Windows/etc.) and BSD-based (FreeBSD/Mac OS X/etc.) system.

This script needs `sed`, `base64`, `curl`(or`wget`). You should have these binaries on you system.

### Usage
```
sh gfwlist2dnsmasq.sh [options] -o FILE
Valid options are:
    -d, --dns <dns_ip>
                DNS IP address for the GfwList Domains (Default: 127.0.0.1)
    -p, --port <dns_port>
                DNS Port for the GfwList Domains (Default: 5353)
    -s, --ipset <ipset_name>
                Ipset name for the GfwList domains
                (If not given, ipset rules will not be generated.)
    -o, --output <FILE>
                /path/to/output_filename
    -i, --insecure
                Force bypass certificate validation (insecure)
    -l, --domain-list
                Convert Gfwlist into domain list instead of dnsmasq rules
                (If this option is set, DNS IP/Port & ipset are not needed)
        --exclude-domain-file <FILE>
                Delete specific domains in the result from a domain list text file
                Please put one domain per line
        --extra-domain-file <FILE>
                Include extra domains to the result from a domain list text file
                This file will be processed after the exclude-domain-file
                Please put one domain per line
    -h, --help  Usage
```

### OpenWRT Usage

( For LEDE 17.01/ OpenWrt 18.06 and later)

To download gfwlist `curl` or `wget` is needed. Because the connection is HTTPS, if you use busybox `wget`, you need to install `libustream-openssl` or `libustream-mbedtls` to support it, otherwise use GNU `wget`.

Because gfwlist is encoded by BASE64, `base64` is needed to decode.

```
# curl
opkg update
opkg install curl coreutils-base64
# busybox wget (default by OpenWrt)
opkg update
opkg install libustream-mbedtls coreutils-base64
# GNU wget
opkg update
opkg install wget coreutils-base64
```

For security reason, this script won't bypass HTTPS certificate validation. So you should install ca-certificates and ca-bundle in addition.

```
opkg update
opkg install ca-certificates ca-bundle
```

If you really want to bypass the certificate validation, use '-i' or '--insecure' option. You should know this is insecure.

### Generated Configuration Files [![Build Status](https://travis-ci.org/cokebar/gfwlist2dnsmasq.svg?branch=master)](https://travis-ci.org/cokebar/gfwlist2dnsmasq)

If you don't want to generate dnsmasq configuration file by yourself, you can directly download them:

- gfwlist to dnsmasq rule file without ipset: https://cokebar.github.io/gfwlist2dnsmasq/dnsmasq_gfwlist.conf

- gfwlist to dnsmasq rule file with ipset: https://cokebar.github.io/gfwlist2dnsmasq/dnsmasq_gfwlist_ipset.conf

- gfwlist to domain list file: https://cokebar.github.io/gfwlist2dnsmasq/gfwlist_domain.txt
