<p>1 Декабря 2010 подошел к концу турнир google AI challange,  организованный университетом "University of Waterloo Computer Science  Club". В турнире приняло участие 4617 игроков (Хотя на самом деле  благодаря отсутствию возможности менять и восстанавливать пароль игроков  гораздо меньше). Из них более 1.5 тысяч игроков залили тестового бота и  больше ничего не меняли.</p>
<p>Для написания был выбран язык Java просто потому, что хотелось  написать на нем чего-нибудь более сложное чем Hello World (ну и всякой  ереси типа прикручивания логина по ГОСТ сертификатам к JBoss :), которая  и к яве то имела посредственное отношение)</p>
<p>В результате мой бот занял 291 место слив чуть более чем полностью и  не поднявшись даже до уровня, на котором он был во время проведения  самого турнира. Самое удивительное было, когда, посмотрев на логику  работы 1 места, стало понятно, что работы велись совсем не в ту сторону  :)</p>
<p>В процессе разработки был найден (а потом забыт и перепридуман) метод  поиска кратчайшего пути по взвешеному графу. Так как я забыл даже его  название, ссылок на метод не будет, а жаль, т.к. когда я его случайно  нашел, там была красивая и понятная гифка, собственно благодаря которой  мне и удалось восстановить этот алгоритм.</p>
<p> </p>
<p> </p>
<h1>Описание поведения бота</h1>
<p>По задумке исходя из этого пункта можно определить, почему бот оказался неадекват и не занял первое место.</p>
<p>Каждый ход разделен на несколько фаз:<br />* Инициализация игрового состояния на очередном ходу<br />* Защита всех своих планет, на которые напал враг<br />* Поиск планет, подходящих для атаки (и атака)<br />* Отправка всех запланированных флотов и завершение хода</p>
<h2>Текущая реализованная стратегия</h2>
<h3>Инициализация</h3>
<p>На этом этапе анализируются полученные от игрового движка данные и  заполняются структуры типа Игрок, Планета, Флот всеми необходимыми  значениями. Во время добавления флотов происходит перерасчет состояния  планет (типа через 10 ходов игрок 1 захватит планету 6 и на ней  останется 1 корабль). После добавления всех планет происходит расчет  "кратчайшего расстояния" между всеми планетами.</p>
<p>Особенностью "кратчайшего расстояния" является то, что оно пролегает  через ближайшие планеты, таким образом в случае опасности есть  возможность отозвать часть летящего флота на защиту. Однако по этой же  причине "кратчайшее расстояние" ниразу не кратчайшее, потому что планеты  редко находятся на одной линии, и почти всегда приходится лететь с некоторым отклонением от основного маршрута.</p>
<p> </p>
<h3>Защита</h3>
<p>На этапе защиты проводится поиск своих планет, на которые вылетел вражеский флот, способный ее захватить.<br /><br />Для  того, чтобы захвата не произошло на планету высылается флот со всех  ближайших планет в количестве необходимом для отражения атаки (размер  флота на планете в результате всех боев должен остаться 0).<br /><br />В  случае если все планеты защитить не удастся, приоритет отдается планетам  с большим размером, однако если выслав все флоты планету защитить  всеравно не удается, - планета остается без защиты и флот на нее не  посылается. Если не удалось защитить ниодну из атакуемых планет - ниодин  корабль не будет послан после этого этапа.<br /><br /></p>
<h3>Атака</h3>
<p>На этапе нападения каждая планета, имеющая флот, выбирает планету,  владелец которой на известный промежуток времени враг или нейтрал, и  расчитывает для нее коэффициент характеристикам, после чего на первую из  них высылается флот, необходимый для захвата (или весь оставшийся, если  на текущий момент флота не хватает).</p>
<p>При расчете не учитываются планеты, которые можно достичь по  "кратчайшему пути" пройдя по вражеским планетам. Таким образом создается  фронт, на котором и идет бой, и если враг решит его перелететь - ему  потребуется значительно больше кораблей и этим самым я защищаюсь от  бесполезной траты флота, посланного вдаль без поддержки соседних планет.</p>
<p>Характеристика планеты вычисляется по формуле:</p>
<pre>Характеристика = "Кратчайшее расстояние" + Время на восстановление потерянного флота<br />Время на восстановление = Количество вражеских кораблей на планете на момент прибытия / Прирост</pre>
<p>Итоговый результат вычисляется в ходах, необходимых для  восстановления флота, убившегося при захвате планеты. Чем он меньше, тем  полезнее планета.</p>
<p>На выбранную планету по "кратчайшему пути" отправляется весь флот.  Если же "кратчайший путь" не содержит никаких планет, то в атаку  посылается только необходимое для захвата количество кораблей (учитывая  все летящие на планету корабли и ее прирост).</p>
<h3>Завершение хода</h3>
<p>Все рассчитанные флоты отправляются движку через консольный вывод.  Никаких проверок не производится, считается, что все рассчеты верны  (иначе по правилам бот проигрывает бой).<br /><br />Также отправляется команда go, чтобы движок понял, что мы закончили.</p>
<p> </p>
<h1>Полезные ссылки</h1>
<p>Исходники движка и всего с ним связанного можно найти на  гуглькоде - <a class="external free" href="http://code.google.com/p/ai-contest/" rel="nofollow">http://code.google.com/p/ai-contest/</a></p>
