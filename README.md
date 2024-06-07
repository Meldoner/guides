# СОЗДАНИЕ ОБХОДА БЛОКИРОВОК, ИСПОЛЬЗУЯ VLESS
1. Покупка VPS (самого дешевого) на ОС  Ubuntu/Debian.
2. Подготовка:
	1. Скачиваем [Termius](https://termius.com/) .
	2. Подключаемся к своему серверу и обновляемся:
	    	```sh
		apt update && apt full-upgrade -y
		```
		Далее делаем перезапуск: `reboot`
	3. Устанавливаем Docker и необходимые пакеты:
		```sh
		apt install docker.io docker-compose git curl bash openssl nano -y
		```
	4. Устанавливаем панель [3x-ui](https://github.com/MHSanaei/3x-ui)  и запускаем её:
		```sh
		git clone https://github.com/MHSanaei/3x-ui.git
		cd 3x-ui
		docker compose up -d	
		```
	5. Устанавливаем самоподписанный TLS сертификат:
		```sh
		openssl req -x509 -newkey rsa:4096 -nodes -sha256 -keyout private.key -out public.key -days 3650 -subj "/CN=APP"
		docker cp private.key 3x-ui:private.key
		docker cp public.key 3x-ui:public.key
		```
3. Первичная настройка 3x-ui:
	1. Переходим по адресу панели: `http:/ВашIPСервера:2053/`
		**Логин:** `admin` , **пароль:** `admin`
	2. Настраиваем вход:
		В разделе **Panel Settings** заполните 4 поля:
		
		**- Panel Port** (любое случайное число от 1000 до 65535, кроме 40000.
		**- Panel Certificate Public Key Path** : `/public.key`  
		**- Panel Certificate Private Key Path** : `/private.key`  
		**- Panel URL Root Path**: секретная строка для доступа к панели, которая начинается и заканчивается "/". Придумываем **свою**, например: `/secretpanel/`
		
		Сохраняем (**Save**) и перезагружаем панель (**Restart Panel**)
		Теперь наша панель находится по адресу: `https://ВашIpСервера:ВашПорт/secretpanel/`
		
	3. Меняем логин и пароль панели:
		В разделе **Panel Settings**  -> **Security Settings** укажите старые (admin/admin) и придумайте новые логин и пароль и сохраните (**Confirm**).
		
		Войдите в панель под новыми данными.
4. Настройка [Xray](https://tldrify.com/1dsz):
	1. Базовый шаблон:
		В панели перейдите в раздел **Xray Settings** и включите две опции:  
		**- IPv4 Configs** -> **Use IPv4 for Google**  
		**- WARP Configs** -> **Route OpenAI (ChatGPT) through WARP**.  
		  
		Сохранитесь (**Save Settings**) и перезагрузитесь (**Restart Xray**)
	2. Пустите весь российский трафик на сервере через **[WARP](https://one.one.one.one/ru-RU/)**:
		Перейдите в раздел **Xray Settings**->**Routing Rules**, найдите строку где написано "`geosite:openai`" и отредактируйте её, чтобы получилось: `geosite:category-gov-ru,regexp:.*\.ru$,geosite:openai`  
  
		Наконец, добавьте новое правило через кнопку Add Rule (IP: `geoip:ru`, Outbound tag: `WARP`).
		![[chrome_zUPXaQU9dJ.jpg]]
		Сохранитесь (**Save Settings**), затем перезагрузитесь (**Restart Xray**)
	3. Находим сайт для маскировки: 
		Лучше найти сайт-донор либо одного провайдера, либо из той же страны.
		[Инструкция](https://tldrify.com/1dsy) (от этого будет зависеть скорость загрузки страниц)
	4. Настраиваем [VLESS](https://tldrify.com/1dt1):
		В разделе (**Inbound**) - (**Add inbound**) надо заполнить следующие поля: 
		  
		**- Remark** - Название соединения в панели.
		
		**- Listening IP** - указываем ваш **IP**.
		  
		**- Port** - строго **443**, чтобы маскироваться под обычный https-сайт.  
		
		**-** **Email** - Название созданного клиента.
		
		**-** Только после выбора **Security: Reality** появится возможность выбора **Flow: xtls-rprx-vision**.
		  
		**- uTLS** - именно Chrome, чтобы маскироваться под самый популярный браузер.  
		  
		**- Dest, Server Names** - указать сайт-донор (хотя можно оставить `yahoo.com`).  
		  
		**- (Get new cert)**  - нажимаем, генерирует ключи.  
		  
		**- (Create)** завершает создание.
		![[chrome_g23T1QYnR2.jpg]]
	5. Получите ключ VLESS:
		В разделе Inbounds нажимаем на (**три точки**) -> **Экспорт ключей (Export Links)** и система даст скопировать строку-ключ вида:
		`vless://1a593243-6d2a-4543-8aa9-9ef42b2bbba6@ServerIp:443?type=tcp&security=reality&pbk=2Piht6cC_B5B3wYFZyU1Gr_iMfs3LsWghrXZNFjhL18&fp=chrome&sni=yahoo.com&sid=fe60e30f&spx=%2F&flow=xtls-rprx-vision#VLESS1-myname`
	

5. Настройка клиента (на компьютерах и телефонах)
		**Windows**: 
			[Hiddify](https://github.com/hiddify/hiddify-next/releases/) :
				- В настройках установить Регион: **RU**
				- Скопировать в буфер и вставить в приложение **строку‑ключ VLESS** через кнопку (**+**)
				- Нажимаем на огромную круглую **кнопку**
				- Приложение может работать в двух режимах:
					- Системный прокси (по умолчанию)
					- TUN-режим
					- Прокси
			[Nekoray](https://github.com/Matsuridayo/nekoray/releases) 
		**Android:**
			[Hiddify](https://play.google.com/store/apps/details?id=app.hiddify.com) ([Github](https://github.com/hiddify/hiddify-next/releases/))
			[V2rayNG](https://play.google.com/store/apps/details?id=com.v2ray.ang)
		**IOS:** 
			[Streisand](https://apps.apple.com/us/app/streisand/id6450534064):
				- Скопировать в буфер и вставить в приложение строку-ключ vless через кнопку (+)
				- Настроить прямое соединение с Рунетом. Тут несколько шагов.   
					1. Сначала - скопировать эту длинную строку с настройками в буфер.
					2. Затем  вставить её в приложение через «+» справа‑вверху.
					3. Зайти в «Роутинг», и там (поставить галочку), (нажать на «включить») и вернуться назад (стрелка слева)![[IMG_2521 1.png]]
			[Shadowrocket](https://apps.apple.com/us/app/shadowrocket/id932747118)(**Платный**)
		**macOS:**
			[Hiddify](https://github.com/hiddify/hiddify-next/releases/)
			[Streisand](https://apps.apple.com/us/app/streisand/id6450534064)
Для создания темы использовались ресурсы:
- [Личный прокси для чайников: универсальный обход цензуры с помощью VPS, 3X-UI, Reality/CDN и Warp](https://habr.com/ru/articles/785186/)
- [Современные технологии обхода блокировок: V2Ray, XRay, XTLS, Hysteria, Cloak и все-все-все](https://habr.com/ru/articles/727868/)
