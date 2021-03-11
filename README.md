# Proxy integration and issuance of certificates

## PKI setup

You can use the `openssl.conf` configuration file which is provided by default on Linux systems to issue certificates. Above is an example of setting up this file    

Command for working with certificates:    
```bash
	$ openssl command [ command_options ] [ command_arguments ]
````

`genrsa` — generate key    
`req` — register a certificate    
`ca` — sign a certificate    

genrsa:    
	**example:**    
		`openssl genrsa -aes256 -out private.key`    
    
req:    
	**commans:**    
		`-x509` — root certificate    
		`-nodes` — no encryption    
		`-days` — number of days
		`-key` — key    
		`-out` — save cert    
	**example:**    
		`openssl req -new -key private.key -out cert.csr`    
    
ca:    
	**commands:**    
		`-extensions` — using an extension    
		`-in` — which certificate to sign    
		`-out` — output file   


For work openssl, you need create files:    
```
touch index.txt    
echo 01 > serial    
mkdir certs crl requests newcerts    
```
    
All extension:    
v3_ca    
v3_intermediate_ca    
server_cert    
usr_cert    
    
## Proxy integration

In the upper part of the configuration file, write the settings (see the documentation of your DLP)    
    
Also write the lines in the configuration file:    
    
```
http_port 0.0.0.0:3128 intercept ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB  cert=/etc/squid/public.pem key=/etc/squid/private.pem    
https_port 0.0.0.0:3129 intercept ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB  cert=/etc/squid/public.pem key=/etc/squid/private.pem    
    
acl step1 at_step SslBump1    
acl step2 at_step SslBump2    
acl step3 at_step SslBump3    
    
ssl_bump peek step1 all    
ssl_bump bump step2 all    
ssl_bump bump step3 all    
    
sslcrtd_program /usr/lib/squid/ssl_crtd -s /var/lib/ssl_db -M 16MB    
```
    
Also set up a firewall on a machine with a proxy installed    
To view the current firewall settings, enter the command    
`iptables-save`    
    
Next, you should open the ports. For example, 2 subnets are taken (the first (183) is old, the second (130) is new)    
```bash
# iptables -t nat -D POSTROUTING -s 192.168.183.0/24 -j MASQUERADE

# iptables -t nat -D PREROUTING -s 192.168.183.0/24 -p tcp -m multiport --dports 80,8080 -j REDIRECT --to-ports 3128

# iptables -t nat -A PREROUTING -s 192.168.130.0/24 -p tcp -m multiport --dports 80,8080 -j REDIRECT --to-ports 3128

# iptables -t nat -D PREROUTING -s 192.168.183.0/24 -p tcp -m multiport --dports 443 -j REDIRECT --to-ports 3129

# iptables -t nat -A PREROUTING -s 192.168.130.0/24 -p tcp -m multiport --dports 443 -j REDIRECT --to-ports 3129

# iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 53 -j ACCEPT

# iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 53 -j ACCEPT

# iptables -t filter -D INPUT -s 192.168.183.0/24 -p udp -m multiport --ports 53 -j ACCEPT

# iptables -t filter -A INPUT -s 192.168.130.0/24 -p udp -m multiport --ports 53 -j ACCEPT

# iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 3128 -j ACCEPT

# iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 3128 -j ACCEPT

# iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 3129 -j ACCEPT

# iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 3129 -j ACCEPT

# iptables -t filter -D FORWARD -s 192.168.183.0/24 -p tcp -m multiport --ports 110,5190,25,21 -j ACCEPT

# iptables -t filter -A FORWARD -s 192.168.130.0/24 -p tcp -m multiport --ports 110,5190,25,21 -j ACCEPT 

```


Save changes:
`netfilter-persistent save`


After the certificate for the proxy has been issued, place the keys (public and private) in the directory with proxy settings    
    
Last step: :white_check_mark: Configure icap on your DLP system