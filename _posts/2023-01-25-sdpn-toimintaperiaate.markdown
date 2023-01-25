---
layout: post
title:  "SDP:n toimintaperiaate"
date:   2023-01-24 13:51:34 +0200
categories: jekyll update
---
Controller on SDP:n toiminnan ”aivot”. Siihen rakennetaan tietokanta, joka sisältää konetunnukset, suojattavat palvelut ja yhdyskäytävät, jotka suojelevat tietyn organisaation arvokasta tietoa. Controller-koneelle annettujen tietojen pohjalta tehdään palomuurin tarkkaan hallintaan perustuvat päätökset siitä, mikä asiakaskone saa ottaa yhteyden mihinkin tarvitsemaansa yksittäiseen palveluun. Tällainen voi olla esim. SSH, HTTPS tai FTP, mutta laajemmin lähes mikä tahansa muu palvelu.

SDP-toteutuksen toiminta perustuu 2000-luvun alussa kehitettyihin Linux-palomuurin käsittelytapoihin. Palomuuri suodattaa liikennettä sisään ja ulos sille määriteltyjen sääntöjen mukaan. Monen kyberhyökkäyksen suunnittelu on alkanut avointen porttien skannauksella nmap-työkalulla. Jos kaikki portit ovat kiinni, hyökkäyksen toteutus esimerkiksi Metasploit-työkalua hyödyntäen on huomattavasti vaikeampaa. Cloud Security Alliance järjesti vuosia sitten murtautumishaasteen, jossa moni taitava eettinen hakkeri ei päässyt murtautumaan SDP:llä suojattuun kohdeympäristöön.

Yritysympäristössä koneiden luvallinen etähallinta vaikeutuu kohtuuttomasti, jos esimerkiksi SSH-yhteyttä ei voi ottaa palomuurin suljettua kaikki portit. Tämän estämiseksi on kehitetty port knocking -toimintatapa, jossa yhteyksiä voidaan avata täysin suljetunkin palomuurin läpi ”kolkuttamalla” portteja tietyssä järjestyksessä. Tämä koputtelu avaa palomuuriin oven ja mahdollistaa vaikkapa SSH-hallintayhteyden autentikoinnin niin, että heti varmennetun yhteyden avaamisen jälkeen koko palomuuri sulkeutuu taas.

Tietoturvatutkija Michael Rash kirjoitti 2000-luvun alussa port knocking -teknologiaan perustuvan fwknop-työkalun (Firewall Knock Operator). Se käyttää Linuxin käyttäjätilan puolella (user space) netfilter-palomuurin hallintaan tarkoitettua iptables-työkalua, tarkemmin tämän lokitietoja, päästääkseen vain toivotut ja oikein autentikoidut yhteydenotot palomuurin läpi. Lisäksi fwknop osaa tunnistaa yhteyttä pyytävän käyttöjärjestelmäversion.

Fwknop kehittyi sittemmin edelleen Single-Packet Authorization -protokollaksi (SPA). Nimensä mukaisesti se antaa kohdepalveluun pääsyoikeudet vain yhdelle tiedonsiirtopaketille kerrallaan. SPA enkoodaa autentikaatiotiedon sovelluskerroksen dataan, tarkemmin lähetettävään hyötykuormaan (packet payload), kun taas aiempi fwknop tallensi tiedot otsikkotietoihin (packet header). Tässä esiteltävä Software-Defined Perimeter -toteutus perustuu SPA-protokollaan.

Mainituista teknologioista on useita versiota. Jo vuonna 2008 Rash arvioi löytyvän 25 port knocking -implementaatiota ja viisi SPA-versiota. SDP-toteutuksia on Zero Trust -ajattelun valtavirtaistumisen myötä hyvin paljon, ja nyt esiteltävä open source -ohje esittelee vain yhden, perusmuotoisen järjestelmän rakentamisen.

Tämä ohje perustuu erityisesti kahteen lähteeseen:

•	Waverley Labsin GitHubissa julkaisemaan asennusohjeeseen Open Source Software Defined Perimeter Installation and Configuration, Version 0.2, July 2020
•	Michael Rashin kotisivuillaan julkaisemaan ohjeeseen Single Packet Authorization - A Comprehensive Guide to Strong Service Concealment with fwknop, updated January 2016

Näistä edellinen on tiivis step by step -asennusohje, jonka pohjalta SDP-asennuksen pitäisi olla mahdollinen. Tämä ohje seuraa alkuperäisen ohjeen numerointia.

Käytännössä matkan varrella voi tulla haasteita, joita helpottamaan ja tiedon jakamisen edistämiseksi tämä ohje on kirjoitettu. Waverley Labsin toteutus on julkaistu GNU General Public -lisenssillä, jonka puitteissa sitä saa jakaa edelleen ja muokata. Ohjelman toimivuudesta ei lisenssin puitteissa anneta erikseen takuuta.

Jälkimmäinen eli Rashin ohje on pidempi ja avaa lukijalle erityisesti Gateway- ja Client-koneisiin vaikuttavia taustateknologioita syvällisemmin. Ensimmäistä kertaa SDP-asennusta tekevän olisi hyvä ymmärtää mahdollisimman hyvin molempia ohjeita, mutta niihin perehtyminen voi viedä aikaa.

Käsillä oleva ohje pyrkii sujuvoittamaan ja nopeuttamaan asennusta tai ainakin sen harjoittelemista. Ohjeessa on pyritty selventämään mahdollisia vastaan tulevia ongelmakohtia, mutta sen noudattaminen ei itsessään takaa SDP-järjestelmän toimivuutta. Kuulen erittäin mielelläni palautettasi: Saitko järjestelmän toimimaan? Mitä haasteita matkan varrella vielä kohtasit? Miten ne ratkaisit?


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
