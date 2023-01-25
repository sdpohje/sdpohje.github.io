---
layout: post
title:  "Gateway-moduulin asentaminen"
date:   2023-01-24 13:51:34 +0200
categories: jekyll update
---
Tämä asennus on sinänsä suoraviivaisempaa kuin edellinen, mutta Linuxin palomuurin toimintaperiaate on syytä tuntea ainakin jollain tasolla. Olennaista on muistaa, ettei blokkaa itseään ulos asennettavalta Gateway-palvelimelta, jos siihen ottaa SSH-etäyhteyden. SDP-ohjeen kohdan 2 kronologia nimittäin voi johtaa harhaan.
Tämä asennusohje on tehty omaan kotilabraympäristööni aiemmin käytetyn etäympäristön sijaan. Tällä halusin varmistaa toiminnan. Pieniä eroja voi siksi näkyä host-koneiden nimeämistyyleissä.
1.	Linuxin palomuurin asetukset: iptables

Iptablesin voi tarvittaessa asentaa komennolla 
sudo apt install iptables
Olemassa olevat palomuurisäännöt (INPUT, FORWARD, OUTPUT) näkee komennolla
sudo iptables -L

KUVA

2.	Connection tracking ja iptables

Tässä vaiheessa kaiken liikenteen pitäisi olla sallittua (ACCEPT). Voit toki harjoitella iptables-komentoja, se on hyvää oppia. Etenemisen sujuvoittamiseksi suosittelen ajamaan Gateway-koneelle Michael Rashin skriptin:
https://www.cipherdyne.org/LinuxFirewalls/ch01/iptables.sh.tar.gz
Tämän selitys löytyy kohdasta 1.3 Default-Drop firewall policy sivulta
https://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html
Voi myös käyttää Rashin lyhyempää versiota samalla sivulla:
iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -j DROP
iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
Myös SDP-asennusohjeessa on tästä lyhennetty versio ilman protokollan (TCP) ja kohdeportin (22) määrityksiä.
Olennaista on ymmärtää optioiden -A ja -I ero. A eli Append lisää sääntöjä loppuun, kun taas I sijoittaa säännöt määriteltyyn kohtaan, kuten yllä INPUT-taulun riville 1 eli ensimmäiselle riville. Säännöt pätevät ylhäältä alas -järjestyksessä, eli alin ohittaa ylimmän. 
Edellä kuvatut Rashin kaksi komentoa taas toimivat seuraavasti. Ensimmäinen hyppää (-j eli Jump) pudottamaan (DROP) kaiken pakettiliikenteen kohdeportista 22 (tässä SSH-oletusportti). Toinen taas hyppää ACCEPT-sääntöön käyttäen kernelin conntrack-ominaisuutta eli olemassa olevien yhteyksien seurantaa. Tämä toinen sääntö hyppää listan ylimmäksi riville 1, eli se toteutetaan ennen DROP-komentoa. Näin varmistetaan, että jo olemassa oleva SSH-etäyhteys pysyy päällä silloinkin, kun DROP-sääntö (tai policy) sen muutoin estää.
Taulun säännöt ovat merkitsevämpiä kuin policyt. Kuitenkin tavoitteena on tuottaa Default Drop policy.
Iptables-asetukset voi tarvittaessa tallettaa:
sudo /sbin/iptables-save
Itse en tehnyt forwarding-asetusta ohjeen kohdan 2 lopussa.

3.	fwknop: taustakirjastojen asentaminen

Asennetaan seuraavaksi fwknopin tarvitsemat taustakirjastot ohjeen komennon mukaisesti.

4.	Projektin kloonaus

Alustetaan git myös tälle koneelle ja kloonataan projekti GitHubista: EI INIT


KUVA

5. Mennään oikeaan hakemistoon

Esimerkissä /home/gateway1/fwknop. Kuten mainittiin, Gateway-kone asennetaan tässä kotilabraympäristööni, jossa nimeämiskäytäntö eroaa hieman Controller-esimerkistä.

6. Kirjoitetaan komennot

Ohjeen mukaan. Itselläni tuli oireita mahdollisista vaikeuksista, joiden vaikutuksesta kuulen mielelläni tarkemmin asiasta tietäviltä.


KUVA

7. Konfiguroidaan projekti kääntämistä varten

Koska kyseessä on Gatewayn asennus, ei asenneta tälle koneelle Clientia eli käytetään lisämäärettä –disable client:
./configure --prefix=/usr --sysconfdir=/etc --with-iptables=/path/to/iptables --disable-client

8. Kääntäminen

Suoraviivaisesti: 
make

9. Asentaminen

sudo make install

Itselläni tässä tuli erroria ja ajoin ohjeen mukaan tarvittavat komennot uudestaan. Tuloksen tulisi kuitenkin näyttää koko lailla tältä:

KUVA

10. Tiedostojen editointi

Asetustiedostoissa on varsin hyvät selvitykset. Itse puutuin kahteen ensin mainittuun tiedostoon mutta en gate.fwknoprc-tiedostoon.

KUVA

11. Gatewayn käynnistys

Tässä vaiheessa Controller kannattaa pitää käynnissä. Jos kaikki menee hyvin, yhteys koneiden välillä syntyy.








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
