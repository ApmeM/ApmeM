<p>Пошаговая инструкция по запуску эмулятора телефона с Windows CE в Visual Studio и подключение его к интернету.</p>
<h1>Загрузка</h1>
<ul>
<li>В первую очередь необходимо скачать MS VS 2008 или ранее (см "Отстрел багов" п.1).</li>
<li>Также необходимо скачать "<a href="http://www.microsoft.com/windowsmobile/ru-ru/downloads/microsoft/device-center-download.mspx">Центр устройств Windows Mobile 6.1</a>". <strong>ВНИМАНИЕ </strong>- при скачивании по ссылке требуется проверка подлинности Windows (Genuine)</li>
</ul>
<h1>Установка</h1>
<p>Процесс установки всего скачанного проходит быстро и безболезненно. <strong>ВНИМАНИЕ </strong>- При установки Центра устройств появится значок поиска драйверов, который с большой вероятностью скажет, что устройство не подключено. На самом деле так и есть, так как эмуляторы еще не запущены и не подключены, по этому предупреждение игнорируется.</p>
<p>
</p>
<h1>Настройка</h1>
<p>Существует несколько способов подключения устройства к интернету. Описание можно найти в <a href="http://blogs.msdn.com/b/alexdan/archive/2009/08/25/enabling-network-connectivity-on-a-windows-mobile-device-emulator.aspx">блогах Microsoft</a>.</p>
<p>Пример простейшей настройки (взят из блогов):</p>
<ol>
<li>Запустить "Центр устройств Windows Mobile" в Windows Vista/7 или ActiveSync 4.5 под Windows XP.</li>
<li>В нем большая кнопка "Mobile Device Settings", затем "Connection Settings".</li>
<li>В открывшемся окне проверить, что галочка "Allow  Connections to one of the following" установлена, и в списке выбрано "DMA".</li>
<li>Нажать "OK".</li>
<li>Запустить MS VS 2008 (см. Отстрел багов п.1)</li>
<li>Запустите эмулятор:<ol> </ol> 
<ul>
<li>В разделе меню "Tools" пункт "Connect to device"</li>
<li>Запустить мобильное приложение на исполнение</li>
</ul>
<ol> </ol></li>
<li>В разделе меню "Tools" нажать "Device Emulator Manager". <br /> 
<ul>
<li>Должна появиться зеленая стрелочка на против включенного устройства. (Если нет - нажать "Refresh")</li>
</ul>
</li>
<li>Нажать правой кнопкой мыши по запущенному устройству и выбрать пункт "Cradle" - это поставит устройство на виртуальную подставку, и WMDC/AS должны обнаружить его (см Отстрел багов п.2).</li>
<li>В "Центре устройств Windows Mobile" настроить подключение (см Отстрел багов п.2).</li>
</ol>
<h1>Отстрел багов</h1>
<ol>
<li>MS VS 2010 не работает с мобильными устройствами в связи с тем, что Microsoft не предоставила поддержки Compact  Framework (и всех соответствующих плюшек типа шаблона для Compact Framework) под MS VS 2010. Возможно в  SP1+ эта возможность таки появится, но пока ее <a href="http://msdn.microsoft.com/en-us/library/sa69he4t%28VS.100%29.aspx">нет</a>.</li>
<li>Обязательно настройте подключение к устройству ("Setup Partnership").  Если настройку не провести - в некоторых случаях устройство не  обнаруживается при повторном подключении. Не известно что нужно сделать,  чтобы оно обнаружилось снова, но через некоторое время (хибернейт на  ночь) он обнаружился сам по неизвестным причинам.</li>
<li>Если Центр управления не обнаруживает устройство интернеты советуют:      
<ul>
<li>Проверить правильность настройки Центра (например слова MDA и галочку) - обычно и так стоит, но стоит проверить</li>
<li>Переустановить WMDC (должно помочь, если с первого раза не настроили подключение из п.2)</li>
<li>При использовании COM поглядеть \\HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows CE Services на предмет "SerialPort" и "SerialBaudRate".</li>
<li>Почистить реестр по адресам<br />* HKCU\\Software\\Microsoft\\Windows CE Services\\Partners <br /> * HKLM\\Software\\Microsoft\\Windows Portable Devices\\Devices <br />* HKEY_USERS\\S-1-5-21-1534184335-3658833746-3301443389-1004\\Software\\Microsoft\\Windows      CE Services\\Partners</li>
<li>Проверить настройки фаирвола (вообще маловероятно, но вдруг).</li>
</ul>
</li>
</ol>
<h1>Полезные ссылки</h1>
<p><a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=04d26402-3199-48a3-afa2-2dc0b40a73b6&amp;DisplayLang=en">http://www.microsoft.com/downloads/en/details.aspx?FamilyID=04d26402-3199-48a3-afa2-2dc0b40a73b6&amp;DisplayLang=en</a> - Ссылка на скачивание "Virtual PC 2007" с сайта Microsoft (без этого не активируется настройка использования сетевой карты в эмуляторе, ну и может чего еще). Не обязательно к установке.</p>
<p><a href="http://www.microsoft.com/downloads/en/details.aspx?displaylang=en&amp;FamilyID=e3821449-3c6b-42f1-9fd9-0041345b3385">http://www.microsoft.com/downloads/en/details.aspx?displaylang=en&amp;FamilyID=e3821449-3c6b-42f1-9fd9-0041345b3385</a> - ссылка на скачивание ".Net Compact Framework 3.5". В принципе он ставится вместе с 2008 студией, но возможно он нужен для более ранних версий.</p>
<ul>
</ul>
<p><a href="http://www.microsoft.com/windowsmobile/ru-ru/downloads/microsoft/device-center-download.mspx">http://www.microsoft.com/windowsmobile/ru-ru/downloads/microsoft/device-center-download.mspx</a> -  ссылка на скачивание  "Центра устройств Windows Mobile 6.1". <strong> </strong>Необходима для осуществления связи между устройством и компьютером. При скачивании по ссылке требуется проверка подлинности Windows (Genuine)</p>
<p><a href="http://blogs.msdn.com/b/alexdan/archive/2009/08/25/enabling-network-connectivity-on-a-windows-mobile-device-emulator.aspx">http://blogs.msdn.com/b/alexdan/archive/2009/08/25/enabling-network-connectivity-on-a-windows-mobile-device-emulator.aspx</a> - Описание подключения мобильного устройства</p>
<p><a href="http://forum.soft32.com/pda/Sync-problem-WMDC-fixed-rolling-back-WMDC-ftopict79588.html">http://forum.soft32.com/pda/Sync-problem-WMDC-fixed-rolling-back-WMDC-ftopict79588.html</a> - Описание распространенных ошибок при подключении</p>
<p> </p>
