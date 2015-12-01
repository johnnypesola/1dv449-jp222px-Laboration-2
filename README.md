# Problem - Messy Labbage (Johnny Pesola / jp222px)

## Säkerhetsproblem

## Okatogoriserat


Analysen av säkerhetsproblemen i applikationen är starkt influerad av organisationen OWASP:s top 10 lista över säkerhetshål. Denna lista kan hittas [här](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)

### Problem 1: SQL injections - Ej önskvärd databasåtkomst och manipulering av data.

#### Vad problemet innebär
Säkerhetshålet innebär kort och gott att attackeraren skriver sitt postdata på ett sådant sätt så att ej önskvärd databasåtkomst och manipulering av datat i databasen blir möjlig.

#### Eventuella följder
Följderna av detta är katastrofala. Attackeraren kan logga in som en administratör i systemet. Användarnas lagrade känsliga uppgifter kan bli tillgängliga för attackeraren (Se Problem 2 för mer info). Attackeraren kan också eventuellt manipulera datat i databastabellen efter egna önskemål (till exempel byta lösenord på administratören), eller ta bort allt data. [55]
   
#### Identifierade SQL injections i applikationen
När en inloggning sker så finns det i applikationen en sårbarhet för denna typ av attack. I källkoden i filen "appModules/login/lib/login.js" så konkateneras strängarna till en enda sql sats utan att de inkommande värderna kontrolleras eller saneras. Det här är en väldigt allvarlig typ av attack och bör undvikas genom att använda tekniken för databindning som används vid skapande och borttagning av meddelanden. Mer om detta nedan. [55]

Applikationen har testats med att utsättas för dessa typer av attacker vid skapande och borttagning av meddelanden och för dessa metoder hittades inga sårbarheter för denna typ av attack.
    
Det jag kontret lyckats göra med hjälp attacken i applikationen är att logga in med sql injections utan att känna till användarnamn eller lösenord, men jag har inte lyckats manipulera datat eller visa känslig data.

#### Hur problemen kan åtgärdas
Organisationen OWASP har på sin hemsida en allmän bra guide för att undvika detta [87], men just i det här fallet så är det rekommenderat att använda sig av sqlite3 bibliotekets dokumenterade metoder [42].
    
Tittar man i filen "appModules/message/messageModel.js" så ser man att SQL satserna använder syntaxen "db.run("INSERT INTO message (message, userID) VALUES (?, ?)", [message, userID]" vilket innebär att inkommande värderna "saneras" i samband med då värderna ifrån arrayen flytas till frågetecknen i sql satsen.
    
Genom att göra på detta sätt genomgående i hela applikationen så bör problemet åtgärdas.

### Problem 2 (A): Lagring av känsligt data

#### Vad problemet innebär
När känslig data inte krypteras/hashas på ett korrekt sätt så är följderna av detta är katastrofala ifall någon obehörig skulle få åtkomst till dessa till exempel genom en SQL-injection attack.

#### Eventuella följder
Ett exempel är att inloggningsuppgifter kan användas till att logga in på andra hemsidor och tjänster där användaren har samma användarnamn och lösenord. Väldigt känsliga bilder skulle kunna användas för utpressning av attackeraren. Kreditkortsuppgifter och personnummer skulle kunna användas för e-handel.[36]

#### Identifierad ohashad känslig data i applikationen
I det här fallet är det extra illa eftersom lösenordet sparas i klartext och inte "hashas" till ett värde som inte går att gissa sig till av en attackerare. Hashning innebär att lösenordet översätts till ett större antal till synes slumpmässiga tecken som INTE går att översätta tillbaka till ursprungslösenordet. Ifall användaren nu får tag i inloggningsuppgifterna så kan han direkt börja använda dessa till att logga in på andra hemsidor/e-tjänster. Men ifall lösenordet hade varit hashat (men en säker metod) så hade attackeraren inte haft någon större nytta av dessa uppgifter.[]

#### Hur problemet kan åtgärdas

### Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker

