# Problem - Messy Labbage (Johnny Pesola / jp222px)

[Problem 9](#p9)

## Säkerhetsproblem (Backend)

Analysen av säkerhetsproblemen i applikationen är starkt influerad av organisationen OWASP:s top 10 lista över säkerhetshål. Denna lista kan hittas [här](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)

### Problem 1: SQL injections - Ej önskvärd databasåtkomst och manipulering av data.

#### Vad problemet innebär
Säkerhetshålet innebär kort och gott att attackeraren skriver sitt postdata på ett sådant sätt så att ej önskvärd databasåtkomst och manipulering av datat i databasen blir möjlig.

#### Eventuella följder
Följderna av detta är katastrofala. Attackeraren kan logga in som en administratör i systemet. Användarnas lagrade känsliga uppgifter kan bli tillgängliga för attackeraren (Se Problem 2 för mer info). Attackeraren kan också eventuellt manipulera datat i databastabellen efter egna önskemål (till exempel byta lösenord på administratören), eller ta bort allt data. [1]
   
#### Identifierade SQL injections i applikationen
När en inloggning sker så finns det i applikationen en sårbarhet för denna typ av attack. I källkoden i filen "appModules/login/lib/login.js" så konkateneras strängarna till en enda sql sats utan att de inkommande värderna kontrolleras eller saneras. Det här är en väldigt allvarlig typ av attack och bör undvikas genom att använda tekniken för databindning som används vid skapande och borttagning av meddelanden. Mer om detta nedan. [1]

Applikationen har testats med att utsättas för dessa typer av attacker vid skapande och borttagning av meddelanden och för dessa metoder hittades inga sårbarheter för denna typ av attack.
    
Det jag kontret lyckats göra med hjälp attacken i applikationen är att logga in med sql injections utan att känna till användarnamn eller lösenord, men jag har inte lyckats manipulera datat eller visa känslig data.

#### Hur problemen kan åtgärdas
Organisationen OWASP har på sin hemsida en allmän bra guide för att undvika detta [2], men just i det här fallet så är det rekommenderat att använda sig av sqlite3 bibliotekets dokumenterade metoder [3].
    
Tittar man i filen "appModules/message/messageModel.js" så ser man att SQL satserna använder syntaxen "db.run("INSERT INTO message (message, userID) VALUES (?, ?)", [message, userID]" vilket innebär att inkommande värderna "saneras" i samband med då värderna ifrån arrayen flytas till frågetecknen i sql satsen.
    
Genom att göra på detta sätt genomgående i hela applikationen så bör problemet åtgärdas.

### Problem 2: Lagring av känsligt data

#### Vad problemet innebär
När känslig data inte krypteras/hashas på ett korrekt sätt så är följderna av detta är katastrofala ifall någon obehörig skulle få åtkomst till dessa till exempel genom en SQL-injection attack.

#### Eventuella följder
Ett exempel är att inloggningsuppgifter kan användas till att logga in på andra hemsidor och tjänster där användaren har samma användarnamn och lösenord. Väldigt känsliga bilder skulle kunna användas för utpressning av attackeraren. Kreditkortsuppgifter och personnummer skulle kunna användas för e-handel.[4]

#### Identifierad ohashad känslig data i applikationen
I det här fallet är det extra illa eftersom lösenordet sparas i klartext och inte "hashas" till ett värde som inte går att gissa sig till av en attackerare. Hashning innebär att lösenordet översätts till ett större antal till synes slumpmässiga tecken som INTE går att översätta tillbaka till ursprungslösenordet. Ifall attackeraren nu får tag i inloggningsuppgifterna så kan han direkt börja använda dessa till att logga in på andra hemsidor/e-tjänster. Men ifall lösenordet hade varit hashat (med en säker metod) så hade attackeraren haft betydligt mindre nytta av dessa uppgifter.[5]

#### Hur problemet kan åtgärdas
Ett tips är att använda sig av följande [bibliotek](https://nodejs.org/api/crypto.html) för att kryptera lösenord. Viktigt att tänka på vid hashning är att använda sig av olika 32 eller 64 bitars "salt" för varje användares lösenord. Detta "salt" värde kan sparas i ett separat fält intill databaslösenordet.  [5]

### Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker

#### Vad problemet innebär
Om data inte valideras korrekt, mer specifikt att javascript, iframe-taggar, html-element med src attribut (andra kodspråk och taggar kan förekomma) inte filtreras/bearbetas i innehållet som postas från klient till server uppstår denna säkerhetsrisk. När innehåller sedan visas i klienters webbläsare så körs den tidigare postade javascript-koden hos klienten och kan då exempelvis stjäla klientens sessionskaka och vidarebefordra denna till attackeraren. I stora drag har attackeraren kontroll över det mesta som presenteras för och som finns lagrat hos klienten gällande den aktuella webbsidan. [6]

#### Eventuella följder
Eftersom attackeraren har kontroll över allt som visas på den kapade hemsidan hos klientens webbläsare så kan användaren enkelt luras till att exempelvis att ange känsliga uppgifter (kreditkort, personnummer, mm). Men det allvarligaste är nog att sessionen enkelt kan stjälas av attackeraren och detta innebär att attackeraren blir "inloggad" på hemsidan/applikationen som användaren utan att kunna användarens uppgifter.[7]

#### Identifierade problem i applikationen
Applikationen bearbetar/filtrerar inte bort javascript eller andra känsliga html taggar ifrån meddelandetexten som postas in. 

#### Hur problemet kan åtgärdas
Det enklaste sättet att skydda sig är att filtrera det postade innehållet och tillämpa whitelists för tillåtna tecken. Annars rekommenderas ett bibliotek för att tillämpa detta då det kan vara svårt att missa alla källor där detta kan uppstå. Organisationen OWASP har en bra källa med regler att tänka på om man väljer att göra detta själv [8]. Sammanfattningsvis är regeln att inte placera opålitlig data inom någon html-tags taggar (attributen), och att alltid filtrera datat som finns emellan start- och sluttaggen. 
   
[Här](https://github.com/chriso/validator.js) är ett förslag på ett bibliotek som skulle vara ett alternativ för projektet

### Problem 4: Säkerhetskontroll för funktioner saknas

#### Vad problemet innebär
Attackerare som känner till eller som kan gissa sig till adresser gömda adresser i systemet kan att utföra funktioner eller metoder som de inte ska ha rätt till egentligen. Detta är möjligt eftersom det inte finns några rättighetskontroller på funktionerna/metoderna.

#### Eventuella följder
Följderna är att objekt av olika slag kan skapas, ändras eller tas bort eller mer avancerade operationer kan utföras i applikationen som attackeraren inte ska ha rätt till. Exempelvis skulle användare skulle kunnas tas bort, eller i den här applikationens fall: att meddelanden kan tas bort bara genom att känna till adressen och de rätta post-parametrarna.[9]

#### Identifierade problem i applikationen
I applikationen går att att gå in på index-sidan direkt utan att behöva logga in. Detta genom att helt enkelt ange adressen i webbläsarens adressfält. Antagligen ska detta endast vara möjligt för de inloggade användarna.   
   
Ett annat betydligt större problem är det går att ta bort meddelanden utan någon som helst inloggning, bara genom att känna till adressen och parametrarna. Exempelvis: Om en POST med värdet (messageID = 3) skickas till http://localhost:3000/message/delete så tas meddelandet med id-numret 3 bort oberoende om man är inloggad eller inte, eller vem man är inloggad som.

Säkerhetskontroll saknas även (check om användaren är inloggad) för att hämta meddelanden. Eftersom det i template filen default.html körs javascript som anropar metoden MessageBoard.getMessages så hämtas meddelanderna till klientens webbläsare när denne anländer till startsidan, utan att ens ha loggat in.

#### Hur problemet kan åtgärdas

En säkerhetskontroll, fördelaktigen efter principen ACL behöver tillämpas. OWASP har en generell guide kring autentisering som kan vara bra att ta del av [10].

### Problem 5: Inget skydd för "Cross-Site Request Forgery" (CSRF)

#### Vad problemet innebär

Det är möjligt för andra hemsidor, webb-applikationer och andra illasinnade källor att anropa funktionalitet i systemet om en användare är inloggad i messy-labbage applikationen. 

#### Eventuella följder

Användare kan, utan att de är medvetna om det och utan medgivande manipulera messy-labbage applikationen innehåll genom att surfa in på andra hemsidor. De illasinnade applikationerna anropar helt enkelt exempelvis http://localhost:3000/message med en POST med ett nytt meddelande genom javascript i användarens webbläsare i samband med att användaren besöker den illasinnade webbplatsen. I och med att användaren fortfarande är inloggad i systemet så går detta utan problem.

#### Identifierade problem i applikationen

I applikationen finns det inget skydd mot CSRF alls på någon sida.

#### Hur problemet kan åtgärdas

Genom att tillämpa Synchronizer Token Pattern och generera en slumpmässig sträng (token), placera den i ett gömt formulärfält för varje POST request, som servern sedan validerar kan detta problem lösas. De illasinnade hemsidorna/källorna kan omöjligt gissa sig till det slumpmässiga strängarna förutsatt att detta är korrekt implementerat. [11]


## Prestandaproblem (Front end)

### Problem 6: Referenser till javascriptfiler script i sidhuvudet

#### Vad problemet innebär

När det finns referenser till externa scriptfiler i sidhuvudet så är det ett prestantaproblem då renderingen av sidan och hämtningen av andra resurser stannar tills webbläsaren har hämtat dessa javascriptfiler. Först när detta är färdigt hämtas resterande resurser och sidan renderas.[12]

#### Eventuella följder

För hemsidebesökare är sidan helvit utan innehåll tills scripten har hämtats och laddats av webbläsaren, först då får klienter en visuell bekräftelse på att sidan över huvud taget laddar. Klienter med dåliga uppkopplingar (mobiltelefoner) upplever detta värst.[]

#### Identifierade problem i applikationen

I filen appModules/siteViews/layouts/partials/head.html så finns det script taggar som refererar till externa javascriptfiler

#### Hur problemet kan åtgärdas

Genom att placera scriptreferenserna i html dokumentets slut så undviks detta problem. Då hämtas och laddas javascripten in först när html dokumentet laddats in och användaren blivit bemött av DOM:en och CSSOM:en[12]

### Problem 7: Onödiga referenser till filer som saknas eller inte används.

#### Vad problemet innebär

När der finns referenser till filer som saknas eller inte används så resulterar det i onödiga HTTP anrop till servern som belastar både servern och klienter. [13]

#### Eventuella följder

När klienters webbläsare försöker ladda ner dessa icke existerande samt onödiga filer så förhindras övriga resurser att laddas ner. Detta gör att sidan renderas onödigt sakta, speciellt då javascript referenserna finns i html-dokumentets sidhuvud.

#### Identifierade problem i applikationen

I filerna appModules/siteViews/layouts/partials/head.html och appModules/login/views/index.html finns referenser till filer som inte existerar.
    
Det finns även filer som laddas in o inödan: Stylesheet referensen till "//fonts.googleapis.com/icon?family=Material+Icons" verkar heller inte användas någonstans i dokumentet, vilket gör hämtning av denna fil helt onödig.
    
På start/login-sidan så laddas dessa två javascript in i onödan eftersom de inte används där: "/static/javascript/Message.js", "/static/javascript/MessageBoard.js".
    
Bakgrundsbilden "/static/images/b.jpg" används inte visuellt på sidan och laddas in i onödan.
    
När man är inloggad laddas filen "/static/css/signin.css" i onödan eftesom den bara används på startsidan.
    
CSS filen "/static/css/bootstrap.css" innehåller månaga onödiga stildefinitioner som inte används.

#### Hur problemet kan åtgärdas

Ta bort referenserna till de icke existerande dokumenten och samt de som laddas in i onödan. Använd inte det gemensamma sidhuvudet i start/login-sidan. Ta bort onödiga stilar definierade i Bootstrap.css filen.

### Problem 8: Onödigt stora javascriptfiler

#### Vad problemet innebär

Det finns javascript-filer som är onödigt stora eftersom att de innehåller onödigt kod som inte används samt att de inte är minifierade. [14]

#### Eventuella följder

Sidan renderas onödigt sakta för klienterna.

#### Identifierade problem i applikationen

I applikationen används en onödigt stor version av jquery som inte är minifierad. Denna version har en hel del kod för jquery moduler som inte används i applikationen. Den enda modulen som egentligen används är ajax-modulen.

#### Hur problemet kan åtgärdas

Förminska och minifiera javascript.

Kika [här](https://github.com/jquery/jquery#how-to-build-your-own-jquery) för att se hur det går att bygga ett minifierat jquery bibliotek med bara ajax-modulen.

### <a name="p9"></a> Problem 9: Felplacerad inline kod (javascript och css)

#### Vad problemet innebär

Inline- javascript och css går förvisso snabbare för webbläsaren att läsa in första gången som sidan laddas in, tappar samtidigt möjligheten att bli cache:at resterande gånger som sidan laddas in.

#### Eventuella följder

Inline-kod cachas inte och detta gör i längden att sidan tar längre tid att hämta för klienten. Det är generellt bättre att placera css- och javascript kod i externa filer, då chansen är större att sidan laddar snabbare med hjälp av att webbläsaren cachar dessa filer (förutsatt att webbservern stöjder detta). [15]

#### Identifierade problem i applikationen

Filen appModules/siteViews/layouts/default.html innehåller javascript-kod som bättre hör hemma i MessageBoard.js
Filerna appModules/message/views/index.html och appModules/siteViews/layouts/default.html innehåller css kod som (för cachningens skull) kan ligga i en egen css fil.

#### Hur problemet kan åtgärdas

Flytta inline javascriptkoden ifrån default.html till MessageBoard.js istället.
Flytta inline csskoden ifrån index.html till en egen css fil och länka denna i sidhuvudet (head.html).

## Egna övergripande reflektioner

HTTPD/2 knackar på dörren, minifiering onödig.

Enda autentiseringschecken finns när man kommer till index. Annars så har vem som helst rätt att göra vad som helst i applikationen.
  
Mycket kod är oimplementerat. Till exempel radera meddelanden. Backend funktionaliteten finns där, och även frontend funktionalitet. Men det är inte synsligt i gränsnittet.



## Tips

## Referenser

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

/ Johnny Pesola
