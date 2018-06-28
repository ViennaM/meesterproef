# meesterproef

Samen met [Koos](https://github.com/hackshackshacks) heb ik gewerkt aan het bugregistratiesysteem voor multimedia tours in het Van Gogh Museum. Het systeem bestaat uit een registratie gedeelte voor de baliemedewerkers en een dashboard voor managers.

## Leerdoelen

### D3 gebruiken om een bruikbare visualisatie te maken van data
Mijn eerste leerdoel komt voort uit een ontwerpkeuze van het dashboard, waarbij we het percentage van bugs inzichtelijk willen maken met een grafiek. In de grafiek moet duidelijk zijn hoeveel bugs en sales er zijn geregistreerd en welk percentage aan bugs daaruit komt. Docent Titus Wormer heeft mij aangeraden om [C3](https://c3js.org/) (gebaseerd op D3) te gebruiken hiervoor, omdat deze library speciaal voor grafieken is. Onderstaande grafiek is de grafiek in het ontwerp.

![](https://i.imgur.com/LBteULm.png)

De voorbeelden op [https://c3js.org/examples.html](https://c3js.org/examples.html) hebben mij op weg geholpen om een basis te maken van de grafiek. Vervolgens heb ik verder gebouwd op deze basis om extra data in de tooltip te krijgen en de CSS te stylen.

#### Tooltip data
Om extra data toe te voegen aan de tooltip van de grafiek, heb ik extra columns moeten toevoegen. 
```javascript
columns: [
  this.chartDateData, 
  this.chartPercentageData,
  this.chartSalesData, 
  this.chartBugData
]
```
Dit zorgde er echter voor dat er twee extra lijnen in de grafiek kwamen. Deze heb ik met CSS verwijderd.
```css
.c3-target-Sales,
.c3-target-Bugs {
  display: none;
}
```
De nummers in de y-as zijn nog wel gebaseerd op de verwijderde lijnen en in verhouding te hoog. Het maximale aantal in y-as moet worden gebaseerd op het percentage die wel wordt weergegeven.
```javascript
let newArr = new Array(...this.chartPercentageData)
newArr.shift()
return Math.max.apply(Math, newArr)
```

Hiermee heb ik toch ongeveer bereikt wat ik heb ontworpen. Onderstaande afbeelding is het eindresultaat van de grafiek.
![](https://i.imgur.com/33N7PjI.png)

### Single page web app toepassen
Omdat mijn opdracht van het vak Web App From Scratch wat basic was, wilde ik de kennis die ik bij dit vak heb opgedaan toepassen op dit project om te oefenen. We hebben van zowel het registratie gedeelte als de dashboard een single page web app gemaakt. Met behulp van hash router [Routie](http://projects.jga.me/routie/) handel ik de routes af van de app. 

```javascript
routie({
  recent: () => {
    search.reset()
    helper.replaceHTML(bugs.elements.title, "Recente bugs")
    bugs.getData("recent")
    this.toggle("recent", "register", true)
  },
  all: () => {
    search.reset()
    helper.replaceHTML(bugs.elements.title, "Alle bugs")
    bugs.getData("all")
    this.toggle("all", "register", true)
  },
  // etc... 
```
Een probleem waar we met Routie tegenaan liepen was dat de subnavigatie anchor links met hash als href werden overschreven. Dit heb ik opgelost door een functie te schrijven die ervoor zorgt dat je alsnog scrollt naar een element wanneer je klikt op een subnavigatie item.
```javascript
this.elements.subNav.forEach((item, i) => {
  item.addEventListener("click", () => {
    if (pos.length === 0) {
      this.elements.categories.forEach((category, j) => {
        pos.push(this.elements.categories[j].getBoundingClientRect().top)
      })
      this.elements.mainElement.scrollTop = this.elements.categories[i].getBoundingClientRect().top - 100
    }
    this.elements.mainElement.scrollTop = pos[i] - 100
  })
})
```

Daarnaast heb ik ook kunnen oefenen met het object literal pattern.

## Aandeel

### Ontwerp
Na een lange periode van prototypen, heb ik de HiFi wireframes omgezet naar een ontwerp die we over de hele app hebben kunnen aanhouden. Dit ontwerp was voor ons niet alleen een goede houvast, maar ook de opdrachtgevers wisten wat ze konden verwachten. Aangezien het ontwerp in Adobe XD is gemaakt, kon ik makkelijk een klikbaar prototype maken. Dit prototype was nuttig om te testen bij de eindgebruikers. Hier hebben we enorm veel feedback uit gekregen die belangrijk was voor de realisatie van het product.
<details><summary>Screenshot van artboards</summary>
  <img src="https://i.imgur.com/62Fl7a3.png">
</details>

### Database structuur
Voordat we zijn gaan bouwen ben ik eerst na gaan denken over hoe de database eruit zou moeten zien. Tijdens het wireframen en ontwerpen hebben we al goed nagedacht over welke data we nodig hadden. Ik ben op basis van wireframes en designs een database structuur gaan opschrijven. Dit deed ik op de manier waarop we het hebben geleerd bij project Tech, waar we met MySQL databases werkten.
![](https://i.imgur.com/kjX7XHC.jpg)
Aangezien we dit project met MongoDB hebben gewerkt, is de database structuur een heel stuk simpeler geworden dan van te voren bedacht. Dankzij gebruik van [MongoDB](https://www.mongodb.com/) hebben we veel queries met JavaScript kunnen doen, waar evenveel (en misschien wel meer) mogelijkheden mee zijn als met SQL.
![](https://i.imgur.com/HFEDxZs.png)

### Modals
Naast de basisfunctionaliteit van het registreren van een een bug (eventueel met device id), was het volgens de opdrachtgever ook wenselijk om bugs toe te kunnen voegen. Ook bleek het kiezen van een balie een belangrijke functie om bugs goed te registreren en het invoeren van sales om het bugpercentage te kunnen bepalen.
Voor deze extra functies heb ik een modal ontworpen en uitgewerkt die herbruikbaar is voor nieuwe modals. De volgende module is eruit voort gekomen:

![](https://i.imgur.com/cjM96LU.png)

Een voorbeeld van een herbruikbare functie voor elke modal is de validate method:

```javascript
validate: function () {
  document.querySelectorAll(".modal").forEach((modal, i) => {
    let requiredFields = modal.querySelectorAll("[data-required='true']")
    requiredFields.forEach(input => {
      input.addEventListener("input", () => {
        let count = 0
        requiredFields.forEach(field => {
          if (field.value !== "") {
            count++
          }
        })
        if (count === requiredFields.length) {
          this.elements.submitButtons[i].disabled = false
        } else {
          this.elements.submitButtons[i].disabled = true
        }
      })
    })
  })
}
```

Deze functie zorgt ervoor dat alle form inputs met data attribuut `required="true"` geteld worden en de submit button pas beschikbaar wordt als alle inputs zijn ingevuld.

## Vakken

### Web design
Tijdens dit project hebben we veel rekening gehouden met de eindgebruiker van het product. We hebben meerdere iteraties gedaan op de prototypes, door vaak te testen. We hebben in de [design brief](https://docs.google.com/document/d/1cUIhdL20bqOLhVn5q1584gELlEXwzi_nU4ESXO8bdsc/edit?usp=sharing) de user scenario's uitgebreid beschreven, om zo veel mogelijk doelgericht te werk te gaan.
In onze procesboeken staan de bevindingen van tests beschreven.

### Performance matters
Aangezien onze app op Node.js draait, hebben we onze API calls zo veel mogelijk serverside gedaan, zodat de rekenkracht niet in de browser gebeurt. Ik heb nog een service worker willen toevoegen, maar helaas was hier geen tijd meer voor. Service worker staat wel als to do in de wishlist.

### CSS To The Rescue
We hebben [PostCSS](https://postcss.org/) gebruikt compilen van CSS. Dit bood ons de mogelijkheid om CSS variablen te gebruiken in onze code. Daarnaast hebben we consequent relative eenheden gebruikt voor afmetingen en font-sizes. Tijdens de minor heb ik gemerkt dat CSS niet zo makkelijk is als gedacht. Ik heb hier voor Weekly Nerd een artikel over geschreven. Bekijk deze [hier](https://github.com/ViennaM/weekly-nerd).

### Web App From Scratch
Zoals beschreven in mijn leerdoel hebben we een Single Page Web App gemaakt. We hebben hier vooraf nog wel over getwijfeld, maar kwamen tot de conclusie dat ons product het best een single page app kon zijn. De voordelen heb ik ook beschreven in een artikel voor Weekly Nerd. Bekijk deze [hier](https://github.com/ViennaM/weekly-nerd). Naast Routie, hebben we het object literal pattern toegepast.
