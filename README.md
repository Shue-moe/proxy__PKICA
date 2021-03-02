# Интеграция прокси и выдача сертификатов

## Настройка PKI

Для выдачи сертификатов можно использовать конфигрурационный файл `openssl.conf` который находится в системах Linux по умолчанию. Выше дан пример настройки этого файла.

Команда для работы с сертификатами:    
```bash
	$ openssl command [ command_options ] [ command_arguments ]
````

`genrsa` — сгенерировать ключ    
`req` — зарегестрировать сертификат    
`ca` — подписать сертификат    

genrsa:    
	**example:**    
		`openssl genrsa -aes256 -out private.key`    
    
req:    
	**commans:**    
		`-x509` — корневой сертификат    
		`-nodes` — без шифрования    
		`-days` — количество дней    
	**example:**    
		`openssl req -new -key private.key -out cert.csr`    
    
ca:    
	**commands:**    
		`-extensions` — использование расширения    
		`-in` — какой сертификат подписать
		`-out` — выходной файл
    
## Интеграция proxy

В верхнюю часть конфигурационного файла прописать настройки (смотрите в докумментацию вашего DLP)    
    
Так же прописать в конфигурационный файл строки:    
    
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
    
Так же настроить firewall на машине с установленным proxy    
Для просмотра нынешних настроек firewall введите команду    
`iptables-save`    
    
Дальше следует открыть порты. Для примера взяты 2 подсети (первая (183) — старая, вторая (130) — новая)    
```bash
# iptables -t nat -D POSTROUTING -s 192.168.183.0/24 -j MASQUERADE

# iptables -t nat -D PREROUTING -s 192.168.183.0/24 -p tcp -m multiport --dports 80,8080 -j REDIRECT --to-ports 3128

# iptables -t nat -A PREROUTING -s 192.168.130.0/24 -p tcp -m multiport --dports 80,8080 -j REDIRECT --to-ports 3128

#  iptables -t nat -D PREROUTING -s 192.168.183.0/24 -p tcp -m multiport --dports 443 -j REDIRECT --to-ports 3129

#  iptables -t nat -A PREROUTING -s 192.168.130.0/24 -p tcp -m multiport --dports 443 -j REDIRECT --to-ports 3129

#  iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 53 -j ACCEPT

#  iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 53 -j ACCEPT

#  iptables -t filter -D INPUT -s 192.168.183.0/24 -p udp -m multiport --ports 53 -j ACCEPT

#  iptables -t filter -A INPUT -s 192.168.130.0/24 -p udp -m multiport --ports 53 -j ACCEPT

#  iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 3128 -j ACCEPT

#  iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 3128 -j ACCEPT

#  iptables -t filter -D INPUT -s 192.168.183.0/24 -p tcp -m multiport --ports 3129 -j ACCEPT

#  iptables -t filter -A INPUT -s 192.168.130.0/24 -p tcp -m multiport --ports 3129 -j ACCEPT

# iptables -t filter -D FORWARD -s 192.168.183.0/24 -p tcp -m multiport --ports 110,5190,25,21 -j ACCEPT

#  iptables -t filter -A FORWARD -s 192.168.130.0/24 -p tcp -m multiport --ports 110,5190,25,21 -j ACCEPT 

```

Сохраните изменения:
`netfilter-persistent save`


После того как был выписан сертификат для proxy поместить ключи (public и private) в дирректорию с настройками proxy    