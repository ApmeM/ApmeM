<p>Регулярные выражения используются для проверки правильности ввода определенного типа полей (таких как адрес электронной почты, адрес сайта или телефонный номер), или поиска подстроки в строке по указанному шаблону. Регулярные выражения обладают большой гибкостью и множеством подходов к написанию и используются практически везде.</p>
<p>Для ускорения поиска в статье собраны наиболее часто используемые регулярные выражения:</p>
<h2>1. Проверка правильности ввода email адреса.</h2>
<pre>^(?("")("".+?""@)|(([0-9a-zA-Z]((\\.(?!\\.))|[-!#\\$%&amp;\'\\*\\+/=\\?\\^`\\{\\}\\|~\\w])*)<br />(?&lt;=[0-9a-zA-Z])@))(?(\\[)(\\[(\\d{1,3}\\.){3}\\d{1,3}\\])|(([0-9a-zA-Z][-\\w]*[0-<br />9a-zA-Z]\\.)+[a-zA-Z]{2,6}))$</pre>
<p>Пример: someone@example.com</p>
<h2>2. Проверка правильности ввода URL адреса:</h2>
<pre>/(?:(?:ht|f)tp(?:s?)\\:\\/\\/|~\\/|\\/)?(?:\\w+:\\w+@)?(?:(?:[-\\w]+\\.)+(?:ru|su|com|<br />org|net|gov|mil|biz|info|mobi|name|aero|jobs|museum|travel|[a-z]{2}))(?::<br />[\\d]{1,5})?(?:(?:(?:\\/(?:[-\\w~!$+|.,=]|%[a-f\\d]{2})+)+|\\/)+|\\?|#)?(?:(?:\\?(?:<br />[-\\w~!$+|.,*:]|%[a-f\\d{2}])+=(?:[-\\w~!$+|.,*:=]|%[a-f\\d]{2})*)(?:&amp;(?:[-\\w~<br />!$+|.,*:]|%[a-f\\d{2}])+=(?:[-\\w~!$+|.,*:=]|%[a-f\\d]{2})*)*)*(?:#(?:[-\\w~!$<br />+|.,*:=]|%[a-f\\d]{2})*)?/</pre>
<p>или проще</p>
<pre>^(ht|f)tp(s?)\\:\\/\\/[0-9a-zA-Z]([-.\\w]*[0-9a-zA-Z])*(:(0-9)*)*(\\/?)([a-zA-Z0-9<br />\\-\\.\\?\\,\\\'\\/\\\\\\+&amp;amp;%\\$#_]*)?$</pre>
<p>Пример: http://apmem.org</p>
<h2>3. Проверка правильности ввода телефонного номера:</h2>
<pre>^[01]?[- .]?(\\([2-9]\\d{2}\\)|[2-9]\\d{2})[- .]?\\d{3}[- .]?\\d{4}$</pre>
<p>Пример:</p>
<p>(425) 555-0123<br /> 425-555-0123<br /> 425 555 0123<br /> 1-425-555-0123</p>
<h1>Полезные ссылки</h1>
<p><a class="external free" href="http://flanders.co.nz/2009/11/08/a-good-url-regular-expression-repost/" rel="nofollow">http://flanders.co.nz/2009/11/08/a-good-url-regular-expression-repost/</a> - пример ссылки проверки URL</p>
<p><a href="http://msdn.microsoft.com/en-us/library/ff650303.aspx">http://msdn.microsoft.com/en-us/library/ff650303.aspx</a> - несколько примеров регулярных выражений от M$</p>
<p><a href="http://regexlib.com/">http://regexlib.com/</a> - библиотека регулярных выражений</p>
<p><a href="http://www.ex-parrot.com/~pdw/Mail-RFC822-Address.html">http://www.ex-parrot.com/~pdw/Mail-RFC822-Address.html</a> - RFC822 mail Address validator</p>
