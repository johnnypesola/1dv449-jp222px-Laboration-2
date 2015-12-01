# Problem - Messy Labbage (Johnny Pesola / jp222px)

## Säkerhetsproblem

Analysen av säkerhetsproblemen i applikationen är starkt influerad av organisationen OWASP:s top 10 lista över säkerhetshål. Denna lista kan hittas [här](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)

### Problem 1 (A1): SQL injections



### Problem 2 (A): Autentisering och sessions säkerhet (Lagring av känsligt data)

### Problem 3: Inget skydd för Cross Site Scripting (XSS) Attacker

### Problem 4: Osäkra objektreferenser

### Problem 5 (A7): Säkerhetskontroll för funktioner saknas

Går att ta bort object, meddelanden i det här fallet utan att vara inloggad som administratör.


## Prestandaproblem (Front end)

## Egna övergripande reflektioner

Enda autentiseringschecken finns när man kommer till index. Annars så har vem som helst rätt att göra vad som helst i applikationen.

Mycket kod är oimplementerat. Till exempel radera meddelanden. Backend funktionaliteten finns där, och även frontend funktionalitet. Men det är inte synsligt i gränsnittet.


## Tips

/ Johnny Pesola
