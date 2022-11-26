<p>Windows Identity Foundation (WIF) - средство, предназначенное для быстрого создания механизмов Авторизации и аутентификации, основанных на утверждениях О_о (Claim-based).  Благодаря шаблонам для Visual Studio, входящим в состав <a href="http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504">SDK</a> разработка базовых решений производится мышкой и занимает не много времени.</p>
<p>Данная статья покажет примеры базового использования Windows Identity Foundation и расскажет о трудностях и ошибках, которые могут возникнуть при работе.</p>
<h1>Установка</h1>
<p>Для разработки и работы системы необходимо скачать и установить <a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=EB9C345F-E830-40B8-A5FE-AE7A864C4D76">WIF runtime</a>. Так же для разработки поверх runtime библиотеки необходимо установить <a href="http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504">WIF SDK</a>.</p>
<p>После установки WIF SDK в Visual Studio будут добавлены несколько шаблонов для создания Claim-based и Security-Token-Service сервисов и сайтов. Так же по адресу "C:\\Program Files (x86)\\Windows Identity Foundation SDK\\v4.0\\Samples\\" появятся некоторые базовые примеры по работе с WIF.</p>
<h1>Описание</h1>
<h2>Claim-based principals</h2>
<p>В своей основе WIF содержит механизм, предназначенный для преобразования стандартных Principal классов в Claim-based Principal классы. Это осуществляется путем встраивания в процесс обработки запроса нового расширения:</p>
<pre>&lt;extensions&gt;<br />  &lt;behaviorExtensions&gt;<br />    &lt;add name="federatedServiceHostConfiguration" type="Microsoft.IdentityModel.<br />        Configuration.ConfigureServiceHostBehaviorExtensionElement, Microsoft.IdentityModel, <br />        Version=3.5.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" /&gt;<br />  &lt;/behaviorExtensions&gt;<br />&lt;/extensions&gt;</pre>
<p>После такого преобразования (которое осуществляется мышкой: Add STS Reference -&gt; ... -&gt; No STS) в коде появляется возможность получить Claim-based Principal из всех классов, ответственных за работу с правами. Например:</p>
<pre>IClaimsIdentity claimsIdentity = ((IClaimsPrincipal)Thread.CurrentPrincipal).Identities[0];</pre>
<h2>Security Token Service</h2>
<p>Так же в состав WIF SDK содержатся шаблоны для создания сервисов по генерации, проверке и удалении Security Tokens. При добавлении ссылки из любого WCF сервиса на STS сервис Visual Studio (при помощи утилиты FedUtil) преобразует web.config, а в связи с этим и результат выдачи wsdl так. чтобы все используемые приложения знали, что для авторизации на сервисе необходим токен, и что получить этот токен можно у указанного STS сервиса.</p>
<p>Обычно весь процесс по созданию запроса на токен, его получению и передаче основному запрашиваемому сервису осуществляется самим WIF и для этого даже не надо менять никакой код в приложении (разве-что он поломает имена endpoint-ов), однако иногда возникает необходимость получать токен вручную и для самого себя.</p>
<p>Для того, чтобы получать токен вручную необходимо также настроить биндинги для STS сервиса. Проще всего это опять же сделать мышкой добавив еще один ServiceReference.</p>
<pre>var factory = new WSTrustChannelFactory("WS2007HttpBinding_IWSTrust13Sync");<br />factory.ConfigureChannelFactory();<br /><br />var clientChannelFactory = new ChannelFactory&lt;ServiceReference1.IService&gt;(<br />    "WS2007FederationHttpBinding_IService");<br />clientChannelFactory.ConfigureChannelFactory();<br /><br />var channel = (WSTrustChannel)factory.CreateChannel();<br />var rst = new RequestSecurityToken(WSTrust13Constants.RequestTypes.Issue);<br />rst.AppliesTo = clientChannelFactory.Endpoint.Address;<br />var token = channel.Issue(rst);<br /><br />var clientChannel = clientChannelFactory.CreateChannelWithIssuedToken(token);<br />ViewBag.Message = clientChannel.GetData(0);<br /></pre>
<p>В дальнейшем токен можно хранить в сессии и передавать при необходимости в другие сервисы.</p>
<p>Для применения полученного токена к текущему пользователю необходимо внести некоторые изменения в web.config и в процесс обработки токена:</p>
<pre>// Now you parse and validate the token which results in a claimsidentity<br />var handlers = FederatedAuthentication.ServiceConfiguration.SecurityTokenHandlers;<br />var token = handlers.ReadToken(new XmlTextReader(new StringReader(genericToken.TokenXml.OuterXml)));<br />var identity = handlers.ValidateToken(token).First();<br /><br />// Create the session token using WIF and write the session token to a cookie<br />var sessionToken = new SessionSecurityToken(ClaimsPrincipal.CreateFromIdentity(identity));<br />FederatedAuthentication.SessionAuthenticationModule.WriteSessionTokenToCookie(sessionToken);</pre>
<p>При чтении токена из ответа могут возникнуть ошибки проверки достоверности сертификата. как вариант можно просто отключить подпись на стороне STS сервиса.</p>
<p>Для того, чтобы SessionAuthentiicationModule был задан и работал в Web.Config веб приложения нудно вставить:</p>
<pre>&lt;system.webServer&gt;
  ...
  &lt;modules runAllManagedModulesForAllRequests="true"&gt;<br />    ...
    &lt;add name="SessionAuthenticationModule" <br />      type="Microsoft.IdentityModel.Web.SessionAuthenticationModule, Microsoft.IdentityModel, <br />      Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" /&gt;
  &lt;/modules&gt;
