<h1>Описание</h1>
<hr />
<p>MTA (Mail Transfer Agent?) - сервер по пересылке электронных  сообщений. Сам по себе postfix умеет принимать, пересылать и отправлять  электронные письма, но не имеет возможностей авторизации, передачи почты  клиентским почтовым прогрммам. Обладает интерфейсом взаимодействия с  sendmail, что позволяет использовать его без изменений многих настроек  систем, работавших до того с sendmail.</p>
<p>Так же для обеспечения безопасности работает через chroot в /var/spool/postfix</p>
<p>Для полноценной работы необходимо установить:</p>
<table class="moduletable" style="width: 100%;" border="1" cellspacing="0" cellpadding="3">
<thead> 
<tr>
<td style="width: 20%;">Название</td>
<td>Описание</td>
</tr>
</thead> 
<tbody>
<tr>
<td>postfix</td>
<td>Собственно сама почтовая программа<br /></td>
</tr>
<tr>
<td>
<p>dk-filter</p>
</td>
<td>
<p>Добавляет DomainKey подпись (в помощь антиспаму)</p>
</td>
</tr>
<tr>
<td>
<p>dkim-filter</p>
</td>
<td>
<p>Добавляет DKIM подпись (продвинутый DomainKey)</p>
</td>
</tr>
<tr>
<td>
<p> </p>
</td>
<td>
<p> </p>
</td>
</tr>
<tr>
<td>
<p> </p>
</td>
<td>
<p> </p>
</td>
</tr>
<tr>
<td>
<p> </p>
</td>
<td>
<p> </p>
</td>
</tr>
</tbody>
</table>
<h1>Установка</h1>
<hr />
<p>Для установки самого postfix`a достаточно команды</p>
<pre>apt-get install postfix</pre>
<p>В конце установки будет задан вопрос как называется почтовый сервер  (напримерр mail.apmem.org) и какого типа сервер настраивать по умолчанию  (например что-то там для интернета).</p>
<p> </p>
<h1>Настройка</h1>
<hr />
<p>Основной конфигурационный файл называется /etc/postfix/main.cf</p>
<p>При первоначальной установке создается сертификат для работы с  клиентскими программами в папке /etc/ssl/private и /etc/ssl/certs. При  желании новый сертификат можно создать при помощи openssl (см. <a href="index.php/administrating/12-certificates#openssl">openssl</a>).</p>
<p>Для того, чтобы thunderbird не ругался страшным словом  SEC_ERROR_INADEQUATE_CERT_TYPE необходимо чтобы openssl extension  содержал</p>
<pre>[ v3_mail ]<br />extendedKeyUsage = 1.3.6.1.5.5.7.3.4<br />subjectKeyIdentifier=hash<br />basicConstraints = CA:false<br />nsCertType = server, email<br />authorityKeyIdentifier=keyid,issuer</pre>
<p>После чего прописать их пути или подменть по указаным в main.cf файле  (я сделал линки на файлы в папке postfix чтобы проще было подменять  пока тестирую). Основные настройки, которые создаются в процессе  установки:</p>
<pre>#Base configuration parameters<br />#This name will be displayed in HELO string<br />mail_name = qmail<br /># Something like computer name if there is multiple mail servers<br />myorigin = /etc/mailname<br /># Domain name that will be worked with postfix<br />mydomain = apmem.org<br /># Host name that will be worked with postfix<br />myhostname = mail.apmem.org<br /># This destinations will be the final destinations for this server<br />#if email sent to one of this names, this server will not send them anywhere next<br />mydestination = $myorigin, $mydomain, $myhostname, localhost.localdomain, localhost<br /># Only this networks can send email THROUGH this server.<br />#Other networks can send emails only TO this server<br />mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.0.0/24<br /># HELO banner text (it is required to sent host name<br />smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)<br />biff = no<br /><br /># appending .domain is the MUA\'s job.<br />append_dot_mydomain = no<br /><br />readme_directory = no<br /><br /># TLS parameters<br />smtpd_tls_cert_file=/etc/postfix/certs/emailcert.pem<br />smtpd_tls_key_file=/etc/postfix/certs/emailkey.pem<br />smtpd_tls_CAfile=/etc/postfix/certs/cacert.pem<br />smtpd_use_tls=yes<br />smtp_use_tls=yes<br />smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache<br />smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache<br /><br />#Domainkey parameters<br />milter_default_action = accept<br />milter_protocol = 2<br />smtpd_milters = inet:localhost:8891,inet:localhost:8892<br />non_smtpd_milters = inet:localhost:8891,inet:localhost:8892<br /><br />#aliases, that will be used to resolve user names<br />alias_maps = hash:/etc/aliases<br />alias_database = hash:/etc/aliases<br />mailbox_command = procmail -a "$EXTENSION"<br />mailbox_size_limit = 0<br />recipient_delimiter = +<br />#interfaces, that will be used to send-receive emails<br />inet_interfaces = all</pre>
<p>Все эти и другие основные настройки можно посмотреть в файлах  /usr/share/postfix/main.cf.dist и /usr/share/postfix/main.cf.tls.</p>
<p>Кроме этого, для того, чтобы тот же gmail не считал письма спамерскими необходимо, чтобы было выполнено  3 условия:</p>
<ul>
<li>Существовала MX запись вида "@apmem.org::mail.apmem.org" -  специальная запись для определения адреса, ответственного за прием email  сообщений (см <a href="index.php/administrating/1-djbdns">DjbDNS</a>)</li>
<li>Существовала SPF запись вида "\'apmem.org:v=spf1 a mx ~all:3600" -  текстовая запись, показывающая адреса и имена тех, кто может рассылать   письма от вашего имени (чтобы случайно не получать спам от себя же :).  (см <a href="index.php/administrating/1-djbdns">DjbDNS</a>)</li>
<li>DomainKeys и/или DKIM для подписи сообщений (см <a href="#DomainKeys">DomainKey</a>)</li>
</ul>
<p> </p>
<h1><a name="DomainKeys"></a>DomainKeys</h1>
<hr />
<p>DomainKeys (или DKIM - DomainKeys Identified Mail) предназначены для  подписи сообщений, чтобы получатель точно знал от кого пришло сообщение.  Для каждого случая используется своя программа:</p>
<p>Для DKIM - dkim-filter</p>
<p>Для DomainKey - dk-filter</p>
<p>У каждой свои конфигурационные файлы, но в целом они друг на друга  похожи и смысл у них одинаковый - указать чем подписывать и на каком  порту висеть, чтобы другие программы (в нашем случае postfix) могли с  ними общаться.</p>
<p>Для начала необходимо сгенерировать открытый и закрытый ключи (описание см. <a href="index.php/administrating/12-certificates#openssl">openssl</a>):</p>
<pre>openssl genrsa -out ca.pvk<br />openssl rsa -in ./server.pvk -pubout -outform pem -out ./server.pub</pre>
<p>После этого открытый ключ (результат второй строки необходимо  положить в открытый доступ - а именно в TXT запись домена, за который  отвечает настраиваемый сервер (см <a href="index.php/administrating/1-djbdns#TinyDNS">TinyDNS</a>).</p>
<pre>\'mail._domainkey.apmem.org:k=rsa; t=y; p=MFwwDQY...AAQ==;:60<br />\'_domainkey.apmem.org:t=y; o=-; r=<br /> <a href="mailto:postmaster@apmem.org">postmaster@apmem.org</a><span style="display: none;">This e-mail address is being protected from spambots. You need JavaScript enabled to view it
 </span>;:60<br /></pre>
<p>В данном случае</p>
<p>- слово mail является так называемым селектором (хз что  это значит,  просто какой-то фильтр похоже), который будет использован в  дальнейшем  при конфигурации</p>
<p>- слово _domainkey является ключевым и всегда должен выглядеть именно так. Строка без селектора mail является общедоменной</p>
<p>- параметр k=rsa означает тип подписи, на сколько я понял других и не бывает</p>
<p>- параметр t=y означает, что данная конфигурационная строка для  domainKey является тестовой (некоторыми игнорируется, некоторые в  зависимости от параметра могут менять поведения при отсутствии подписи и  т.д.)</p>
<p>- параметр p=... открытый ключ, которым производится проверка письма.</p>
<p>- параметр o=- означает, что все исходящие письма должны быть  подписаны, если письмо не подписано, получатель может отказать в приеме.  Чтобы указать, что не все письма с этого адреса могут быть подписаны  используется o=~</p>
<p>- параметр r=mail@name адрес электронной почты, на котую должны  посылаться сообщения о неправильно подписаных письмах, отказах при тесте  и т.д. большинство этот параметр игнорирует.</p>
<p>После настройки домена можно переходить к настройке самих сервисов,  отвечающих за подписку. Установка их запускается как обычно из  репозитория:</p>
<pre>apt-get install dk-filter<br />apt-get install dkim-filter</pre>
<p>Конфигурационный файл dkim-filter находится по адресу /etc/dkim-filter.conf</p>
<pre># Log to syslog<br />Syslog                  yes<br /># Required to use local socket with MTAs that access the socket as a non-<br /># privileged user (e.g. Postfix)<br />UMask                   002<br />Socket                  inet:8891@localhost<br /><br /># Sign for apmem.org with key in /etc/ssl/mycerts/private/emaildomainkey.pem using<br /># selector \'mail\' (e.g. mail._domainkey.apmem.org)<br />Domain                  apmem.org<br />KeyFile                 /etc/ssl/mycerts/private/emaildomainkey.pem<br />Selector                mail<br /><br /># Common settings. See dkim-filter.conf(5) for more information.<br />AutoRestart             no<br />Background              yes<br />Canonicalization        simple<br />DNSTimeout              5<br />Mode                    sv<br />SignatureAlgorithm      rsa-sha256<br />SubDomains              no<br />ASPDiscard              no<br />#Version                rfc4871<br />X-Header                no<br /><br />#dkim-filter will sign, but not verify mails, sent from this networks<br />InternalHosts           /etc/dkim-hosts.conf</pre>
<p>Здесь указывается местоположение закрытого ключа, имя домена и  селектор (должен совпадать с записью в днс), список сетей, из которых  письма надо не проверять, а подписывать, а так же сокет, через который  postfix будет общаться с сервисом (в postfix/main.cf в разделе  smtpd_milters указывается сервисы, через которые должно пройти письмо)</p>
<p>Конфигурационный файл dk-filter находится в /etc/default/dk-filter. он так же содержит данные о ключе, селекторе и сокете:</p>
<pre># Sane defaults: log to syslog<br />DAEMON_OPTS="-l"<br /># Domain, key and selector options<br />DAEMON_OPTS="$DAEMON_OPTS -d apmem.org -s /etc/ssl/mycerts/private/emaildomainkey.pem -S mail"<br /># Socket to communicate with postfix<br />SOCKET="inet:8892@localhost"</pre>
<p>После всего этого перезапускаем сервисы, и пытаемся отправить письмо.  В результате в заголовках письма должны присутствовать строки  "DKIM-Signature:" и "DomainKey-Signature: "</p>
<p> </p>
<h1>Возможные проблемы</h1>
<h1>
<hr />
</h1>
<h2>Почта не ходит на gmail адреса.</h2>
<p>Возвращается ответ</p>
<pre>
 The IP you\'re using to send
    mail is not authorized to 550-5.7.1 send email directly to our servers.
    Please use the SMTP relay at your 550-5.7.1 service provider instead. Learn
    more at 550 5.7.1 http://support.google.com/mail/bin/answer.py?answer=10336
</pre>
<p>Если все настройки верны, и указанная ссылка не приводит к внятным объяснениям, скорее всего адрес, с которого производится рассылка попал в <a class="bbc_link" href="http://www.spamhaus.org/pbl" target="_blank" style="color: #007ea1; font-family: Verdana, Arial, Helvetica, sans-serif; line-height: 17.328125px; background-color: #f2f2f2;">http://www.spamhaus.org/pbl</a> (чем gmail очень даже пользуется).</p>
<p> </p>
<h1>Полезные ссылки</h1>
<hr />
<p><a href="http://old.openspf.org/wizard.html">http://old.openspf.org/wizard.html</a> - Создание правильной spf записи</p>
<p><a href="mailto:check-auth@verifier.port25.com">check-auth@verifier.port25.com</a><span style="display: none;">This e-mail address is being protected from spambots. You need JavaScript enabled to view it </span> - Адрес для проверки настроек почтового сервера</p>
<p><a href="http://domainkeys.sourceforge.net/policycheck.html">http://domainkeys.sourceforge.net/policycheck.html</a> - Проверка настройки DomainKyes.</p>
<p><a href="http://domainkeys.sourceforge.net/selectorcheck.html">http://domainkeys.sourceforge.net/selectorcheck.html</a> - Проверка настройки DomainKyes селектора.</p>
