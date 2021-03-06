- [About this repository](#about-this-repository)
- [About RESTCONF](#about-restconf)
- [EOS configuration](#eos-configuration)
  - [Generates a self-signed certificate](#generates-a-self-signed-certificate)
  - [Change the default control-plane ACL](#change-the-default-control-plane-acl)
  - [Configure RESTCONF](#configure-restconf)
  - [Verify](#verify)
- [Verify connectivity to the RESTCONF server](#verify-connectivity-to-the-restconf-server)
  - [Ping the device](#ping-the-device)
  - [Scan the open ports](#scan-the-open-ports)
  - [Verify if the device accepts TCP SYN on the port 6020](#verify-if-the-device-accepts-tcp-syn-on-the-port-6020)
  - [Try to open a TCP connection on the port 6020](#try-to-open-a-tcp-connection-on-the-port-6020)
- [RESTCONF examples](#restconf-examples)
  - [GET](#get)
    - [Using cURL](#using-curl)
    - [Using Python](#using-python)
  - [HEAD](#head)
    - [Using cURL](#using-curl-1)
    - [Using Python](#using-python-1)
  - [PUT](#put)
    - [Using cURL](#using-curl-2)
      - [Interface configuration example](#interface-configuration-example)
      - [Device hostname example](#device-hostname-example)
  - [POST](#post)
    - [Using cURL](#using-curl-3)
      - [Interface configuration example](#interface-configuration-example-1)
  - [DELETE](#delete)
    - [Using cURL](#using-curl-4)
    - [Using Python](#using-python-2)

# About this repository

RESTCONF examples with EOS

# About RESTCONF

RESTCONF is defined in the [RFC 8040](https://datatracker.ietf.org/doc/html/rfc8040)

- The GET method is sent by the client to retrieve data for a resource.
- The HEAD method is sent by the client to retrieve just the header fields (which contain the metadata for a resource) that would be returned for the comparable GET method, without the response message-body. It is supported for all resources that support the GET method.
- The POST method is sent by the client to create a data resource.
- The PUT method is sent by the client to create or replace the target data resource.
- The DELETE method is used to delete the target resource.

# EOS configuration

The RESTCONF server is in the EOS device.
## Generates a self-signed certificate

```
DC1-LEAF1A#security pki certificate generate self-signed restconf.crt key restconf.key generate rsa 2048 parameters common-name restconf
certificate:restconf.crt generated
DC1-LEAF1A#
```
## Change the default control-plane ACL

The default RESTCONF port on Arista devices is TCP 6020.
We need to change the default control-plane ACL on EOS in order to allow TCP 6020.
```
DC1-LEAF1A#show ip access-lists default-control-plane-acl
```
```
DC1-LEAF1A(config)#show ip access-lists def2
IP Access List def2
        9 permit tcp any any eq 6020
        10 permit icmp any any
        20 permit ip any any tracked [match 147 packets, 0:00:14 ago]
        30 permit udp any any eq bfd ttl eq 255
        40 permit udp any any eq bfd-echo ttl eq 254
        50 permit udp any any eq multihop-bfd
        60 permit udp any any eq micro-bfd
        70 permit udp any any eq sbfd
        80 permit udp any eq sbfd any eq sbfd-initiator
        90 permit ospf any any [match 1882 packets, 0:00:04 ago]
        100 permit tcp any any eq ssh telnet www snmp bgp https msdp ldp netconf-ssh gnmi
        110 permit udp any any eq bootps bootpc ntp snmp rip ldp
        120 permit tcp any any eq mlag ttl eq 255
        130 permit udp any any eq mlag ttl eq 255
        140 permit vrrp any any
        150 permit ahp any any
        160 permit pim any any [match 124 packets, 0:00:14 ago]
        170 permit igmp any any [match 90 packets, 0:01:24 ago]
        180 permit tcp any any range 5900 5910
        190 permit tcp any any range 50000 50100
        200 permit udp any any range 51000 51100
        210 permit tcp any any eq 3333
        220 permit tcp any any eq nat ttl eq 255
        230 permit tcp any eq bgp any
        240 permit rsvp any any
        250 permit tcp any any eq 6040
```
```
DC1-LEAF1A(config)#sh run sec control
system control-plane
   ip access-group def2 vrf MGMT in
```
## Configure RESTCONF
```
DC1-LEAF1A(config)#sh run sec restconf
management api restconf
   transport https test
      ssl profile restconf
      vrf MGMT
!
management security
   ssl profile restconf
      certificate restconf.crt key restconf.key
```
## Verify
```
DC1-LEAF1A(config)#show management api restconf
Enabled:            Yes
Server:             running on port 6020, in MGMT VRF
SSL Profile:        restconf
QoS DSCP:           none
```
```
DC1-LEAF1A(config)# show ip interface Management1 brief
                                                                              Address
Interface         IP Address           Status       Protocol           MTU    Owner
----------------- -------------------- ------------ -------------- ---------- -------
Management1       10.73.1.105/24       up           up                1500
```
```
DC1-LEAF1A(config)#show run interface Management1
interface Management1
   description oob_management
   vrf MGMT
   ip address 10.73.1.105/24
```


# Verify connectivity to the RESTCONF server

From the RESTCONF client:

```
lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.4 LTS
Release:        18.04
Codename:       bionic
```
```
python3 -V
Python 3.6.9
```
```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install nmap hping3 jq curl netcat -y
pip3 install requests
```
## Ping the device
```
ping 10.73.1.105
PING 10.73.1.105 (10.73.1.105) 56(84) bytes of data.
64 bytes from 10.73.1.105: icmp_seq=1 ttl=64 time=0.559 ms
64 bytes from 10.73.1.105: icmp_seq=2 ttl=64 time=0.486 ms
^C
--- 10.73.1.105 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1017ms
rtt min/avg/max/mdev = 0.486/0.522/0.559/0.043 ms
```
## Scan the open ports
```
nmap -p 6015-6025  10.73.1.105

Starting Nmap 7.60 ( https://nmap.org ) at 2021-06-29 00:04 UTC
Nmap scan report for 10.73.1.105
Host is up (0.00082s latency).

PORT     STATE    SERVICE
6015/tcp filtered x11
6016/tcp filtered x11
6017/tcp filtered xmail-ctrl
6018/tcp filtered x11
6019/tcp filtered x11
6020/tcp open     x11
6021/tcp filtered x11
6022/tcp filtered x11
6023/tcp filtered x11
6024/tcp filtered x11
6025/tcp filtered x11

Nmap done: 1 IP address (1 host up) scanned in 1.28 seconds
```
## Verify if the device accepts TCP SYN on the port 6020

```
sudo hping3 10.73.1.105 -p 6020 -S
HPING 10.73.1.105 (ens4 10.73.1.105): S set, 40 headers + 0 data bytes
len=46 ip=10.73.1.105 ttl=64 DF id=0 sport=6020 flags=SA seq=0 win=29200 rtt=3.7 ms
len=46 ip=10.73.1.105 ttl=64 DF id=0 sport=6020 flags=SA seq=1 win=29200 rtt=7.4 ms
len=46 ip=10.73.1.105 ttl=64 DF id=0 sport=6020 flags=SA seq=2 win=29200 rtt=7.1 ms
len=46 ip=10.73.1.105 ttl=64 DF id=0 sport=6020 flags=SA seq=3 win=29200 rtt=6.9 ms
len=46 ip=10.73.1.105 ttl=64 DF id=0 sport=6020 flags=SA seq=4 win=29200 rtt=6.6 ms
^C
--- 10.73.1.105 hping statistic ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 3.7/6.3/7.4 ms
```
## Try to open a TCP connection on the port 6020
```
nc -v -z 10.73.1.105 6020
Connection to 10.73.1.105 6020 port [tcp/*] succeeded!
```
# RESTCONF examples

## GET
### Using cURL

```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1/config/description' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:config"'
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:config".description'
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:state".counters."in-octets"'
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:interface"[].name'
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:interface"[2].name'
curl -X GET https://10.73.1.105:6020/restconf/data/system --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
curl -X GET https://10.73.1.105:6020/restconf/data/system --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .
curl -X GET https://10.73.1.105:6020/restconf/data/system/config --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
curl -X GET https://10.73.1.105:6020/restconf/data/system --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-system:config".hostname'
```
### Using Python
```
ksator@automation_1:~$ python3
Python 3.6.9 (default, Jan 26 2021, 15:33:00)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> from requests.auth import HTTPBasicAuth
>>> import json
>>> USER = 'arista'
>>> PASS = 'arista'
>>> requests.packages.urllib3.disable_warnings()
>>> headers = {'Content-Type': 'application/yang-data+json', 'Accept': 'application/yang-data+json'}
>>> headers = {'Accept': 'application/yang-data+json'}
>>> api_call = "https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1/state"
>>> result = requests.get(api_call, auth=HTTPBasicAuth(USER, PASS), headers=headers, verify=False)
>>> result.status_code
200
>>> result.ok
True
>>> result.url
'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1/state'
>>> result.content
b'{"openconfig-interfaces:admin-status":"UP","openconfig-interfaces:counters":{"in-broadcast-pkts":"0","in-discards":"0","in-errors":"0","in-fcs-errors":"0","in-multicast-pkts":"972","in-octets":"116602","in-unicast-pkts":"131","out-broadcast-pkts":"1","out-discards":"0","out-errors":"0","out-multicast-pkts":"1761","out-octets":"199997","out-unicast-pkts":"122"},"openconfig-interfaces:description":"restconf_test","openconfig-interfaces:enabled":true,"openconfig-platform-port:hardware-port":"Port1","openconfig-interfaces:ifindex":1,"arista-intf-augments:inactive":false,"openconfig-interfaces:last-change":"1624966430515012864","openconfig-interfaces:loopback-mode":false,"openconfig-interfaces:mtu":0,"openconfig-interfaces:name":"Ethernet1","openconfig-interfaces:oper-status":"UP","openconfig-vlan:tpid":"openconfig-vlan-types:TPID_0X8100","openconfig-interfaces:type":"iana-if-type:ethernetCsmacd"}\n'
>>> result.json()
{'openconfig-interfaces:admin-status': 'UP', 'openconfig-interfaces:counters': {'in-broadcast-pkts': '0', 'in-discards': '0', 'in-errors': '0', 'in-fcs-errors': '0', 'in-multicast-pkts': '972', 'in-octets': '116602', 'in-unicast-pkts': '131', 'out-broadcast-pkts': '1', 'out-discards': '0', 'out-errors': '0', 'out-multicast-pkts': '1761', 'out-octets': '199997', 'out-unicast-pkts': '122'}, 'openconfig-interfaces:description': 'restconf_test', 'openconfig-interfaces:enabled': True, 'openconfig-platform-port:hardware-port': 'Port1', 'openconfig-interfaces:ifindex': 1, 'arista-intf-augments:inactive': False, 'openconfig-interfaces:last-change': '1624966430515012864', 'openconfig-interfaces:loopback-mode': False, 'openconfig-interfaces:mtu': 0, 'openconfig-interfaces:name': 'Ethernet1', 'openconfig-interfaces:oper-status': 'UP', 'openconfig-vlan:tpid': 'openconfig-vlan-types:TPID_0X8100', 'openconfig-interfaces:type': 'iana-if-type:ethernetCsmacd'}
>>> result.json()['openconfig-interfaces:oper-status']
'UP'
>>> result.json()['openconfig-interfaces:counters']['out-octets']
'199997'
>>> exit()
```
```
python3 get.py
{'arista-intf-augments:inactive': False,
 'openconfig-interfaces:admin-status': 'UP',
 'openconfig-interfaces:counters': {'in-broadcast-pkts': '0',
                                    'in-discards': '0',
                                    'in-errors': '0',
                                    'in-fcs-errors': '0',
                                    'in-multicast-pkts': '1762',
                                    'in-octets': '202553',
                                    'in-unicast-pkts': '183',
                                    'out-broadcast-pkts': '1',
                                    'out-discards': '0',
                                    'out-errors': '0',
                                    'out-multicast-pkts': '2552',
                                    'out-octets': '284793',
                                    'out-unicast-pkts': '174'},
 'openconfig-interfaces:description': 'restconf_test',
 'openconfig-interfaces:enabled': True,
 'openconfig-interfaces:ifindex': 1,
 'openconfig-interfaces:last-change': '1624966430515012864',
 'openconfig-interfaces:loopback-mode': False,
 'openconfig-interfaces:mtu': 0,
 'openconfig-interfaces:name': 'Ethernet1',
 'openconfig-interfaces:oper-status': 'UP',
 'openconfig-interfaces:type': 'iana-if-type:ethernetCsmacd',
 'openconfig-platform-port:hardware-port': 'Port1',
 'openconfig-vlan:tpid': 'openconfig-vlan-types:TPID_0X8100'}
```
## HEAD
### Using cURL

```
curl --head 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
HTTP/1.1 200 OK
Content-Type: application/yang.data+json
Date: Sun, 04 Jul 2021 14:20:39 GMT
```
### Using Python

```
$ python3
Python 3.6.9 (default, Jan 26 2021, 15:33:00)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> from requests.auth import HTTPBasicAuth
>>> requests.packages.urllib3.disable_warnings()
>>> USER = 'arista'
>>>
>>> PASS = 'arista'
>>> headers = {'Accept': 'application/yang-data+json'}
>>> api_call = "https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1/state"
>>> result = requests.head(api_call, auth=HTTPBasicAuth(USER, PASS), headers=headers, verify=False)
>>> result.url
'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet1/state'
>>> result.ok
True
>>> result.status_code
200
>>> result.headers
{'Content-Type': 'application/yang.data+json', 'Date': 'Sun, 04 Jul 2021 14:13:00 GMT'}
>>> result.content
b''
>>> result
<Response [200]>
>>> print(result)
<Response [200]>
>>> type(result)
<class 'requests.models.Response'>
>>> exit()
```
## PUT

### Using cURL
#### Interface configuration example

Let's check before the change
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
{"openconfig-interfaces:description":"blabla","openconfig-interfaces:enabled":false,"arista-intf-augments:load-interval":300,"openconfig-interfaces:loopback-mode":false,"openconfig-interfaces:mtu":0,"openconfig-interfaces:name":"Ethernet4","openconfig-vlan:tpid":"openconfig-vlan-types:TPID_0X8100","openconfig-interfaces:type":"iana-if-type:ethernetCsmacd"}
```
Let's use [this file](interface.json)
```
curl -X PUT https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config -H 'Content-Type: application/json' -u arista:arista -d @interface.json  --insecure
{"openconfig-interfaces:description":"restconf_test","openconfig-interfaces:enabled":true,"openconfig-interfaces:name":"Ethernet4"}
```
Let's verify after the change
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
{"openconfig-interfaces:description":"restconf_test","openconfig-interfaces:enabled":true,"arista-intf-augments:load-interval":300,"openconfig-interfaces:loopback-mode":false,"openconfig-interfaces:mtu":0,"openconfig-interfaces:name":"Ethernet4","openconfig-vlan:tpid":"openconfig-vlan-types:TPID_0X8100","openconfig-interfaces:type":"iana-if-type:ethernetCsmacd"}
```
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .
{
  "openconfig-interfaces:description": "restconf_test",
  "openconfig-interfaces:enabled": true,
  "arista-intf-augments:load-interval": 300,
  "openconfig-interfaces:loopback-mode": false,
  "openconfig-interfaces:mtu": 0,
  "openconfig-interfaces:name": "Ethernet4",
  "openconfig-vlan:tpid": "openconfig-vlan-types:TPID_0X8100",
  "openconfig-interfaces:type": "iana-if-type:ethernetCsmacd"
}
```

#### Device hostname example

Let's check before the change
```
curl -X GET https://10.73.1.105:6020/restconf/data/system/config --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
{"openconfig-system:hostname":"DC1-LEAF1A"}
```
```
curl -X GET https://10.73.1.105:6020/restconf/data/system --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-system:config".hostname'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 74748    0 74748    0     0   300k      0 --:--:-- --:--:-- --:--:--  300k
"DC1-LEAF1A"
```
```
curl -X PUT https://10.73.1.105:6020/restconf/data/system/config -H 'Content-Type: application/json' -u arista:arista -d '{"openconfig-system:hostname":"test"}'  --insecure
{"openconfig-system:hostname":"test"}
```
Let's verify after the change
```
curl -X GET https://10.73.1.105:6020/restconf/data/system/config --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
{"openconfig-system:hostname":"test"}
```

## POST
### Using cURL
#### Interface configuration example

Let's check before the change
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:config"'
{
  "description": "",
  "enabled": true,
  "arista-intf-augments:load-interval": 300,
  "loopback-mode": false,
  "mtu": 0,
  "name": "Ethernet4",
  "openconfig-vlan:tpid": "openconfig-vlan-types:TPID_0X8100",
  "type": "iana-if-type:ethernetCsmacd"
}
```
```
curl -X POST https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config -H 'Content-Type: application/json' -u arista:arista -d '{"openconfig-interfaces:description":"restconf_test"}'  --insecure
{"openconfig-interfaces:description":"restconf_test"}
```
```
curl -X POST https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4/config -H 'Content-Type: application/json' -u arista:arista -d '{"openconfig-interfaces:enabled":false}'  --insecure {"openconfig-interfaces:enabled":false}
```
Let's verify after the change
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:config".description'
"restconf_test"
```
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/openconfig-interfaces:interfaces/interface=Ethernet4' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure | jq .'"openconfig-interfaces:config"'
{
  "description": "restconf_test",
  "enabled": false,
  "arista-intf-augments:load-interval": 300,
  "loopback-mode": false,
  "mtu": 0,
  "name": "Ethernet4",
  "openconfig-vlan:tpid": "openconfig-vlan-types:TPID_0X8100",
  "type": "iana-if-type:ethernetCsmacd"
}
```

## DELETE

### Using cURL

Let's check before the change

```
curl -s GET 'https://10.73.1.105:6020/restconf/data/ietf-interfaces:interfaces/interface=Loopback100' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
{"openconfig-interfaces:config":{"description":"test","enabled":true,"arista-intf-augments:load-interval":300,"loopback-mode":true,"name":"Loopback100","openconfig-vlan:tpid":"openconfig-vlan-types:TPID_0X8100","type":"iana-if-type:softwareLoopback"},"openconfig-interfaces:hold-time":{"config":{"down":0,"up":0},"state":{"down":0,"up":0}},"openconfig-interfaces:name":"Loopback100","openconfig-interfaces:state":{"enabled":true,"loopback-mode":false,"openconfig-vlan:tpid":"openconfig-vlan-types:TPID_0X8100"},"openconfig-interfaces:subinterfaces":{"subinterface":[{"config":{"description":"test","enabled":true,"index":0},"index":0,"openconfig-if-ip:ipv4":{"config":{"dhcp-client":false,"enabled":true,"mtu":1500},"state":{"dhcp-client":false,"enabled":true,"mtu":1500}},"openconfig-if-ip:ipv6":{"config":{"dhcp-client":false,"enabled":false,"mtu":1500},"state":{"dhcp-client":false,"enabled":false,"mtu":1500}},"state":{"enabled":true,"index":0}}]}}
```
```
curl -X DELETE https://10.73.1.105:6020/restconf/data/ietf-interfaces:interfaces/interface=Loopback100 -u arista:arista  --insecure
```
Let's verify after the change.
The interface lo100 is not deleted.
This is due to the bug 599524.
This bug is already fixed but it is not included in this EOS release.
```
curl -s GET 'https://10.73.1.105:6020/restconf/data/ietf-interfaces:interfaces/interface=Loopback100' --header 'Accept: application/yang-data+json' -u arista:arista  --insecure
```

### Using Python

```
>>> import requests
>>> from requests.auth import HTTPBasicAuth
>>> import json
>>> USER = 'arista'
>>> PASS = 'arista'
>>> requests.packages.urllib3.disable_warnings()
>>> headers = {'Content-Type': 'application/yang-data+json', 'Accept': 'application/yang-data+json'}
>>> api_call = "https://10.73.1.105:6020/restconf/data/ietf-interfaces:interfaces/interface=Loopback100"
>>> result = requests.get(api_call, auth=HTTPBasicAuth(USER, PASS), headers=headers, verify=False)
>>> result.status_code
200
>>> result.json()
{'openconfig-interfaces:config': {'description': 'test', 'enabled': True, 'arista-intf-augments:load-interval': 300, 'loopback-mode': True, 'name': 'Loopback100', 'openconfig-vlan:tpid': 'openconfig-vlan-types:TPID_0X8100', 'type': 'iana-if-type:softwareLoopback'}, 'openconfig-interfaces:hold-time': {'config': {'down': 0, 'up': 0}, 'state': {'down': 0, 'up': 0}}, 'openconfig-interfaces:name': 'Loopback100', 'openconfig-interfaces:state': {'enabled': True, 'loopback-mode': False, 'openconfig-vlan:tpid': 'openconfig-vlan-types:TPID_0X8100'}, 'openconfig-interfaces:subinterfaces': {'subinterface': [{'config': {'description': 'test', 'enabled': True, 'index': 0}, 'index': 0, 'openconfig-if-ip:ipv4': {'config': {'dhcp-client': False, 'enabled': True, 'mtu': 1500}, 'state': {'dhcp-client': False, 'enabled': True, 'mtu': 1500}}, 'openconfig-if-ip:ipv6': {'config': {'dhcp-client': False, 'enabled': False, 'mtu': 1500}, 'state': {'dhcp-client': False, 'enabled': False, 'mtu': 1500}}, 'state': {'enabled': True, 'index': 0}}]}}
>>> result = requests.delete(api_call, auth=HTTPBasicAuth(USER, PASS), headers=headers, verify=False)
>>> result.status_code
200
>>> result.ok
True
>>>
>>> result = requests.get(api_call, auth=HTTPBasicAuth(USER, PASS), headers=headers, verify=False)
>>> result.json()
```
