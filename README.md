# Identifierade problem i Messy Labbage
Skriven av Johnny Pesola (jp222px) December 2015

-  [Problem 1: SQL injections - Ej önskvärd databasåtkomst och manipulering av data.](#p1)
-  [Problem 2: Lagring av känsligt data](#p2)
-  [Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker](#p3)
-  [Problem 4: Säkerhetskontroll för funktioner saknas](#p4)
-  [Problem 5: Inget skydd för "Cross-Site Request Forgery" (CSRF)](#p5)
-  [Problem 6: Referenser till javascriptfiler script i sidhuvudet](#p6)
-  [Problem 7: Onödiga referenser till filer som saknas eller inte används](#p7)
-  [Problem 8: Onödigt stora javascriptfiler](#p8)
-  [Problem 9: Inline kod (javascript och css)](#p9)
-  [Egna övergripande reflektioner](#refl)
-  [Referenser](#refer)

***

## Säkerhetsproblem (Backend)

### <a name="p1"></a>Problem 1: SQL injections - Ej önskvärd databasåtkomst och manipulering av data.

#### Vad problemet innebär
Säkerhetshålet innebär i stora drag att attackeraren skriver sitt postdata på ett sådant sätt så att ej önskvärd databasåtkomst och eventuell manipulering av datat i databasen blir möjlig.[1]

#### Eventuella följder
Följderna av detta är katastrofala. Attackeraren kan logga in som en administratör i systemet. Användarnas lagrade känsliga uppgifter kan bli tillgängliga för attackeraren (Se Problem 2 för mer info). Attackeraren kan också eventuellt manipulera datat i databastabellen efter egna önskemål (till exempel byta lösenord på administratören), eller ta bort data.
   
#### Identifierade SQL injections i applikationen
När en inloggning sker så finns det i applikationen en sårbarhet för denna typ av attack. I källkoden i filen "appModules/login/lib/login.js" så konkateneras strängarna till en enda sql sats utan att de inkommande värderna kontrolleras eller saneras. Det här är en väldigt allvarlig typ av attack och bör undvikas genom att använda tekniken för databindning som används vid skapande och borttagning av meddelanden i applikationen. [1]
    
Applikationen har testats med att utsättas för dessa typer av attacker vid skapande och borttagning av meddelanden och för dessa metoder hittades inga sårbarheter för denna typ av attack.
    
Det jag kontret lyckats göra med hjälp attacken i applikationen är att logga in med sql injections utan att känna till användarnamn eller lösenord, men jag har inte lyckats manipulera datat eller visa känslig data.

#### Hur problemen kan åtgärdas
Organisationen OWASP har på sin hemsida en allmän bra guide för att undvika detta [2], men just i det här fallet så är det rekommenderat att använda sig av sqlite3 bibliotekets dokumenterade metoder [3].
    
Tittar man i filen "appModules/message/messageModel.js" så ser man att SQL satserna använder syntaxen "db.run("INSERT INTO message (message, userID) VALUES (?, ?)", [message, userID]" vilket innebär att inkommande värderna "saneras" i samband med då värderna ifrån arrayen flyttas till frågetecknen i sql satsen.
    
Genom att göra på detta sätt genomgående i hela applikationen så bör problemet åtgärdas.

***

### <a name="p2"></a>Problem 2: Lagring av känsligt data

#### Vad problemet innebär
När känslig data inte krypteras/hashas på ett korrekt sätt så är följderna av detta är katastrofala ifall någon obehörig skulle få åtkomst till dessa till exempel genom en SQL-injection attack.[4]

#### Eventuella följder
Ett exempel är att inloggningsuppgifter kan användas till att logga in på andra hemsidor och tjänster där användaren har samma användarnamn och lösenord. Väldigt känsliga bilder skulle kunna användas för utpressning av attackeraren. Kreditkortsuppgifter och personnummer skulle kunna användas för e-handel.

#### Identifierad ohashad känslig data i applikationen
I det här fallet är det extra illa eftersom lösenordet sparas i klartext och inte "hashas" till ett värde som inte går att gissa sig till av en attackerare. Hashning innebär att lösenordet översätts till ett större antal till synes slumpmässiga tecken som INTE går att översätta tillbaka till ursprungslösenordet. Ifall attackeraren nu får tag i inloggningsuppgifterna så kan han direkt börja använda dessa till att logga in på andra hemsidor/e-tjänster. Men ifall lösenordet hade varit hashat (med en säker metod) så hade attackeraren haft betydligt mindre nytta av dessa uppgifter.[5]

#### Hur problemet kan åtgärdas
Ett tips är att använda sig av följande [bibliotek](https://nodejs.org/api/crypto.html) för att hasha lösenord. Viktigt att tänka på vid hashning är att använda sig av olika 32 eller 64 bitars "salt" för varje användares lösenord. Detta "salt" värde kan sparas i ett separat fält intill databaslösenordet. [5]

***

### <a name="p3"></a>Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker

#### Vad problemet innebär
Om data inte valideras korrekt, mer specifikt att javascript, iframe-taggar, html-element med src attribut (andra kodspråk och taggar kan förekomma) inte filtreras/bearbetas i innehållet som postas från klient till server uppstår denna säkerhetsrisk. När innehållet sedan visas i klienters webbläsare så körs denna javascript-kod hos klienten och kan då exempelvis stjäla klientens sessionskaka och vidarebefordra denna till attackeraren. I stora drag har attackeraren kontroll över det mesta som presenteras för och som finns lagrat hos klienten gällande den aktuella webbsidan. [6]

#### Eventuella följder
Eftersom attackeraren har kontroll över allt som visas på den kapade hemsidan hos klientens webbläsare så kan användaren enkelt luras till att exempelvis att ange känsliga uppgifter (kreditkort, personnummer, mm). Men det allvarligaste är nog att sessionen enkelt kan stjälas av attackeraren och detta innebär att attackeraren blir inloggad på hemsidan/applikationen som användaren utan att känna till användarens inloggningsuppgifter.[7]

#### Identifierade problem i applikationen
Applikationen bearbetar/filtrerar inte bort javascript eller andra känsliga html taggar ifrån meddelandetexten som postas in. 

#### Hur problemet kan åtgärdas
Det enklaste sättet att skydda sig är att filtrera det postade innehållet och tillämpa whitelists för tillåtna tecken. Annars rekommenderas ett bibliotek för att tillämpa detta då det kan vara svårt att missa alla källor där detta kan uppstå. Organisationen OWASP har en bra källa med regler att tänka på om man väljer att göra detta själv [8]. Sammanfattningsvis är regeln att inte placera opålitlig data inom någon html tag (attributen), och att alltid filtrera datat som finns emellan start- och sluttaggen. 
   
[Här](https://github.com/chriso/validator.js) är ett förslag på ett bibliotek som skulle vara ett alternativ för projektet

***

### <a name="p4"></a>Problem 4: Säkerhetskontroll för funktioner saknas

#### Vad problemet innebär
Attackerare som känner till eller som kan gissa sig till adresser gömda adresser i systemet kan att utföra funktioner eller metoder som de inte ska ha rätt till egentligen. Detta är möjligt eftersom det inte finns några rättighetskontroller på funktionerna/metoderna.[9]

#### Eventuella följder
Följderna är att objekt av olika slag kan skapas, ändras eller tas bort eller mer avancerade operationer kan utföras i applikationen som attackeraren inte ska ha rätt till. Exempelvis skulle användare skulle kunnas tas bort, eller i den här applikationens fall: att meddelanden kan tas bort bara genom att känna till adressen och de rätta post-parametrarna.

#### Identifierade problem i applikationen
I applikationen går det att gå in på index-sidan direkt utan att behöva logga in. Detta genom att helt enkelt ange adressen i webbläsarens adressfält. Antagligen ska detta endast vara möjligt för de inloggade användarna.   
   
Ett annat betydligt större problem är det går att ta bort meddelanden utan någon som helst inloggning, bara genom att känna till adressen och parametrarna. Exempelvis: Om en POST med värdet (messageID = 3) skickas till http://localhost:3000/message/delete så tas meddelandet med id-numret 3 bort oberoende om man är inloggad eller inte, eller vem man är inloggad som.

En säkerhetskontroll saknas även (en check om användaren är inloggad) för att hämta meddelanden. Eftersom det i template filen default.html körs javascript som anropar metoden MessageBoard.getMessages så hämtas meddelanderna till klientens webbläsare när denne anländer till startsidan, utan att ens ha behövt logga in.

#### Hur problemet kan åtgärdas

En säkerhetskontroll, fördelaktigen efter principen ACL behöver tillämpas. OWASP har en generell guide kring autentisering som kan vara bra att ta del av [10].

***

### <a name="p5"></a>Problem 5: Inget skydd för "Cross-Site Request Forgery" (CSRF)

#### Vad problemet innebär

Det är möjligt för andra hemsidor, webb-applikationer och andra illasinnade källor att anropa funktionalitet i systemet om en användare är inloggad i messy-labbage applikationen. [17]

#### Eventuella följder

Användare kan, utan att de är medvetna om det och utan medgivande manipulera messy-labbage applikationens innehåll genom att surfa in på andra hemsidor. De illasinnade applikationerna anropar helt enkelt exempelvis http://localhost:3000/message med en POST med ett nytt meddelande genom javascript i användarens webbläsare i samband med att användaren besöker den illasinnade webbplatsen. I och med att användaren fortfarande är inloggad i systemet så går detta utan problem.

#### Identifierade problem i applikationen

I applikationen finns det inget skydd mot CSRF alls på någon sida.

#### Hur problemet kan åtgärdas

Genom att tillämpa Synchronizer Token Pattern och generera en slumpmässig sträng (token), placera den i ett gömt formulärfält för varje POST request, som servern sedan validerar kan detta problem lösas. De illasinnade hemsidorna/källorna kan omöjligt gissa sig till det slumpmässiga strängarna förutsatt att detta är korrekt implementerat. [11]

***

## Prestandaproblem (Front end)

### <a name="p6"></a>Problem 6: Referenser till javascriptfiler script i sidhuvudet.

#### Vad problemet innebär

När det finns referenser till externa scriptfiler i sidhuvudet så är det ett prestantaproblem då renderingen av sidan och hämtningen av andra resurser stannar tills webbläsaren har hämtat dessa javascriptfiler. Först när detta är färdigt hämtas resterande resurser och sidan renderas.[12]

#### Eventuella följder

För hemsidebesökare är sidan helvit utan innehåll tills dess att scripten har hämtats och lästs in av webbläsaren, först då får klienter en visuell bekräftelse på att sidan över huvud taget laddar. Klienter med dåliga uppkopplingar (mobiltelefoner) upplever detta värst.

#### Identifierade problem i applikationen

I filen appModules/siteViews/layouts/partials/head.html så finns det script-taggar som refererar till externa javascriptfiler.

#### Hur problemet kan åtgärdas

Genom att placera scriptreferenserna i html-dokumentets slut så undviks detta problem. Då hämtas och laddas javascripten in först när html dokumentet laddats in och användaren blivit bemött av DOM:en och CSSOM:en[12]

***

### <a name="p7"></a>Problem 7: Onödiga referenser till filer som saknas eller inte används.

#### Vad problemet innebär

När det finns referenser till filer som saknas eller inte används så resulterar det i onödiga HTTP anrop till servern som belastar både servern och klienter. [13]

#### Eventuella följder

När klienters webbläsare försöker ladda ner dessa icke existerande samt onödiga filer så förhindras övriga resurser att laddas ner. Detta gör att sidan renderas onödigt sakta, speciellt då javascript referenserna finns i html-dokumentets sidhuvud.

#### Identifierade problem i applikationen

I filerna appModules/siteViews/layouts/partials/head.html och appModules/login/views/index.html finns referenser till filer som inte existerar.
    
Det finns även filer som laddas in i onödan: Stylesheet referensen till "//fonts.googleapis.com/icon?family=Material+Icons" verkar heller inte användas någonstans i dokumentet, vilket gör hämtningen av denna fil helt onödig.
    
På start/login-sidan så laddas dessa två javascript-filer in i onödan eftersom de inte används där: "/static/javascript/Message.js", "/static/javascript/MessageBoard.js".
    
Bakgrundsbilden "/static/images/b.jpg" används inte visuellt på sidan och laddas in i onödan.
    
När man är inloggad laddas filen "/static/css/signin.css" i onödan eftesom den bara används på startsidan.
    
CSS filen "/static/css/bootstrap.css" innehåller många onödiga stildefinitioner som inte används.

#### Hur problemet kan åtgärdas

Ta bort referenserna till de icke existerande dokumenten och samt de som laddas in i onödan.[13] Använd inte det gemensamma sidhuvudet i start/login-sidan. Ta bort onödiga stilar definierade i Bootstrap.css filen.

***

### <a name="p8"></a>Problem 8: Onödigt stora javascriptfiler

#### Vad problemet innebär

Det finns javascript-filer som är onödigt stora eftersom att de innehåller onödig kod som inte används samt att de inte är minifierade. [14]

#### Eventuella följder

Sidan renderas onödigt sakta för klienterna.

#### Identifierade problem i applikationen

I applikationen används en onödigt stor version av jquery som inte är minifierad. Denna version har en hel del kod för jquery moduler som inte används i applikationen. Den enda modulen som egentligen används är ajax-modulen.

#### Hur problemet kan åtgärdas

Förminska och minifiera javascripten. Kika [här](https://github.com/jquery/jquery#how-to-build-your-own-jquery) för att se hur det går att bygga ett minifierat jquery bibliotek med bara ajax-modulen.

***

### <a name="p9"></a>Problem 9: Inline kod (javascript och css)

#### Vad problemet innebär

Inline- javascript och css går förvisso snabbare för webbläsaren att läsa in första gången som sidan laddas in, men tappar samtidigt möjligheten att bli cache:at hos klientens webbläsare för att sedan hämtas därifrån de resterande gånger som sidan laddas in.[15]

#### Eventuella följder

Inline-kod cache:as inte och detta gör i längden att sidan tar längre tid att hämta för klienten. Det är generellt bättre att placera css- och javascript koden i externa filer, då chansen är större att sidan laddar snabbare med hjälp av att webbläsaren cache:ar dessa filer (förutsatt att webbservern är konfigurerad för detta). 

#### Identifierade problem i applikationen

Filen appModules/siteViews/layouts/default.html innehåller javascript-kod som bättre hör hemma i MessageBoard.js
Filerna appModules/message/views/index.html och appModules/siteViews/layouts/default.html innehåller css kod som (för cachningens skull) kan ligga i en egen css fil.

#### Hur problemet kan åtgärdas

Flytta inline javascriptkoden ifrån default.html till MessageBoard.js istället.
Flytta inline csskoden ifrån index.html till en egen css fil och länka denna i sidhuvudet (head.html). [15]

***

## <a name="refl"></a>Egna övergripande reflektioner

### Säkerhet

Häromdagen satt jag och tittade på flödet frågor som strömmade in i Stackoverflow. Många ställde frågor om varför inte en viss funktion inte fungerade som den skulle i deras (i det här fallet php) kod, men det jag reagerade på var att de inte hade någon som helst säkerhetstänk. SQL-satserna var vidöppna för attacker och det fanns oftast ingen access-kontroll för känsliga funktioner. Det värsta av allt var det att de som svarade på frågorna inte fokuserade någonting på att påtala säkerhetsbristerna, utan mest verkade tävla om att svara på den ställda frågan så fort som möjligt för att få högre poäng på sitt konto. De tyckte väl helt enkelt inte att det var värt besväret. 
    
Det är inte konstigt att det finns många nybörjare som gör osäkra hemsidor på internet när det inte finns någon som påtalar deras brister, inte ens på Stackoverflow som jag anser är en bra källa för information.

### Optimering

Nu när HTTPD/2 knackar på dörren så känns vissa tidskrävande optimeringar onödiga. Minifiering blir lite onödigt eftersom det datat ändå komprimeras binärt. Domain sharding kommer att rentav drar ner prestandan för HTTP/2 och bör därför inte användas. In-line resurser blir onödiga i och med att webservrarna kan puscha resurser direkt till klienten. [16] Det är trevligt att dessa optimeringar kommer att skötas på protokollnivå istället för att man som webbutvecklare är tvungen att lägga lika mycket tid på det som idag. Fast det är ju förutsatt att webservern stödjer och är konfigurerad för detta samt att alla klienter använder en modern webbläsare.

***

## <a name="refer"></a>Referenser

[1] The Open Web Application Security Project, "Top 10 2013-A1-Injection", OWASP, Juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Hämtad: 4 december, 2015].

[2] The Open Web Application Security Project, "SQL Injection Prevention Cheat Sheet", OWASP, November 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet [Hämtad: 4 december, 2015].

[3] Mapbox, "Github - sqlite3 API Wiki", Github, November 2015 [Online] Tillgänglig: https://github.com/mapbox/node-sqlite3/wiki/API [Hämtad: 12 nobemver, 2015]

[4] The Open Web Application Security Project, "Top 10 2013-A6-Sensitive Data Exposure", OWASP, Juni 2013, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure [Hämtad: 4 december, 2015].

[5] The Open Web Application Security Project, "Password Storage Cheat Sheet", OWASP, November 2015, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet [Hämtad: 4 december, 2015].

[6] The Open Web Application Security Project, "Top 10 2013-A3-Cross-Site Scripting (XSS)", OWASP, Februari 2014, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS) [Hämtad: 4 december, 2015].

[7] The Open Web Application Security Project, "Cross-site Scripting (XSS)", OWASP, December 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/Cross-site_Scripting_(XSS) [Hämtad: 4 december, 2015].   

[8] The Open Web Application Security Project, "XSS (Cross Site Scripting) Prevention Cheat Sheet", OWASP, Septempber 2015, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet [Hämtad: 4 december, 2015].

[9] The Open Web Application Security Project, "Top 10 2013-A7-Missing Function Level Access Control", OWASP, Juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A7-Missing_Function_Level_Access_Control [Hämtad: 4 december, 2015].   

[10] The Open Web Application Security Project, "Guide to Authorization," OWASP, Maj 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_Authorization [Hämtad: 4 december, 2015].

[11] The Open Web Application Security Project, "Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet", OWASP, November 2015, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet

[12] Steve Sounders, High Performance Websites, O'Reilly, 2007, sid. 45.

[13] Steve Sounders, High Performance Websites, O'Reilly, 2007, sid. 10.

[14] Steve Sounders, High Performance Websites, O'Reilly, 2007, sid. 69.

[15] Steve Sounders, High Performance Websites, O'Reilly, 2007, sid. 55.

[16] Ilya Grigorik, "Chapter 12. HTTP/2", O'Reilly, 2013, [Online]  Tillgänglig: http://chimera.labs.oreilly.com/books/1230000000545/ch12.html#HTTP2_PUSH

[17] The Open Web Application Security Project, "Top 10 2013-A8-Cross-Site Request Forgery (CSRF)", OWASP, September 2013, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Top_10_2013-A8-Cross-Site_Request_Forgery_(CSRF)
