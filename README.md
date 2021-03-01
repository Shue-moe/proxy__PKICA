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
	**example:**    
		`openssl req -new -key private.key -out cert.csr`    
    
ca:    
	**commands:**    
		`-extensions` — использование расширения    
    
## Интеграция proxy
    
Прописать в конф файл строки:    
    
http_port 192.168.99.100:3128 intercept ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB  cert=/etc/squid/public.pem key=/etc/squid/private.pem    
https_port 192.168.99.100:3129 intercept ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB  cert=/etc/squid/public.pem key=/etc/squid/private.pem    
    
acl step1 at_step SslBump1    
acl step2 at_step SslBump2    
acl step3 at_step SslBump3    
    
ssl_bump peek step1 all    
ssl_bump bump step2 all    
ssl_bump bump step3 all    
    
sslcrtd_program /usr/lib/squid/ssl_crtd -s /var/lib/ssl_db -M 16MB    
    
    
После того как был выписан сертификат для proxy поместить его в дирректорию с настройками proxy    