#### Vad problemet innebär
Om data inte valideras korrekt, mer specifikt att javascript, iframe-taggar, html-element med src attribut (andra kodspråk och taggar kan förekomma) inte filtreras/bearbetas i innehållet som postas från klient till server uppstår denna säkerhetsrisk. När innehåller sedan visas i klienters webbläsare så körs den tidigare postade javascript-koden hos klienten och kan då exempelvis stjäla klientens sessionskaka och vidarebefordra denna till attackeraren. I stora drag har attackeraren kontroll över det mesta som presenteras för och som finns lagrat hos klienten gällande den aktuella webbsidan. [81]

#### Eventuella följder
Eftersom attackeraren har kontroll över allt som visas på den kapade hemsidan hos klientens webbläsare så kan användaren enkelt luras till att exempelvis att ange känsliga uppgifter (kreditkort, personnummer, mm). Men det allvarligaste är nog att sessionen enkelt kan stjälas av attackeraren och detta innebär att attackeraren blir "inloggad" på hemsidan/applikationen som användaren utan att kunna användarens uppgifter.[]

#### Identifierade problem i applikationen
Applikationen bearbetar/filtrerar inte bort javascript eller andra känsliga html taggar ifrån meddelandetexten som postas in. 

#### Hur problemet kan åtgärdas
Det enklaste sättet att skydda sig är att filtrera det postade innehållet och tillämpa whitelists för tillåtna tecken. Annars rekommenderas ett bibliotek för att tillämpa detta då det kan vara svårt att missa alla källor där detta kan uppstå. Organisationen OWASP har en bra källa med regler att tänka på om man väljer att göra detta själv [73]. Sammanfattningsvis är regeln att inte placera opålitlig data inom någon html-tags taggar (attributen), och att alltid filtrera datat som finns emellan start- och sluttaggen. 
   
[Här](https://github.com/chriso/validator.js) är ett förslag på ett bibliotek som skulle vara ett alternativ för projektet

### Problem 4: Osäkra objektreferenser

### Problem 5: Säkerhetskontroll för funktioner saknas

#### Vad problemet innebär


#### Eventuella följder

#### Identifierade problem i applikationen

Går att posta nya meddelanden utan att vara inloggad: Nej
Går att ta bort meddelanden utan att vara inloggad: Ja

Det går att se alla meddelanden fast man inte är inloggad.

Det finns en sårbarhet där det går att ta bort meddelanden utan att vara inloggad som administratör.   
Exempel: Om en POST med värdet (messageID = 3) skickas till http://localhost:3000/message/delete så tas meddelandet bort oberoende om man är inloggad eller inte, eller vem man är inloggad som.


#### Hur problemet kan åtgärdas

En säkerhetskontroll, fördelaktigen efter principen ACL behöver tillämpas. OWASP har en generell guide kring autentisering som kan vara bra att ta del av [57].

## Prestandaproblem (Front end)

## Egna övergripande reflektioner

Enda autentiseringschecken finns när man kommer till index. Annars så har vem som helst rätt att göra vad som helst i applikationen.
  
Mycket kod är oimplementerat. Till exempel radera meddelanden. Backend funktionaliteten finns där, och även frontend funktionalitet. Men det är inte synsligt i gränsnittet.


## Tips

## Referenser

[73] The Open Web Application Security Project, "XSS (Cross Site Scripting) Prevention Cheat Sheet", OWASP, Septempber 2015, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet [Hämtad: 12 november, 2015].

[81] The Open Web Application Security Project, "Top 10 2013-A3-Cross-Site Scripting (XSS)", OWASP, Februari 2014, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS) [Hämtad: 12 november, 2015].

[36] The Open Web Application Security Project, "Top 10 2013-A6-Sensitive Data Exposure", OWASP, Juni 2013, [Online]  Tillgänglig: 
https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure [Hämtad: 12 november, 2015].

[87] The Open Web Application Security Project, "SQL Injection Prevention Cheat Sheet", OWASP, November 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet [Hämtad: 12 november, 2015].

[55] The Open Web Application Security Project, "Top 10 2013-A1-Injection", OWASP, Juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Hämtad: 12 november, 2015].
   
[42] Mapbox, "Github - sqlite3 API Wiki", Github, November 2015 [Online] Tillgänglig: https://github.com/mapbox/node-sqlite3/wiki/API [Hämtad: 12 nobemver, 2015]
   
[57] The Open Web Application Security Project, "Guide to Authorization," OWASP, Maj 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_Authorization [Hämtad: 12 november, 2015].



/ Johnny Pesola
