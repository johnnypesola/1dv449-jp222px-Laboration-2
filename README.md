# Problem - Messy Labbage (Johnny Pesola / jp222px)

## Säkerhetsproblem

## Okatogoriserat

Går att posta nya meddelanden utan att vara inloggad: Nej
Går att ta bort meddelanden utan att vara inloggad: Ja


Analysen av säkerhetsproblemen i applikationen är starkt influerad av organisationen OWASP:s top 10 lista över säkerhetshål. Denna lista kan hittas [här](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)

### Problem 1 (A1): SQL injections

Applikationen har testats med att utsättas för sql attacker vid skapande och borttagning av meddelanden och det verkar inte finnas sårbarheter för denna typ av attack.   
Eftersom SQL satserna använder syntaxen "db.run("INSERT INTO message (message, userID) VALUES (?, ?)", [message, userID]" så escapas de inkommande värderna i arrayen när de placeras vid frågetecknen i själva sql satsen.[42]




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

[42] Mapbox, "Github - sqlite3 API Wiki", Github, November 2015 [Online] Tillgänglig: https://github.com/mapbox/node-sqlite3/wiki/API [Hämtad: 12 nobemver, 2015]
[57] The Open Web Application Security Project, "Guide to Authorization," OWASP, Maj 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Guide_to_Authorization. [Hämtad: 12 november, 2015].



/ Johnny Pesola
