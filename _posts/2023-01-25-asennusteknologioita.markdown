---
layout: post
title:  "Asennuksessa tarvittavia teknologioita"
date:   2023-01-24 13:51:34 +0200
categories: jekyll update
---
Käytännössä seuraavia taustateknologioita tarvitaan asennuksessa.

1.	git ja GitHub

Parissa asennuksen kohdassa siirretään GitHubissa oleva materiaali omalle koneelle. Itse käytin tähän mahdollisimman yksinkertaista mallia. Olettaen, että git on jo asennettu koneellesi, Linuxin komentorivillä tarvitsee kirjoittaa SDP-moduulien kohdehakemistossa:

git clone [Code-kohdassa annettu https-osoite; ensin Controller-hakemistossa ja sitten fwknop-hakemistossa]

2.	node.js ja npm

Controller-moduuli on kirjoitettu JavaScriptillä ja sen ajamiseen tarvitaan node.js-ajoympäristö sekä node package manager (npm). Näiden asennuksen pitäisi sujua ongelmitta ohjetta noudattaen.

3.	MySQL-tietokanta

Controller-koneelle luodaan MySQL-tietokanta, jonka tauluihin täytetään SDP-moduulien olennaisia tietoja. Riippuen taustastasi tämä voi olla helppoa tai hankalampaa. Itse käytin tässä vaiheessa melko paljon aikaa, joten työtäsi helpottaakseni olen luonut mallikomentoja, joilla saat varsin nopeasti täytettyä taulut. Toki voit tehdä omatkin taulusi.

4.	OpenSSL

SDP-projektin avulla saa väistämättä harjoitella myös kryptografiaa. Projektissa käytetään avoimen lähdekoodin OpenSSL-kirjastoa, jonka avulla hallitaan usein esimerkiksi web-selainten suojattua TLS-liikennettä. 
Salaus voi olla symmetrinen tai asymmetrinen riippuen siitä, käyttävätkö lähettäjä ja vastaanottaja samaa vai eri salausavainta. Tässä projektissa käytetään asymmetristä eli julkisen avaimen salausta. Controller-koneen asennuksen yhteydessä luodaan oma juurivarmenne, jonka pohjalta tarjotaan käyttöön tuleva varmenne ja allekirjoitetaan eri laitteiden käyttöön tulevat palvelinvarmenteet. Tässä käytetään komentorivillä OpenSSL:ää, hyvin monipuolista avoimen lähdekoodin kryptografista alustaa, jonka tunteminen on järjestelmän ylläpitäjille ja tietoturvan toteuttajille hyödyllistä.
Oma OpenSSL-osaamiseni ei ollut projektin aikana kovin vahvaa, joten törmäsin varsin kiusallisiin ongelmiin. Alusta myös kehittyy koko ajan, eikä sen dokumentaatio ole aina kovin selkeää. Pyrin asennusohjeessa käsittelemään näitä haasteita ja toivon, että saat oman autentikointisi ja sähköisen allekirjoituksesi toimimaan kunnialla. 

5.	iptables

Linux-käyttöjärjestelmän ytimessä sijaitseva tehokas pakettisuodatinjärjestelmä on nimeltään netfilter. Tätä kernel-pohjaista suodatusta on käyttäjätilassa (user space) perinteisesti hallinnoitu iptables-käyttöliittymällä, jota tässäkin ohjeessa käytetään. Netfilter-projektin kehittäjät pyrkivät tosin saamaan käyttäjäkunnan siirtymään edistyneempään nftables-käyttöliittymään, joka on kuitenkin jätetty tämän ohjeen ulkopuolelle, koska alkuperäinen ohje käytti iptablesia.
Iptables-käyttöliittymän hallintalogiikka vaatii oman harjoittelunsa. Jos käyttö ei ole sinulle ennestään tuttua, yksi varoituksen sana tässä asennuksessa: Jos teet asetuksia SSH-etäyhteyden kautta, seuraamalla orjallisesti Waverley Labsin ohjetta voit vahingossa sulkea itsesi koko koneen ulkopuolelle (myönnän – tämä tapahtui aluksi minulle). Ohjeen kronologia voi tässä viedä harhaan, sillä ensimmäisessä kohdassa ohjeistetaan luomaan kaiken liikenteen estävä palomuurisääntö. Jos et tätä ennen ole varmistanut pääsyä sisään koneelle, tämä sääntö blokkaa sinut ulos. Uusi yhteys on mahdollista luoda esimerkiksi konsoliyhteydellä tai asentamalla käyttöjärjestelmä/virtuaalikone uudelleen.
Iptables-konfiguroinnissa voin suositella käyttämään Michael Rashin kirjoittamaa skriptiä, johon palataan tarkemmin alla. Sanottakoon, että fwknop-asennus luo iptables-tauluja vastaavat uudet taulut, joiden tila voidaan tarkistaa syslog-viestien avulla. Rashin SPA/fwknop-tutoriaali näyttää esimerkkejä tästä.

6.	conntrack

Toinen iptablesin hallintaan liittyvä tärkeä sääntö liittyy käsitteeseen connection tracking (conntrack). Conntrack pitää käytännössä yllä kernelissä olevia tilatietoja käynnissä olevista yhteyksistä. Se on user spacen puolella oleva työkalu, joka tulee olla käytössä iptables-sääntöjä tehdessä. Käytännössä juuri conntrack syöttää fwknopin palomuurihallinnalle tiedon siitä, onko autentikoitu yhteys kerran luotu (ESTABLISHED), ja tämän kernel-tason tiedon perusteella palomuuri osaa jatkaa juuri tämän yhteyden sallimista sulkien kaikki muut käyttäjätilan tasolta tulevat yhteysyritykset (esim. uusi autentikoimaton SSH-yhteys tai nmap-skannausyritys). 
Connection trackingin toiminnan varmistaminen voi olla työlästä. Valmiissa SDP-ohjeessa mielestäni toimitaan hieman takaperoisesti, sillä sitä orjallisesti seuraten conntrackin pitäisi olla toiminnassa jo ennen asennusta. Itse käytin mainittua Michael Rashin asennusskriptiä, joka huolehtii myös conntrack-asetukset kuntoon. Skriptin tulkinta on hyödyllinen oppimiskokemus itsessään.

7.	SSH

Secure Shell (SSH) -yhteyttä käytetään koneiden väliseen turvalliseen etäyhteyteen. SSH:n käyttö liittyy teemoiltaan OpenSSL:ään: luodaan yksityinen ja julkinen avain, pidetään yksityinen avain vain omana tietona ja toimitetaan julkinen avain autentikointia varten kohdekoneelle.
Kohdekoneella on sekä ssh client että ssh server eli sshd-ohjelma (daemon), joka aktiivisesti kuuntelee yhteydenottoja. Tarvittaessa asennus voidaan tehdä:
sudo apt install openssh-server
sudo systemctl enable ssh
Avainten luonnissa voi käyttää eri algoritmeja. Ohjeessa käytössä on RSA-algoritmi, jonka jälkeen suositellummaksi tavaksi on tullut elliptisen käyrän kryptografiaa (ECC) hyödyntävä Ed25519-algoritmi. SSH-avaimen luonti sillä tapahtuu näin:
ssh-keygen -t ed25519 

8. Linux

Tämän ohjeen seuraamiseksi on hyvä olla jonkinlainen peruskokemus Linux- komentojen suorittamisesta, sillä valtaosa toiminnasta tapahtuu Linuxin komentorivillä. Tässä ohjeessa ei oteta kantaa eri Linux-jakeluiden toiminnallisuuksiin tai paketinhallintaratkaisuihin.



Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/