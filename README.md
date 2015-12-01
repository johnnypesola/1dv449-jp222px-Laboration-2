# Problem - Messy Labbage (Johnny Pesola / jp222px)

## Säkerhetsproblem

## Okatogoriserat

Går att posta nya meddelanden utan att vara inloggad: Nej
Går att ta bort meddelanden utan att vara inloggad: Ja


Analysen av säkerhetsproblemen i applikationen är starkt influerad av organisationen OWASP:s top 10 lista över säkerhetshål. Denna lista kan hittas [här](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)

### Problem 1 (A1): SQL injections - Ej önskvärd databasåtkomst och manipulering.

#### Vad problemet innebär
Säkerhetshålet innebär kort och gott att attackeraren skriver sitt postdata på ett sådant sätt så att ej önskvärd databasåtkomst och manipulering av datat i databasen blir möjlig.

#### Eventuella följder
Följderna av detta är katastrofala. Användarnas lagrade användaruppgifter läcker ut och kan användas genom att logga in med dessa på andra hemsidor och tjänster. I det här fallet är det extra illa eftersom lösenordet sparas i klartext och inte "hashas" till ett värde som inte går att gissa sig till av en attackerare. Attackeraren kan också genom manipulera datat i databastabellen efter egna önskemål (till exempel byta lösenord på administratören), eller ta bort allt data. [55]

#### Identifierade SQL injections i applikationen
När en inloggning sker så finns det i applikationen en sårbarhet för denna typ av attack. I källkoden i filen "appModules/login/lib/login.js" så konkateneras strängarna till en enda sql sats utan att de inkommande värderna kontrolleras eller saneras. Det här är en väldigt allvarlig typ av attack och bör undvikas genom att använda tekniken för databindning som används vid skapande och borttagning av meddelanden. Mer om detta nedan. [55]

Applikationen har testats med att utsättas för dessa typer av attacker vid skapande och borttagning av meddelanden och för dessa metoder hittades inga sårbarheter för denna typ av attack.

#### Hur problemen kan åtgärdas
Organisationen OWASP har på sin hemsida en allmän bra guide för att undvika detta [87], men just i det här fallet så är den rekommenderade fallet att använda sig av sqlite3 bibliotekets dokumenterade metoder [42].

Tittar man i filen "appModules/message/messageModel.js" så ser man att SQL satserna använder syntaxen "db.run("INSERT INTO message (message, userID) VALUES (?, ?)", [message, userID]" vilket innebär att inkommande värderna "saneras" i samband med då värderna ifrån arrayen flytas till frågetecknen i sql satsen.
   
Genom att göra på detta sätt genomgående i hela applikationen så bör problemet åtgärdas.

### Problem 2 (A): Autentisering och sessions säkerhet (Lagring av känsligt data)

### Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker

### Problem 4: Osäkra objektreferenser

### Problem 5 (A7): Säkerhetskontroll för funktioner saknas

Det går att se alla meddelanden fast man inte är inloggad.
   
(Osäkert) Det finns en sårbarhet där det går att ta bort meddelanden utan att vara inloggad som administratör.   
Exempel: Om en POST med värdet (messageID = 3) skickas till http://localhost:3000/message/delete så tas meddelandet bort oberoende om man är inloggad eller inte, eller vem man är inloggad som.
   
En säkerhetskontroll, fördelaktigen efter principen ACL behöver tillämpas. OWASP har en generell guide kring autentisering som kan vara bra att ta del av [57].


## Prestandaproblem (Front end)

## Egna övergripande reflektioner

Enda autentiseringschecken finns när man kommer till index. Annars så har vem som helst rätt att göra vad som helst i applikationen.
  
Mycket kod är oimplementerat. Till exempel radera meddelanden. Backend funktionaliteten finns där, och även frontend funktionalitet. Men det är inte synsligt i gränsnittet.


## Tips

## Referenser
[87] The Open Web Application Security Project, "SQL Injection Prevention Cheat Sheet", OWASP, November 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet [Hämtad: 12 november, 2015].

[55] The Open Web Application Security Project, "Top 10 2013-A1-Injection", OWASP, Juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Hämtad: 12 november, 2015].
   
[42] Mapbox, "Github - sqlite3 API Wiki", Github, November 2015 [Online] Tillgänglig: https://github.com/mapbox/node-sqlite3/wiki/API [Hämtad: 12 nobemver, 2015]
   
[57] The Open Web Application Security Project, "Guide to Authorization," OWASP, Maj 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_Authorization [Hämtad: 12 november, 2015].



/ Johnny Pesola
