---
mainImage: ../../../images/part-3.svg
part: 3
letter: b
lang: fi
---

<div class="content">

Yhdistetään seuraavaksi [osassa 2](/osa2) tekemämme frontend omaan backendiimme. 

Edellisessä osassa backendinä toiminut JSON Server tarjosi muistiinpanojen listan osoitteessa http://localhost:3001/notes frontendin käyttöön. Backendimme urlien rakenne on hieman erilainen (muistiinpanot ovat osoitteessa http://localhost:3001/api/notes) eli muutetaan frontendin tiedostossa <i>src/services/notes.js</i> määriteltyä muuttujaa _baseUrl_ seuraavasti:

```js
import axios from 'axios'
const baseUrl = 'http://localhost:3001/api/notes' //highlight-line

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...

export default { getAll, create, update }
```

Frontendin tekemä GET-pyyntö osoitteeseen <http://localhost:3001/api/notes> ei jostain syystä toimi:

![](../../images/3/3ae.png)

Mistä on kyse? Backend toimii kuitenkin selaimesta ja postmanista käytettäessä ilman ongelmaa.

### Same origin policy ja CORS

Kyse on asiasta nimeltään CORS eli Cross-origin resource sharing. [Wikipedian](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) sanoin

> <i>Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos. Certain "cross-domain" requests, notably Ajax requests, are forbidden by default by the same-origin security policy.</i>

Lyhyesti sanottuna meidän kontekstissa kyse on seuraavasta: web-sovelluksen selaimessa suoritettava JavaScript-koodi saa oletusarvoisesti kommunikoida vain samassa [originissa](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) olevan palvelimen kanssa. Koska palvelin on localhostin portissa 3001 ja frontend localhostin portissa 3000, niiden origin ei ole sama.

Korostetaan vielä, että [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) ja CORS eivät ole mitenkään React- tai Node-spesifisiä asioita, vaan yleismaailmallisia periaatteita web-sovellusten toiminnasta.

