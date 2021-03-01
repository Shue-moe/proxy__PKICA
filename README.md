# Интеграция прокси и выдача сертификатов

## Настройка PKI

Для выдачи сертификатов можно использовать конфигрурационный файл `openssl.conf` который находится в системах Linux по умолчанию. Выше дан пример настройки этого файла.

Команда для работы с сертификатами:
$ openssl command [ command_options ] [ command_arguments ]

`genrsa` -- сгенерировать ключ    
`req` -- зарегестрировать сертификат    
`ca` -- подписать сертификат    

genrsa:
	**example:**
		`openssl genrsa -aes256 -out private.key`

req:
	**example:**
		`openssl req -new -key private.key -out cert.csr`

ca:
	**commands:**
		`-extensions` -- использование расширения