&lt;/system.webServer&gt;
</pre>
<p>И включить хранение токена, для дальнейшей ее передачи в сервисы:</p>
<pre>  &lt;microsoft.identityModel&gt;<br />    &lt;service saveBootstrapTokens="true"&gt;</pre>
<p>Подробнее о создании приложения, использующего активные STS подключения можно почитать на <a href="http://koenwillemse.wordpress.com/2010/08/02/making-a-web-application-use-an-active-sts/">вордпрессе</a>.</p>
<p>Кроме этого если есть необходимость использовать http протокол для основной работы в web.config надо дописать строки:</p>
<pre>&lt;microsoft.identityModel&gt;<br />  &lt;service&gt;<br />    &lt;federatedAuthentication&gt;<br />      &lt;cookieHandler requireSsl="false" /&gt;<br />    &lt;/federatedAuthentication&gt;<br />  &lt;/service&gt;<br />&lt;/microsoft.identityModel&gt;</pre>
<div><span style="color: #666666; font-family: Helvetica, Arial, sans-serif; font-size: 16px; font-weight: bold;">Ошибки и их решение</span></div>
<p>При внесении (даже генераторами) указанных изменений в процессе работы иногда возникают ошибки. В этом разделе приведены описания, причины и решение ошибок (которые я смог решить :) ).</p>
<h2>The incoming policy could not be validated. For more information, please see the event log.</h2>
<p>При этом в event log`е с большой вероятностью находится сообщение от CardSpace со стэктрейсом, по которому явно видно что ничего не видно.</p>
<p>Причина: Подобная ошибка может может возникнуть от чего угодно (и возможно даже можно как-то сделать, чтобы эта причина тоже писалась в евентлог).</p>
<p>Решение: Для того, чтобы ошибка обрела очертания и смысл можно отключить использование CardSpace</p>
<pre>factory.Credentials.SupportInteractive = false;</pre>
<p>В результате в некоторых случаях все начинает работать само :), однако почти всегда вместо этой ошибки появляется более осмысленное описание и не выскакивают всякие окошки, призванные помочь, но никогда не помогающие.</p>
<h2>The address of the security token issuer is not specified.</h2>
<p>Причина: Эта ошибка возникает когда на клиентской стороне в биндинге сервиса, использующего STS, не указан сам адрес этого STS сервиса. Обычно вместо него написана строка типа</p>
<pre>&lt;issuer address="http://schemas.microsoft.com/2005/12/ServiceModel/Addressing/Anonymous" /&gt;</pre>
<p>Однако в случае с ручным получением токена эта ошибка не возникает (так как токен уже у нас есть, и не надо искать где его получить)</p>
<p>При автоматической генерации такое происходит в случае, если STS сервис был не доступен, или адрес mex endpoint`а ссылается на несуществующую страницу (например протокол https вместо http).</p>
<p>Решение: Включить STS сервис, поправить линки в конфиге Claim-based сервиса, проверить его доступность. После этого обновить ServiceReference в основном приложении. В нормальном состоянии тэг issuer выглядит примерно так:</p>
<pre>&lt;issuer address="http://localhost:62491/ClaimsAwareService1_STS/Service.svc/IWSTrust13"<br />  binding="ws2007HttpBinding" bindingConfiguration=<br />  "http://localhost:62491/ClaimsAwareService1_STS/Service.svc/IWSTrust13"&gt;<br />  &lt;identity&gt;<br />    &lt;userPrincipalName value="APMEM-PC\\ApmeM" /&gt;<br />  &lt;/identity&gt;<br />&lt;/issuer&gt;<br /></pre>
<h2><span class="st">System.ServiceModel.FaultException: ID3242: The security token could not be authenticated or authorized.</span></h2>
<p><span class="st">Ошибка возникает не одна и в большинстве случаев на сторону клиента приходит только оставшийся от нее текст ошибки. Сам по себе текст ни о чем не говорит, но какбэ намекает, что проблема где-то в процессе обработки пришедшего на STS сервис запроса о токене.</span></p>
<p><span class="st">Причина: Причин ошибки может быть очень много, но в основе своей лежит список SecurityTokenHandlers, содержащий классы обработки токенов. По умолчанию после создания STS сервиса мышкой их 8, среди которых есть читатели xml-ки, проверятели login/password и многое другое.</span></p>
<p><span class="st">Например при смене clientCredentialType на UserName в список добавляется элемент WindowsUserNameSecurityTokenHandler, который проверяет входит ли указанный login/password пользователь в группу ServerIdentityUsers (или как-то так, не смог сразу найти). При отправке кастомных credentials, которых нет в этой группе, система падает с этой очень объясняющей ошибкой.</span></p>
<p><span class="st">Решение: проверить (видимо в google.com) из-за какого элемента все падает и подменить его в зависимости от нужд. Например для своего login/password обработчика можно попробовать код, приведенный в 4 или 8 примере из <a href="http://claimsid.codeplex.com/">набора более сложных примеров</a>. Немного модифицированный код можно посмотреть <a href="index.php/code/22-customusernamesecuritytokenhandler">у меня на сайте</a>.</span></p>
<p><span class="st">Для изменения или добавления нового обработчика существует специальная секция в web.config сервиса:</span></p>
<pre><span class="st">  &lt;microsoft.identityModel&gt;<br />    &lt;service name="App_Code.Service1"&gt;<br />      &lt;securityTokenHandlers&gt;<br />        &lt;clear /&gt;<br />        &lt;remove type="Microsoft.IdentityModel.Tokens.WindowsUserNameSecurityTokenHandler, Microsoft.IdentityModel, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/&gt;<br />        &lt;add type="App_Code.CustomUserNameSecurityTokenHandler, App_Code" /&gt;<br />      &lt;/securityTokenHandlers&gt;<br />    &lt;/service&gt;</span></pre>
<p><span class="st">Однако приведенный код не работает в случае, когда сервис создается фабрикой (что собственно и генерирует Visual Studio по шаблону). Для того чтобы применить эти изменения в коде необходимо добавить строку </span></p>
<pre><span class="st">this.SecurityTokenHandlers.AddOrReplace(new CustomUserNameSecurityTokenHandler());</span></pre>
<p><span class="st">в конструктор фабрики CustomSecurityTokenServiceConfiguration</span></p>
<p><span class="st">ВНИМАНИЕ: Слово AddOrReplace обязательно, так как в списке может существовать только один обработчик одного типа (в данном случае по проверке логина с паролем). Иначе вы увидите ошибку что-то типа Already Registered.</span></p>
<h2><span class="st">Https работает а Http нет</span></h2>
<p><span class="st">При установке токена вручную (активный способ работы с STS) логин перестает работать по http, хотя по https работает как надо. </span></p>
<p><span class="st">Причина: Не до конца сконфигурирована работа с STS на стороне веб клиента.</span></p>
<p><span class="st">Решение: Добавить в web.config строки:</span></p>
<p><span class="st"> </span></p>
<pre>&lt;microsoft.identityModel&gt;<br />  &lt;service&gt;<br />    &lt;federatedAuthentication&gt;<br />      &lt;cookieHandler requireSsl="false" /&gt;<br />    &lt;/federatedAuthentication&gt;<br />  &lt;/service&gt;<br />&lt;/microsoft.identityModel&gt;</pre>
<p> </p>
<h1>Полезные ссылки</h1>
<p><a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=EB9C345F-E830-40B8-A5FE-AE7A864C4D76">http://www.microsoft.com/downloads/en/details.aspx?FamilyID=EB9C345F-E830-40B8-A5FE-AE7A864C4D76</a> - Windows Identity Foundation runtime</p>
<p><a href="http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504">http://www.microsoft.com/downloads/details.aspx?FamilyID=c148b2df-c7af-46bb-9162-2c9422208504</a> - Windows Identity Foundation SDK</p>
<p><a href="http://claimsid.codeplex.com/">http://claimsid.codeplex.com/</a> - набор более сложных примеров для Claim-based сервисов.</p>
<p><a href="index.php/code/22-customusernamesecuritytokenhandler">index.php/code/22-customusernamesecuritytokenhandler</a> - пример своей реализации проверки login/password/</p>
<p><a href="http://koenwillemse.wordpress.com/2010/08/02/making-a-web-application-use-an-active-sts/">http://koenwillemse.wordpress.com/2010/08/02/making-a-web-application-use-an-active-sts/</a> - пошаговая инструкция по созданию веб приложений, использующих активное STS подключение.</p>