Voimme sallia muista <i>origineista</i> tulevat pyynnöt käyttämällä Noden [cors](https://github.com/expressjs/cors)-middlewarea.

Asennetaan backendiin <i>cors</i> komennolla

```bash
npm install cors
```

Otetaan middleware käyttöön toistaiseksi sellaisella konfiguraatiolla, joka sallii kaikista origineista tulevat pyynnöt kaikkiin backendin Express routeihin:

```js
const cors = require('cors')

app.use(cors())
```

Nyt frontend toimii! Tosin muistiinpanojen tärkeäksi muuttavaa toiminnallisuutta backendissa ei vielä ole.

CORS:ista voi lukea tarkemmin esim. [Mozillan sivuilta](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Sovelluksen suoritusympäristö näyttää nyt seuraavalta:

![](../../images/3/100.png)

Selaimessa toimiva frontendin koodi siis hakee datan osoitteessa localhost:3001 olevalta Express-palvelimelta.

### Sovellus Internetiin

Kun koko "stäkki" on saatu vihdoin kuntoon, siirretään sovellus Internetiin. Käytetään seuraavassa vanhaa kunnon [Herokua](https://www.heroku.com).

> <i>Jos et ole koskaan käyttänyt Herokua, löydät käyttöohjeita kurssin [Tietokantasovellus](https://hy-tsoha.github.io/materiaali/osa-3/#sovellus-tuotantoon)-materiaalista ja googlaamalla...</i>

Lisätään backendin projektin juureen tiedosto <i>Procfile</i>, joka kertoo Herokulle, miten sovellus käynnistetään:

```bash
web: npm start
```

Muutetaan tiedoston <i>index.js</i> lopussa olevaa sovelluksen käyttämän portin määrittelyä seuraavasti:

```js
const PORT = process.env.PORT || 3001  // highlight-line
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

Nyt käyttöön tulee [ympäristömuuttujassa](https://en.wikipedia.org/wiki/Environment_variable) _PORT_ määritelty portti tai 3001, jos ympäristömuuttuja _PORT_ ei ole määritelty. Heroku konfiguroi sovelluksen portin ympäristömuuttujan avulla.

Tehdään projektihakemistosta Git-repositorio ja lisätään <i>.gitignore</i>, jolla on seuraava sisältö:

```bash
node_modules
```

Luodaan Heroku-sovellus komennolla _heroku create_, tehdään sovelluksen hakemistosta Git-repositorio, commitoidaan koodi ja siirretään se Herokuun komennolla _git push heroku main_ (tai komennolla _git push heroku master_).

Jos kaikki meni hyvin, sovellus toimii:

![](../../images/3/25ea.png)

Jos ei, vikaa voi selvittää Herokun lokeja lukemalla eli komennolla _heroku logs_.

> **HUOM:** ainakin alussa on järkevää tarkkailla Herokussa olevan sovelluksen lokeja koko ajan. Parhaiten tämä onnistuu antamalla komento _heroku logs -t_, jolloin logit tulevat konsoliin sitä mukaan kun palvelimella tapahtuu jotain.

Myös frontend toimii Herokussa olevan backendin avulla. Voit varmistaa asian muuttamalla frontendiin määritellyn backendin osoitteen viittaamaan <i>http://localhost:3001</i>:n sijaan Herokussa olevaan backendiin.

Seuraavaksi herää kysymys: miten saamme myös frontendin Internetiin? Vaihtoehtoja on useita, mutta käydään seuraavaksi läpi yksi niistä.

### Frontendin tuotantoversio

Olemme toistaiseksi suorittaneet React-koodia <i>sovelluskehitysmoodissa</i>, jossa sovellus on konfiguroitu antamaan havainnollisia virheilmoituksia, päivittämään koodiin tehdyt muutokset automaattisesti selaimeen ym.

Kun sovellus viedään tuotantoon, täytyy siitä tehdä [production build](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build) eli tuotantoa varten optimoitu versio.

<i>create-react-app</i>:in avulla tehdyistä sovelluksista saadaan tehtyä tuotantoversio komennolla [_npm run build_](https://github.com/facebookincubator/create-react-app#npm-run-build-or-yarn-build).

Suoritetaan nyt komento <i>frontendin projektin juuressa</i>.

**HUOM:** tätä kirjoitettaessa (20.1.2022) create-react-app:issa on pieni bugi, ja jos komento aiheuttaa virheilmoituksen _TypeError: MiniCssExtractPlugin is not a constructor_, korjaus ongelmaan löytyy [täältä](https://github.com/facebook/create-react-app/issues/11930), eli lisää tiedostoon <i>package.json</i> 

```json
{
  // ...
  "resolutions": {
    "mini-css-extract-plugin": "2.4.5"
  }
}
```

ja suorita komennot

```bash
rm -rf package-lock.json
rm -rf node_modules
npm cache clean --force
npm install
```

Komennon _npm run build_ pitäisi taas toimia.

Komennon seurauksena syntyy hakemiston <i>build</i> (joka sisältää jo sovelluksen ainoan html-tiedoston <i>index.html</i>) sisään hakemisto <i>static</i>, jonka alle generoituu sovelluksen JavaScript-koodin [minifioitu](<https://en.wikipedia.org/wiki/Minification_(programming)>) versio. Vaikka sovelluksen koodi on kirjoitettu useaan tiedostoon, generoituu kaikki JavaScript yhteen tiedostoon. Samaan tiedostoon tulee  myös kaikkien sovelluksen koodin tarvitsemien riippuvuuksien koodi.

Minifioitu koodi ei ole miellyttävää luettavaa. Koodin alku näyttää seuraavalta:

```js
!function(e){function r(r){for(var n,f,i=r[0],l=r[1],a=r[2],c=0,s=[];c<i.length;c++)f=i[c],o[f]&&s.push(o[f][0]),o[f]=0;for(n in l)Object.prototype.hasOwnProperty.call(l,n)&&(e[n]=l[n]);for(p&&p(r);s.length;)s.shift()();return u.push.apply(u,a||[]),t()}function t(){for(var e,r=0;r<u.length;r++){for(var t=u[r],n=!0,i=1;i<t.length;i++){var l=t[i];0!==o[l]&&(n=!1)}n&&(u.splice(r--,1),e=f(f.s=t[0]))}return e}var n={},o={2:0},u=[];function f(r){if(n[r])return n[r].exports;var t=n[r]={i:r,l:!1,exports:{}};return e[r].call(t.exports,t,t.exports,f),t.l=!0,t.exports}f.m=e,f.c=n,f.d=function(e,r,t){f.o(e,r)||Object.defineProperty(e,r,{enumerable:!0,get:t})},f.r=function(e){"undefined"!==typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"})
```

### Staattisten tiedostojen tarjoaminen backendistä

Eräs mahdollisuus frontendin tuotantoon viemiseen on kopioida tuotantokoodi eli hakemisto <i>build</i> backendin repositorion juureen ja määritellä backend näyttämään pääsivunaan frontendin <i>pääsivu</i> eli tiedosto <i>build/index.html</i>.

Aloitetaan kopioimalla frontendin tuotantokoodi backendin alle, projektin juureen. Omalla koneellani kopiointi tapahtuu frontendin hakemistosta käsin komennolla

```bash
cp -r build ../notes-backend
```

Backendin sisältävän hakemiston tulee nyt näyttää seuraavalta:

![](../../images/3/27ea.png)

Jotta saamme Expressin näyttämään <i>staattista sisältöä</i> eli sivun <i>index.html</i> ja sen lataaman JavaScriptin ym. tarvitsemme Expressiin sisäänrakennettua middlewarea [static](http://expressjs.com/en/starter/static-files.html).

Kun lisäämme muiden middlewarejen määrittelyn yhteyteen seuraavan

```js
app.use(express.static('build'))
```

tarkastaa Express GET-tyyppisten HTTP-pyyntöjen yhteydessä ensin löytyykö pyynnön polkua vastaavan nimistä tiedostoa hakemistosta <i>build</i>. Jos löytyy, palauttaa Express tiedoston.

Nyt HTTP GET -pyyntö osoitteeseen <i>www.palvelimenosoite.com/index.html</i> tai <i>www.palvelimenosoite.com</i> näyttää Reactilla tehdyn frontendin. GET-pyynnön esim. osoitteeseen <i>www.palvelimenosoite.com/api/notes</i> hoitaa backendin koodi.

Koska tässä tapauksessa sekä frontend että backend toimivat samassa osoitteessa, voidaan React-sovelluksessa eli frontendin koodissa oleva palvelimen _baseUrl_ määritellä [suhteellisena](https://www.w3.org/TR/WD-html40-970917/htmlweb.html#h-5.1.2) URL:ina eli ilman palvelinta yksilöivää osaa:

```js
import axios from 'axios'
const baseUrl = '/api/notes' // highlight-line

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...
```

Muutoksen jälkeen frontendistä on luotava uusi production build ja kopioitava se backendin repositorion juureen.

Sovellusta voidaan käyttää nyt <i>backendin</i> osoitteesta <http://localhost:3001>:

![](../../images/3/28e.png)

Sovelluksemme toiminta vastaa nyt täysin osan 0 luvussa [Single page app](/osa0/web_sovelluksen_toimintaperiaatteita#single-page-app) läpikäydyn esimerkkisovelluksen toimintaa.

Kun mennään selaimella osoitteeseen <http://localhost:3001> palauttaa palvelin hakemistossa <i>build</i> olevan tiedoston <i>index.html</i>, jonka sisältö hieman tiivistettynä on seuraava:

```html
<head>
  <meta charset="utf-8"/>
  <title>React App</title>
  <link href="/static/css/main.f9a47af2.chunk.css" rel="stylesheet">
</head>
<body>
  <div id="root"></div>
  <script src="/static/js/1.578f4ea1.chunk.js"></script>
  <script src="/static/js/main.104ca08d.chunk.js"></script>
</body>
</html>
```

Sivu sisältää ohjeen ladata sovelluksen tyylit määrittelevän CSS-tiedoston, sekä kaksi <i>script</i>-tagia, joiden ansiosta selain lataa sovelluksen JavaScript-koodin, eli varsinaisen React-sovelluksen.

React-koodi hakee palvelimelta muistiinpanot osoitteesta <http://localhost:3001/api/notes> ja renderöi ne ruudulle. Selaimen ja palvelimen kommunikaatio selviää tuttuun tapaan konsolin välilehdeltä <i>Network</i>:

![](../../images/3/29ea.png)

Tuotantoa varten tehty suoritusympäristö näyttää siis seuraavalta:

![](../../images/3/101.png)

Toisin kuin sovelluskehitysympäristössä, kaikki sovelluksen tarvitsema löytyy nyt node/express-palvelimelta osoitteesta localhost:3001. Kun osoitteeseen mennään, renderöi selain pääsivun <i>index.html</i> mikä taas aiheuttaa sen, että React-sovelluksen tuotantoversio haetaan palvelimelta ja selain alkaa suorittamaan sitä. Tämä taas saa aikaan sen, että ruudulla näytettävä JSON-muotoinen data haetaan osoitteesta localhost:3001/api/notes.

### Koko sovellus Internetiin

Kun sovelluksen "Internetiin vietävä" tuotantoversio todetaan toimivaksi paikallisesti, commitoidaan frontendin tuotantoversio backendin repositorioon ja pushataan koodi uudelleen Herokuun.

[Sovellus](https://obscure-harbor-49797.herokuapp.com/) toimii moitteettomasti lukuun ottamatta vielä backendiin toteuttamatonta muistiinpanon tärkeyden muuttamista:

![](../../images/3/30ea.png)

Sovelluksemme tallettama tieto ei ole ikuisesti pysyvää, sillä sovellus tallettaa muistiinpanot muuttujaan. Jos sovellus kaatuu tai se uudelleenkäynnistetään, kaikki tiedot katoavat.

Tarvitsemme sovelluksellemme tietokannan. Ennen tietokannan käyttöönottoa katsotaan kuitenkin vielä muutamaa asiaa.

Tuotannossa oleva sovellus näyttää seuraavalta:

![](../../images/3/102.png)

Nyt siis node/express-backend sijaitsee Herokun palvelimella. Kun selaimella mennään sovelluksen "juuriosoitteeseen", joka on muotoa https://glacial-ravine-74819.herokuapp.com/, alkaa selain suorittaa React-koodia joka taas hakee JSON-muotoisen datan Herokusta.

### Frontendin deployauksen suoraviivaistus

Jotta uuden frontendin version generointi onnistuisi jatkossa ilman turhia manuaalisia askelia, lisätään uusia skriptejä backendin <i>package.json</i>-tiedostoon:
```json
{
  "scripts": {
    // ...
    "build:ui": "rm -rf build && cd ../part2-notes/ && npm run build && cp -r build ../notes-backend",
    "deploy": "git push heroku main",
    "deploy:full": "npm run build:ui && git add . && git commit -m uibuild && git push && npm run deploy",    
    "logs:prod": "heroku logs --tail"
  }
}
```
Skripteistä _npm run build:ui_ kääntää ui:n tuotantoversioksi ja kopioi sen. _npm run deploy_ julkaisee Herokuun.

_npm run deploy:full_ yhdistää nuo molemmat sekä lisää vaadittavat <i>git</i>-komennot versionhallinnan päivittämistä varten. Lisätään lisäksi oma skripti _npm run logs:prod_ lokien lukemiseen, jolloin käytännössä kaikki toimii npm-skriptein.

Huomaa, että skriptissä <i>build:ui</i> olevat polut riippuvat repositorioiden sijainnista.

### Proxy

Frontendiin tehtyjen muutosten seurauksena on nyt se, että kun suoritamme frontendiä sovelluskehitysmoodissa eli käynnistämällä sen komennolla _npm start_, yhteys backendiin ei toimi:

![](../../images/3/32ea.png)

Syynä tälle on se, että backendin osoite muutettiin suhteellisesti määritellyksi:

```js
const baseUrl = '/api/notes'
```

Koska frontend toimii osoitteessa <i>localhost:3000</i>, menevät backendiin tehtävät pyynnöt väärään osoitteeseen <i>localhost:3000/api/notes</i>. Backend toimii kuitenkin osoitteessa <i>localhost:3001</i>.

create-react-app:illa luoduissa projekteissa ongelma on helppo ratkaista. Riittää, että frontendin repositorion tiedostoon <i>package.json</i> lisätään seuraava määritelmä:

```bash
{
  "dependencies": {
    // ...
  },
  "scripts": {
    // ...
  },
  "proxy": "http://localhost:3001"  // highlight-line
}
```

Uudelleenkäynnistyksen jälkeen Reactin sovelluskehitysympäristö toimii [proxynä](https://create-react-app.dev/docs/proxying-api-requests-in-development). Jos React-koodi tekee HTTP-pyynnön palvelimen <i>http://localhost:3000</i> johonkin osoitteeseen, joka ei ole React-sovelluksen vastuulla (eli kyse ei ole esim. sovelluksen JavaScript-koodin tai CSS:n lataamisesta), lähetetään pyyntö edelleen osoitteessa <i>http://localhost:3001</i> olevalle palvelimelle.

Nyt myös frontend on kunnossa. Se toimii sekä sovelluskehitysmoodissa että tuotannossa yhdessä palvelimen kanssa.

Eräs negatiivinen puoli käyttämässämme lähestymistavassa on, että sovelluksen uuden version tuotantoon vieminen edellyttää erillisessä repositoriossa olevan frontendin koodin tuotantoversion generoimista. Tämä taas hankaloittaa automatisoidun [deployment pipelinen](https://martinfowler.com/bliki/DeploymentPipeline.html) toteuttamista. Deployment pipelinellä tarkoitetaan automatisoitua ja hallittua tapaa viedä koodi sovelluskehittäjän koneelta erilaisten testien ja laadunhallinnallisten vaiheiden kautta tuotantoympäristöön. Aiheeseen tutustutaan kurssin [osassa 11](https://fullstackopen.com/osa11).

Tähänkin on useita erilaisia ratkaisuja (esim. sekä frontendin että backendin [sijoittaminen samaan repositorioon](https://github.com/mars/heroku-cra-node)), emme kuitenkaan nyt mene niihin. Myös frontendin koodin deployaaminen omana sovelluksenaan voi joissain tilanteissa olla järkevää. _create-react-app_:in avulla luotujen sovellusten osalta se on [suoraviivaista](https://github.com/mars/create-react-app-buildpack).

Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [GitHubissa](https://github.com/fullstack-hy2020/part3-notes-backend/tree/part3-3), branchissa <i>part3-3</i>.

Frontendin koodiin tehdyt muutokset ovat [frontendin repositorion](https://github.com/fullstack-hy2020/part2-notes/tree/part3-1) branchissa <i>part3-1</i>.

</div>

<div class="tasks">

### Tehtävät 3.9.-3.11.

Seuraavissa tehtävissä koodia ei tarvita montaa riviä. Tehtävät ovat kuitenkin haastavia, sillä nyt on tarkalleen hallittava, missä tapahtuu mitäkin, ja kaikki konfiguraatiot on tehtävä täsmälleen oikein. 

#### 3.9 puhelinluettelon backend step9

Laita backend toimimaan edellisessä osassa tehdyn puhelinluettelon frontendin kanssa muilta osin paitsi mahdollisen puhelinnumeron muutoksen osalta, jonka vastaava toiminnallisuus toteutetaan backendiin vasta tehtävässä 3.17.

Joudut todennäköisesti tekemään frontendiin erinäisiä pieniä muutoksia ainakin backendin oletettujen urlien osalta. Muista pitää selaimen konsoli koko ajan auki. Jos jotkut HTTP-pyynnöt epäonnistuvat, kannattaa katsoa <i>Network</i>-välilehdeltä, mitä tapahtuu. Pidä myös silmällä, mitä palvelimen konsolissa tapahtuu. Jos et tehnyt edellistä tehtävää, kannattaa POST-pyyntöä käsittelevässä tapahtumankäsittelijässä tulostaa konsoliin mukana tuleva data eli <i>request.body</i>.

#### 3.10 puhelinluettelon backend step10

Vie sovelluksen backend Internetiin, esim. Herokuun. 

**Huom.** komento _heroku_ toimii laitoksen koneilla ja fuksikannettavilla. Jos et jostain syystä saa [asennettua](https://devcenter.heroku.com/articles/heroku-cli) herokua koneellesi, voit käyttää komentoa [npx heroku](https://www.npmjs.com/package/heroku).

Testaa selaimen ja Postmanin tai VS Coden REST-clientin avulla, että Internetissä oleva backend toimii.

**PRO TIP:** kun deployaat sovelluksen Herokuun, kannattaa ainakin alkuvaiheissa pitää **KOKO AJAN** näkyvillä Herokussa olevan sovelluksen loki antamalla komento <em>heroku logs -t</em>.

Seuraavassa loki eräästä tyypillisestä ongelmatilanteesta, jossa Heroku ei löydä sovelluksen riippuvuutena olevaa moduulia <i>express</i>:

![](../../images/3/33.png)

Syynä ongelmalle on se, että <i>Express</i>-kirjastoa ei ole asennettu <em>npm install express</em> komennolla, joka tallentaa tiedon riippuvuudesta tiedostoon <i>package.json</i>. 

Toinen tyypillinen ongelma on se, että sovellusta ei ole konfiguroitu käyttämään ympäristömuuttujana <em>PORT</em> määriteltyä porttia:

![](../../images/3/34.png)

Tee repositorion juureen tiedosto README.md ja lisää siihen linkki Internetissä olevaan sovellukseesi.

#### 3.11 puhelinluettelo full stack

Generoi frontendistä tuotantoversio ja lisää se Internetissä olevaan sovellukseesi tässä osassa esiteltyä menetelmää noudattaen.

**HUOM:** eihän hakemisto <i>build</i> ole gitignoroituna projektissasi?

Huolehdi myös, että frontend toimii edelleen myös paikallisesti.

Jotta kaikki toimisi, tulee repositoriosi näyttää hakemisttorakenteen suhteen samalta kuin [esimerkkisovelluksen repositorion](https://github.com/fullstack-hy2020/part3-notes-backend/tree/part3-3).

</div>
