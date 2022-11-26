<h1>Описание</h1>
<hr />
<p>djbdns представляет из себя набор простых программ для создания, обслуживания и диагностики серверов доменных имен (DNS).</p>
<p>Программы, предназначенные для создания DNS серверов. В отличие от Bind серверы разделены на 2 вида по выполняемым обязанностям:</p>
<table class="moduletable" style="width: 100%;" border="1" cellspacing="0" cellpadding="3">
<thead>
<td style="width: 20%;">Название</td>
<td>Описание</td>
</thead> 
<tbody>
<tr>
<td><a href="#TinyDNS">TinyDNS</a></td>
<td>полномочный сервер доменных имен</td>
</tr>
<tr>
<td><a href="#DNSCache">DNScache</a></td>
<td>кэширующий сервер</td>
</tr>
<tr>
<td><a href="#axfrDNS">axfrDNS</a></td>
<td>Сервер синхронизации</td>
</tr>
</tbody>
</table>
<p> </p>
<p> </p>
<p>Для проверки конфигураций серверов и сети в целом существует несколько утилит, каждая из которых выполняет определенную задачу:</p>
<table class="moduletable" style="width: 100%;" border="1" cellspacing="0" cellpadding="3">
<thead>
<td style="width: 20%;">Команда</td>
<td>Описание</td>
</thead> 
<tbody>
<tr>
<td>dnsip FQDN</td>
<td>Вывод ip адреса для указанного доменного имени<br /></td>
</tr>
<tr>
<td>dnsipq FQDN<br /></td>
<td>Вывод FQDN и ip адреса для указанного доменного имени</td>
</tr>
<tr>
<td>dnsname ipaddr</td>
<td>Вывод FQDN по ip адресу<br /></td>
</tr>
<tr>
<td>dnsmx FQDN</td>
<td>Вывод приоритета и FQDN MX записей для указанного доменного имени.  Если таких записей не существует - будет выведена строка с приоритетом 0  и FQDN самого домена<br /></td>
</tr>
<tr>
<td>dnstxt FQDN</td>
<td>Вывод TXT записей для указанного доменного имени<br /></td>
</tr>
<tr>
<td>dnsq type FQDN server</td>
<td>Посылает 1 итеративный запрос указанного типа на DNS сервер server о  указанном доменном имени FQDN. В результате работы выдаются все  обнаруженые записи указанного типа для FQDN. Подходит для локального  тестирования TinyDNS сервера.<br /></td>
</tr>
<tr>
<td>dnsqr type FQDN</td>
<td>Посылает рекурсивные запрос для поиска указанного FQDN. В результате  работы выдаются все обнаруженые записи указанного типа для FQDN. Если  необходимо переопределить кэширующий сервер, записаный в  /etc/resolv.conf, можно задать переменную DNSCACHEIP. Подходит для  тестирования DNSCache<br /></td>
</tr>
<tr>
<td>dnstrace type FQDN server</td>
<td>Прослеживает полный путь от server до FQDN. server lолжен быть корневым сервером.<br /></td>
</tr>
</tbody>
</table>
<p>Так же для проверки настроек dns сервера можно использовать вниешние валидаторы (<a href="http://dnscheck.iis.se/">http://dnscheck.iis.se/</a>, ...)</p>
<p>Где type - одно из следующих значений<br />"ptr" - обратная зона, <br />"a" - прямое преобразование, <br />"txt" ,  <br />"mx" - почтовые записи, <br />"ns" - именя ответственных DNS серверов, <br />"soa", <br />"any" - все вышеперечисленные.</p>
<p> </p>
<h1>Установка</h1>
<hr />
<p>Первоначальная установка tinydns и dnscache одинакова и тербует  установленного daemontools на компьютере. Отличный вариант для получения  самого пакета и всех его зависимостей - "apt-get install djbdns".  Первоначальная конфигурация осуществляется командой:</p>
<pre>tinydns-conf tinydns logdns /var/lib/djbdns/tinydns 91.146.57.100<br />axfrdns-conf axfrdns logdns /var/lib/djbdns/axfrdns /var/lib/djbdns/tinydns 91.146.57.100<br />dnscache-conf cachedns logdns /var/lib/djbdns/cachedns 91.146.57.100</pre>
<p>где<br /> 1 параметр - имя пользователя, из под которого будет запускаться сервис (пользователь должен существовать)<br /> 2 параметр - имя пользователя, из под которого будут писаться логи (пользователь должен существовать)<br /> 3 параметр - путь до папки, куда надо сконфигурировать сервис<br /> 4 параметр - ip адрес, который будет слушать сервис</p>
<p>Для создания пользователя используется команда:</p>
<pre>adduser --system axfrdns</pre>
<p>После того, как сервис будет сконфигурирован, необходимо создать  ссылку на каталог, используемый daemontools`ом для запуска сервисов</p>
<pre>ln -s /var/lib/djbdns/tinydns /etc/service/tinydns<br />ln -s /var/lib/djbdns/axfrdns /etc/service/axfrdns<br />ln -s /var/lib/djbdns/cachedns /etc/service/cachedns</pre>
<p>и подождать несколько секунд, после чего сервис будет запущен и готов к работе. Описание работы daemontools - <a href="http://cr.yp.to/daemontools.html">http://cr.yp.to/daemontools.html</a></p>
<p> </p>
<h1><a name="TinyDNS"></a>TinyDNS</h1>
<h1>
<hr />
</h1>
<p>Конфигурирование осуществляется в файле tinydns/root/data. Простейшая  конфигураци должна включать в себя объявление серверов, ответственных  за домен и описание их ip адресов. Для того, чтобы домен был признан  валидным в большинстве систем он должен иметь как минимум 2 ns сервера с  разными ip адресами, иметь правильную запись обратной зоны и как  минимум одну запись MX.</p>
<p>В TinyDNS каждый тип записи начинается с определенного символа. ttl,  timestamp и lo, присутствующие во всех командах, указывать не  обязательно. Нижеследующая табличка сперта с man tinydns-conf</p>
<table class="moduletable" style="width: 100%;" border="1" cellspacing="0" cellpadding="3">
<thead>
<td>Команда</td>
<td>Тип</td>
<td>Структура</td>
<td>Описание</td>
</thead> 
<tbody>
<tr>
<td>%</td>
<td>-</td>
<td>
<p>lo: ipprefix</p>
</td>
<td>
<p>Определение адресов, принадлежащих определенному местоположению.  Каждый клиент может относиться только к одному местоположению. Например:</p>
<pre># клиентам с ip 192.168.0.0\\16<br />%in:192.168<br /># example.apmem.org резолвится в 192.168.1.2<br />+example.apmem.org:192.168.1.2:::in<br /># остальным<br />%ex<br /># в 91.146.57.100<br />+example.apmem.org:91.146.57.100:::ex</pre>
</td>
</tr>
<tr>
<td>.<br /></td>
<td>A+NS+SOA | NS+SOA<br /></td>
<td>
<p>fqdn: ip: x: ttl: timestamp: lo</p>
</td>
<td>
<p>Создает A (если ip указан, то указаный ns будет резолвиться на него),  NS и SOA запись. Если x содержит точку, то он будет назначен  неизменным, если же нет, то в качестве имени сервера будет назначен  x.ns.zone. Например:</p>
<pre># создаст A, NS и SOA запись a.ns.apmem.org,<br />.apmem.org:91.146.57.100:a<br /># создаст  A, NS и SOA запись dns2.apmem.org<br />.apmem.org:91.146.57.100:dns2.apmem.org<br /># создаст NS и SOA запись a.ns.apmem.org<br />.apmem.org::a.ns.apmem.org<br /></pre>
</td>
</tr>
<tr>
<td>&amp;</td>
<td>NS | A+NS</td>
<td>fqdn: ip: name: ttl: timestamp: lo</td>
<td>
<p>Работает так же как . только не создает SOA запись.</p>
</td>
</tr>
<tr>
<td>=</td>
<td>A+PTR</td>
<td>
<p>fqdn: ip: ttl: timestamp: lo</p>
</td>
<td>
<p>Создает A и PTR записи для указанного fqdn имени. PTR запись является  обратной записью ip адреса с приставкой in-addr.arpa. Например</p>
<pre># создать A запись и PTR запись вида 100.57.146.91.in-addr.arpa.<br />=example.ApmeM.org:91.146.57.100</pre>
</td>
</tr>
<tr>
<td>+</td>
<td>A<br /></td>
<td>
<p>fqdn: ip: ttl: timestamp: lo</p>
</td>
<td>
<p>Создает псевдоним fqdn для указанного ip адреса. Работает так же как = только не создает PTR записи</p>
</td>
</tr>
<tr>
<td>@<br /></td>
<td>MX | A+MX</td>
<td>
<p>fqdn: ip: x: dist: ttl: timestamp: lo</p>
</td>
<td>
<p>Создает A (если ip указан, то указаный ns будет резолвиться на него) и  MX записи. Если x содержит точку, то он будет назначен неизменным, если  же нет, то в  качестве имени сервера будет назначен x.mx.zone. dist -  расстояние до сервера (обратно приоритету) по умолчанию 0. Например:</p>
<p># создает A запись для mail.apmem.org<br /># и MX запись для apmem.org с расстоянием 0<br />@apmem.org:91.146.57.100:mail.apmem.org</p>
</td>
</tr>
<tr>
<td>#</td>
<td>-</td>
<td>
<p>*</p>
</td>
<td>Комментарий. Можно писать многабуков. Любых.<br /></td>
</tr>
<tr>
<td>`</td>
<td>TXT</td>
<td>
<p>fqdn: s: ttl: timestamp: lo</p>
</td>
<td>Создает TXT запись с текстом s.<br /></td>
</tr>
<tr>
<td>^</td>
<td>PTR</td>
<td>
<p>fqdn: p: ttl: timestamp: lo</p>
</td>
<td>Создает только PTR запись для домена fqdn.<br /></td>
</tr>
<tr>
<td>C</td>
<td>CNAME</td>
<td>
<p>fqdn: p: ttl: timestamp: lo</p>
</td>
<td>
<p>Предоставляет псевдоним для fqdn - не стоит использовать.</p>
</td>
</tr>
<tr>
<td>Z</td>
<td>SOA</td>
<td>
<p>fqdn: mname: rname: ser: ref: ret: exp: min: ttl: timestamp: lo</p>
</td>
<td>
<p>Краткое представление зоны:</p>
<pre>mname - главный сервер имен<br />rname - email адрес для связи (где первая . заменяется на @)<br />ser - серийный номер (по умолчанию равен дате изменения файла data)<br />ref - время обновления (по умолчанию 16384)<br />ret - время повтора (по умолчанию 2048)<br />exp - срок годности (по умолчанию 1048576)<br />min - минимальное время (по умолчанию 2560)<br /></pre>
</td>
</tr>
</tbody>
</table>
<p>Пример файла конфигурации:</p>
<pre>#define the authoritative nameserver<br />.apmem.org::ns1.apmem.org<br />.apmem.org::a.ns.zerigo.net<br />.100.57.146.91.in-addr.arpa::apmem.org<br />.100.57.146.91.in-addr.arpa::ns1.apmem.org<br />.100.57.146.91.in-addr.arpa::a.ns.zerigo.net<br /><br />#IP for dns names<br />=apmem.org:91.146.57.100<br />=ns1.apmem.org:91.146.57.100<br /><br />#mail servers<br />@apmem.org::mail.apmem.org<br />=mail.apmem.org:91.146.57.100<br /><br />#spf records<br />\'apmem.org:v=spf1 a mx ~all:3600<br />\'mail.apmem.org:v=spf1 a -all:3600<br /><br />#domainkeys records<br />\'mail._domainkey.apmem.org:k=rsa; t=y; p=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJ<br />BAM1VOyhF3Xe/IUcse3U7Hg4OSVfy0egxYSOD05Emxyp64YR/Pbaj/oB1p2j9LkpeeCno<br />l83KzrC1ItR3FLuwwosCAwEAAQ==;:60<br />\'_domainkey.apmem.org:t=y; o=-; r=<br /> <a href="mailto:postmaster@apmem.org">postmaster@apmem.org</a><span style="display: none;">This e-mail address is being protected from spambots. You need JavaScript enabled to view it
 </span>;:60<br /><br />#subdomains and web servers<br />=www.apmem.org:91.146.57.100<br />=clutch.apmem.org:91.146.57.100<br />=joomla.apmem.org:91.146.57.100<br />=svn.apmem.org:91.146.57.100<br /><br /><br /></pre>
<p>Для применения произведенных изменений используется команда make. Эта  команда может быть использована даже если TinyDNS уже работает. Если  что-то пойдет не по плану, то конфигуратор не заменит файл и ничего не  испортится.</p>
<h2>Принятие делегирования:</h2>
<p>1. администратор домена org должен делегировать домен apmem.org   серверу ns1.ApmeM.org, работающему на  				IP-адресе 91.146.57.100, и  серверу ns2.ApmeM.org, работающему на  IP-адресе 91.146.57.100  (вообще-то они должны отличаться)</p>
<p>2. администратор домена 57.146.91.in-addr.arpa должен делегировать   домен 100.57.146.91.in-addr.arpa серверу  a.ns.100.57.146.91.in-addr.arpa,  				работающему на IP-адресе  91.146.57.100, и серверу  b.ns.100.57.146.91.in-addr.arpa, работающему  на IP-адресе 91.146.57.100 (вообще-то они тоже должны быть разными).</p>
<h2>Обратная зона</h2>
<p>Объявление обратной зоны необходимо для получения FQDN домена по его  IP адресу. Отличительная особенность этого преобразования заключается в  том, что ip адрес необходимо указывать в обратном порядке. Подозреваю  это сделано для того, чтобы поиск по ip адресу не отличался от поиска по  имени - тоесть сперва будут искаться владельцы 91.in-addr.arpa, затем  146.91.in-addr.arpa и т.д. Выделением обратных адресов занимается <a href="http://www.ripn.net:8080/nic/IP-reg/IN-ADDR/index.html">Российский НИИ развития общественных сетей</a> или<a href="http://www.ripe.net/rs/reverse/index.html"> </a><span class="titleheader"><a href="http://www.ripe.net/rs/reverse/index.html">RIPE             Network Coordination Centre</a>.</span></p>
<p>Если выделен только 1 адрес, то строка, начинающаяся с символа \'=\' добавляет обратную зону для себя автоматически:</p>
<p>.100.57.146.91.in-addr.arpa:91.146.57.100</p>
<p>Если днс сервер распределяет имена между несколькими компьютерами, то необходимо указать это следующим образом:</p>
<p>.57.146.91.in-addr.arpa:91.146.57.100</p>
<p>Проверить ответственного за указаный ip адрес можно на сайте <a href="http://www.db.ripe.net/whois">RIPE</a></p>
<p> </p>
<h1><a name="axfrDNS"></a>axfrDNS</h1>
<hr />
<p>Сервер синхронизации создан для обмена данными между master и slaves  dns серверами. При первоначальной конфигурации axfrDNS спрашивает путь к  tinyDNS серверу, из базы которой и будут отдаваться синхранизационные  данные.</p>
<p>Основной конфигурационный файл находится по адресу axfrdns/tcp и состоит из строк, описывающих с какого  ip адреса какое действие можно делать. Каждая строка состоит из адреса,  двоеточия и списка инструкций. Строки читаются сверху вниз, до первого  совпадения. Для каждого ip адреса может существовать только одна строка.</p>
<p>Инструкции</p>
<pre># разрешить просматривать всем<br />:allow,AXFR=""<br /># разрешить самому себе делать копии домена apmem.org<br />127.0.0.1:allow,AXFR="apmem.org"<br />192.168.0.1:allow,AXFR="apmem.org"<br /># разрешить другим компьютерам делать копии домена apmem.org<br />68.71.141.22:allow,AXFR="apmem.org"<br />174.36.24.251:allow,AXFR="apmem.org"<br /># разрывать все соединения с другими запросами<br />:deny<br /></pre>
<p>По окончанию конфигурирования так же нужно выполнить команду make</p>
<h1><a name="DNSCache"></a>DNSCache                                                                                                                      
<hr />
</h1>
<p>пока ничего, еще не разбирался.</p>
<p> </p>
<h1>Полезные ссылки                                                                                                                      
<hr />
</h1>
<p><a href="http://www.lifewithdjbdns.com/">http://www.lifewithdjbdns.com</a> - руководство по установки и настройке серверов</p>
<p><a href="http://cr.yp.to/djbdns/dot-arpa.html">http://cr.yp.to/djbdns/dot-arpa.html</a> - делегирование днс адресов</p>
<p><a href="http://dnscheck.iis.se/">http://dnscheck.iis.se</a> - Проверка валидности DNS сервера</p>
<p><a href="http://www.ripn.net/">http://www.ripn.net</a> - Российский НИИ развития общественных сетей</p>
<p><a href="http://www.ripe.net/">http://www.ripe.net/</a> - <span class="titleheader">RIPE             Network Coordination Centre</span></p>
<p><a href="http://cr.yp.to/daemontools.html">http://cr.yp.to/daemontools.html</a> - описание работы daemontools</p>
<p><a href="http://lithium.opennet.ru/index.html">http://lithium.opennet.ru/index.html</a> - внятное описание настроек обоих серверов и много чего еще</p>
