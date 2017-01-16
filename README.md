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
- [x] Wanneer kicken we een speler? Hoeveel invalid moves mag diegene uitvoeren? 
- [x] Wat doen we als een keyword binnenkomt dat niet is afgesproken? 

---

### Algemeen
#### Er is één interface voor een server 
Iedereen maakt zijn eigen implementatie van de server die de interface implementeert. 
- [x] Maken van een server interface
> Dit gaan we niet doen.
 
#### Communicatie tussen server en client
Alle communicatie tussen server en clients gebeurt d.m.v. een commando (keyword) in hoofdletters gevolgd door een spatie. Dit keyword staat altijd aan het begin van de input. Evt. argumenten volgen gescheiden door spaties. Deze argumenten zijn altijd geschreven in kleine letters. Messages (Strings) mogen wel hoofdletters bevatten, zoals bij `CHAT` en `INVALID`. Het einde van de communicatie is een new line character: \n. 
 
#### Ongeldige keywords
Deze mogen genegeerd worden. 

> Je kunt niet iemand kicken op een 'unknown' keyword (anders kick je bijv. iemand voor het chatten). De server nergeert ze.
De server mag terugcommuniceren dit ie het keyword niet begrijpt. BV `WARNING keyword CHAT unknown`.

#### Wanneer kicken? 
> Een speler (client) wordt gekicked (van de server) als deze een invalid move stuurt. Je kunt dit namelijk opvangen in je client.


---

### Opstarten
Een client wil een spel starten, de server is in een staat waarin hij toegankelijk is voor clients om zich aan te bieden voor een spel. 
 
Keyword: `GO`
Argumenten: 
- name: alleen kleine letters en aan elkaar, naam.length() <= 20 
- boardsize: een oneven integer: 5 <= boardsize <= 131 && boardsize % 2 != 0 
- (optional) otherName: naam van de tegenstander, alleen kleine letters en aan elkaar, naam.length() <= 20)

Voorbeeld: `GO zangerrinus 9\n` 
Betekent: Ik wil Go spelen op een 9x9 bord en ik heet zangerrinus. 

> Als er meerdere clients zijn, die een verschillende bordgrootte requesten mag de server verschillende dingen doen.
Zoals bijv. twee mensen met verschillende grootte aan elkaar koppelen (de server geeft aan welke bordgrootte gekozen is, zie READY), of meerdere spellen klaar zetten en wachten.
 
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
- opponentname: naam van de tegenstander, deze heeft impliciet de andere kleur dan die gecommuniceerd is met het READY keyword 
- boardsize: grootte van het bord

Voorbeeld: `READY black barrybadpak 9\n`
Betekent: Je kunt nu Go spelen tegen barrybadpak en jij hebt kleur zwart.

---

### Spel
Zowel de server als de client kent de regels en past deze toe. De server handhaaft de regels en kan ze toeleggen op de client, daarnaast houdt de server de score bij. De regels en score van de server zijn **doorslaggevend**. De client doet een zet, checkt of deze volgens zijn eigen regels geldig is en stuurt deze naar de server. De server bepaalt of dit inderdaad een geldige zet is en communiceert dit aan beide clienten zodat deze de zet kunnen verwerken. Mogelijkheden van zetten zijn: een steen zetten, passen of opgeven. 
 
#### Geldige zet 
Keyword: `MOVE` 
Argumenten: 
- x: x-coördinaat van zet 
- y: y-coördinaat van zet 

Voorbeeld: `MOVE 13 14\n`
Betekent: De huidige speler zet een steen op coordinaat 13,14

> We hebben er voor gekozen om de kleur niet mee te sturen, aangezien dit vals spelen bevorderd. Aka de server moet dan sowieso checken welke kleur een speler heeft (dus dan heeft het meesturen geen zin).
 
De oriëntatie van x en y zijn niet afgesproken, maar worden geïmplementeerd door de client. Het bord kan er voor iedereen dus anders uitzien, maar (4,5) is bij iedereen natuurlijk wel (4,5). 
> Het is wel handig om de x,y orientatie van de GUI aan te houden.
 
Indien geldig (gecontroleerd door server), stuurt server aan alle clients een bevestiging. 
 
Keyword: `VALID` 
Argumenten: 
- color: [black/white] kleur van de steen die geplaatst is
- x: x-coördinaat van zet 
- y: y-coördinaat van zet 
Voorbeeld: `VALID black 13 14\n` 
 
#### Ongeldige zet 
Een speler doet een ongeldige zet. Er zijn allerlei redenen voor ongeldigheid, maar een voorbeeld is een negatieve coördinaat in een MOVE. 
 
Server stuurt `INVALID` aan de client die ongeldige zet deed **en kicked deze uit de game**.
Keyword: `INVALID` 
Argumenten:
- message: een String met de fouttekst / reden van ongeldigheid, mag spaties bevatten, maar geen new line character 

Voorbeeld: `INVALID cannot place stone: negative coordinate\n`  
 
#### Passen 
De client kan passen om geen zet te doen. 
Keyword: `PASS` 
Argumenten:
- color: [black/white] kleur van de speler die past
 
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
- color [black/white]: kleur van de speler die opgeeft heeft 

Voorbeeld: `TABLEFLIPPED black\n` 

#### Chat 
Een chatbericht kan door zowel clients als servers gestuurd worden. 
Keyword: `CHAT`
Argumenten: 
- message: het bericht, mag spaties bevatten, maar geen new line character 

Voorbeeld: `CHAT Waar ga je heen met je kamelenteen?\n`

> Het toevoegen van de naam is overbodig, omdat de naam van de speler aan het begin van het spel al gegeven wordt. (En anders kunnen mensen vals spelen door te doen alsof iemand anders iets zegt).


#### Warning
De server kan een warning geven aan de client als hij een bepaald keyword niet herkend.
Keyword: `WARNING`
Argumenten: 
- message: de warning message.

Voorbeeld : `WARNING keyword CHAT unknown`

---

### Afsluiten
Omdat de server weet wanneer het spel is afgelopen, kondigt de server dit aan bij alle clients o.v.v. de winnende kleur en de scores. 
 
Keyword: `END` 
Argumenten: 
- blackscore: score van zwart, een double 
- whitescore: score van wit, een double 

Voorbeeld: `END 16.0 17.5\n` 

> De kleur was hier overbodig.

### Vragen aan coaches
- [x] Ko rule, multiple moves backward looking
Ja dit moet!
- [x] Wordt 0.5, 6.5, 7.5 of geen extra score toegevoegd aan wit? (Aka kan draw)
NEE. Er kan dus draw zijn.
- [x] Hoe wordt de area score berekend? Vooral als er stenen van de tegenstander in jouw gebied liggen?
Als er stenen in jou gebied liggen krijg je GEEN punten. aka je moet ze eerst slaan.
- [x] De openJML hoeft dus niet? Wordt Google Java Style aangeraden of die van Blackboard?
OpenJML hoeft NIET. Ze willen graag EEN checkstyle (welke je neemt mag je zelf weten, Google Java Style wordt aangeraden)
- [ ] Dorien: Wat te doen met score als iemand TABLEFLIPPED? 1. Geen score, 2. Bereken huidige score, 3. Geef andere speler het hele bord, 4. anders.
