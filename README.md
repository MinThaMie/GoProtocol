# GoProtocol
This repo contains the communication protocol and rules for the Nedap University Go Project (2016-2018)

## Protocol
Het protocol is in te delen in drie fases:
- Opstarten 
- Spel 
- Afsluiten 
 
Deze worden verdeeld in: 
- Verantwoordelijkheden client 
- Verantwoordelijkheden server 
 
We beschouwen de server we als perfect en de waarheid, de clients mogen hiervan uitgaan. 
 
Nog af te spreken na deze sessie: 
- [ ] Wanneer kicken we een speler? Hoeveel invalid moves mag diegene uitvoeren? 
- [ ] Wat doen we als een keyword binnenkomt dat niet is afgesproken? 

---

### Algemeen
#### Er is één interface voor een server 
Iedereen maakt zijn eigen implementatie van de server die de interface implementeert. 
- [ ] Maken van een server interface
 
#### Communicatie tussen server en client
Alle communicatie tussen server en clients gebeurt d.m.v. een commando (keyword) in hoofdletters gevolgd door een spatie. Dit keyword staat altijd aan het begin van de input. Evt. argumenten volgen gescheiden door spaties. Deze argumenten zijn altijd geschreven in kleine letters. Messages (Strings) mogen wel hoofdletters bevatten, zoals bij `CHAT` en `INVALID`. Het einde van de communicatie is een new line character: \n. 
 
#### Ongeldige keywords
TBA 
#### Wanneer kicken? 
TBA

---

### Opstarten
Een client wil een spel starten, de server is in een staat waarin hij toegankelijk is voor clients om zich aan te bieden voor een spel. 
 
Keyword: `GO`
Argumenten: 
- name: alleen kleine letters en aan elkaar, naam.length() <= 20 
- boardsize: een oneven integer: 9 <= boardsize <= 131 && boardsize % 2 != 0 
Voorbeeld: `GO zangerrinus 9\n` 
Betekent: Ik wil Go spelen op een 9x9 bord en ik heet zangerrinus. 
 
#### Wachten 
De client en server wachten op een tweede speler. De client kan de verbinding verbreken om te annuleren, de server moet zijn verloren verbinding goed afhandelen. 
Server beantwoordt het verzoek aan de client door het sturen van het keyword `WAITING`. De client kan de verbinding verbreken door het keyword `CANCEL` te sturen. 
 
Keyword: `WAITING` 
Argumenten: geen 
Voorbeeld: `WAITING\n`
Keyword: `CANCEL`
Argumenten: geen
Voorbeeld: `CANCEL\n`
 
Zodra er een tweede client is waartegen gespeeld kan worden, wordt dit door de server aan de clients gecommuniceerd. De server kent elke speler een kleur toe. 
 
Keyword: `READY` 
Argumenten: 
- color: kleur van de clientspeler
-- black: zwart 
-- white: wit 
- opponentname: naam van de tegenstander, deze heeft impliciet de andere kleur dan die gecommuniceerd is met het READY keyword .
Voorbeeld: `READY black barrybadpak\n`
Betekent: Je kunt nu Go spelen tegen barrybadpak en jij hebt kleur zwart.

---

### Spel
Zowel de server als de client kent de regels en past deze toe. De server handhaaft de regels en kan ze toeleggen op de client, daarnaast houdt de server de score bij. De regels en score van de server zijn **doorslaggevend**. De client doet een zet, checkt of deze volgens zijn eigen regels geldig is en stuurt deze naar de server. De server bepaalt of dit inderdaad een geldige zet is en communiceert dit aan beide clienten zodat deze de zet kunnen verwerken. Mogelijkheden van zetten zijn: een steen zetten, passen of opgeven. 
 
#### Geldige zet 
Keyword: `MOVE` 
Argumenten: 
- color: kleur van de speler die de zet doet 
- x: x-coördinaat van zet 
- y: y-coördinaat van zet 
Voorbeeld: `MOVE black 13 14\n`
Betekent: Zwart zet een steen op coordinaat 13,14
 
De oriëntatie van x en y zijn niet afgesproken, maar worden geïmplementeerd door de client. Het bord kan er voor iedereen dus anders uitzien, maar (4,5) is bij iedereen natuurlijk wel (4,5). 
 
Indien geldig (gecontroleerd door server), stuurt server aan alle clients een bevestiging. 
 
Keyword: `VALID` 
Argumenten: zie `Client.MOVE` 
Voorbeeld: `VALID black 13 14\n` 
 
#### Ongeldige zet 
Een speler doet een ongeldige zet. Er zijn allerlei redenen voor ongeldigheid, maar een voorbeeld is een negatieve coördinaat in een MOVE. 
 
Server stuurt `INVALID` aan de client die ongeldige zet deed. 
Keyword: `INVALID` 
Argumenten:
- message: een String met de fouttekst / reden van ongeldigheid, mag spaties bevatten, maar geen new line character 
Voorbeeld: `INVALID cannot place stone: negative coordinate\n`  
 
#### Passen 
De client kan passen om geen zet te doen. 
Keyword: `PASS` 
Argumenten: geen 
 
De server bevestigt dit aan alle clients. 
Keyword: `PASSED` 
Argumenten: 
- color [black/white]: kleur van de speler die gepast heeft 
Voorbeeld: `PASSED white\n` 
 
#### Opgeven 
Een clientspeler kan opgeven. 
Keyword: `TABLEFLIP` 
Argumenten: geen 
 
De server bevestigt dit aan alle clients. 
Keyword: `TABLEFLIPPED` 
Argumenten:  
- color [black/white]: kleur van de speler die gepast heeft 
Voorbeeld: `TABLEFLIPPED black\n` 

#### Chat 
Een chatbericht kan door zowel clients als servers gestuurd worden. 
Keyword: `CHAT`
Argumenten: 
- name: naam van de speler die zendt, ter verificatie wie de afzender is 
- message: het bericht, mag spaties bevatten, maar geen new line character 
Voorbeeld: `CHAT barrybadpak Waar ga je heen met je kamelenteen?\n`

---

### Afsluiten
Omdat de server weet wanneer het spel is afgelopen, kondigt de server dit aan bij alle clients o.v.v. de winnende kleur en de scores. 
 
Keyword: `END` 
Argumenten: 
- winnercolour: kleur van de winnaar 
- blackscore: score van zwart, een double 
- whitescore: score van wit, een double 
Voorbeeld: `END white 16.0 17.5\n` 
