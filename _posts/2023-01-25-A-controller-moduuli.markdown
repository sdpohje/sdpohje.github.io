---
layout: post
title:  "Controller-moduulin asentaminen"
date:   2023-01-24 13:51:34 +0200
categories: jekyll update
---
Controller koostuu muutamasta peruskomponentista, jotka on kuvattu alle. Asennuskoneet kannattaa muutoin pitää mahdollisimman puhtaina erillisistä ohjelmista – peruspäivitykset toki on syytä tehdä.

(KUVA)

1.	Node 

Aluksi kannattaa tehdä tavalliset Ubuntun pakettitilanteen päivitykset (tässä käyttäen listaoperaattoria &&, mutta toki voi kirjoittaa erikseenkin):
sudo apt update && sudo apt upgrade 
Asennus lähtee liikkeelle Node.js:n asennuksella. 
sudo apt install nodejs
Itselläni tämä ei aiheuttanut ongelmia puhtaassa Ubuntu-koneessa. Tarkistetaan Noden versio ja varmistetaan samalla asennus:
node -v

2.	NPM

Sitten kannattaa tarkistaa, onko Node.js packace manager asentunut: 
npm
Tässä tapauksessa ei, joten asennan:
sudo apt install npm
Tarkistan komennolla npm, että on asentunut ja versio näkyy. Huom! Snapin kautta asentaessa voi tulla eroavaisuuksia, joten itse suosittelen asentamaan apt:n avulla.
3.	Git-projektin kloonaus

Lataa tai kloonaa SDP Controller GitHubista. Käytän itse tähän kloonausta. Tarkista, että Git toimii. Sitten tee Controller-koneelle Git-repositorio haluamastasi hakemistosta käsin, joka minulla on /home/samik/:
sudo git clone https://github.com/WaverleyLabs/SDPcontroller.git
Tämä luo SDPcontroller-alihakemiston ja Gitin versionhallinnan.
4.	Node-pakettien asennus

Seuraavaksi siirrytään cd-komennolla tähän hakemistoon home/samik/SDPcontroller, jossa asennetaan node-packaget:
npm install

5.	MySQL

Seuraavaksi asennetaan MySQL-palvelin. Ollaan edelleen samassa hakemistossa SDPcontroller:
sudo apt install mysql-server
(Setting up mysql-server-8.0 (8.0.31-0ubuntu0.22.04.1)
Voit tarkistaa, että server on päällä:
sudo systemctl status mysql.service
tai
sudo service mysql status
Toivottavasti MySQL-palvelin on ongelmitta päällä. Seuraavat vaiheet voivat tapahtua hieman eri tavoin riippuen järjestelmästä ja toimintatavasta. Tässä vaiheessa alkuperäinen ohje antaa niukat tiedot, joita nyt laajennetaan. Lisäohjeita voit katsoa mm. täältä:
https://ubuntu.com/server/docs/databases-mysql
https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04
https://www.digitalocean.com/community/tutorials/how-to-import-and-export-databases-in-mysql-or-mariadb

6.	Esimerkkitietokannan tuominen

Ennen sample-kannan hakua tulee tehdä tietokantaan turva-asetukset. Avataan MySQL-kehote:
sudo mysql
Voimme nähdä kaikki tietokannat (MySQL-komentojen jälkeen tulee aina puolipiste):
SHOW DATABASES;
Katsotaan MySQL-nimisestä tietokannasta tiedot käyttäjistä ja tunnistautumistavoista:
SELECT user, plugin FROM mysql.user;
Tein tämän muutoksen:
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
Tässä pääsyoikeudet saaneen root-käyttäjän tunnistautumistavaksi määritellään MySQL:n natiivisalasana, joka tässä tapauksessa on ’password’. 
MySQL-kehotteessa voimme sanoa ’exit’. Seuraavaksi suojaamme tietokannan:
sudo mysql_secure_installation
Aluksi määritellään rootille uusi salasana. Seuraaviin turvakovennuksiin voi vastata kyllä eli Y.

Tämän jälkeen loin root-käyttäjänä kantaan esimerkkitietokannan tällaisella komennolla:

KUVA

Menemällä takaisin MySQL-kehotteeseen voimme katsoa, mitä tauluja tuodussa kannassa on:

KUVA

7.	Luodaan uusi käyttäjä tietokantaan

Tavoitteemme on luoda rootin lisäksi toinen käyttäjä käyttämään sdp.sql-tietokantaa. Se tapahtuu CREATE USER -komennolla, itse luon käyttäjän ’samik’. Seuraavassa tämän käyttäjän kirjautumistavaksi laitetaan authentication_plugin (katso aiemmista MySQL-ohjeista lisätietoja):

KUVA

Uuden käyttäjän pitäisi nähdä hieman eri tietokannat kuin root-käyttäjän:

KUVA

Komennolla DESCRIBE voi nähdä taulujen kenttien vaadittavia tyyppejä ja esimerkiksi olennaisen Primary Keyn, kuten seuraavasta taulusta sdpid:

KUVA

8.	Valmiit tiedot taulukoihin

Waverley Labsin ohjeen kohdissa 8a – 8d on eritelty vähintään neljä täytettävää taulua. Tein itse ensin suunnitelman aiemmassa kuvassa näkyvistä 15 taulusta (nro 1 closed_connection – nro 15 user_group), mutta päätin sitten täyttää vain vähimmäismäärän neljä taulua, jotka ovat kuvan listauksessa taulut:
•	10 (sdpid)
•	11 (sdpid_service)
•	12 (service) ja 
•	13 (service_gateway)
Seuraavassa on mallina arvot näihin neljään tauluun. Arvoja voi joutua muuttamaan myöhemmin, mutta se lienee helpompaa valmiiden arvojen pohjalta. Lisäykseen on varmasti parempiakin tapoja, mutta omalla SQL-osaamisellani tein nyt näin.



Taulu 10:
INSERT INTO sdpid VALUES ('100', 1, 'controller', 'US', 'California', 'Los Angeles', 'Corp Corp', NULL, NULL, NULL, NULL, NULL, '12345', '2022-12-08 16:16:00', '2023-06-05 23:59:00', NULL, NULL);
INSERT INTO sdpid VALUES ('200', 1, 'gateway', 'US', 'California', 'Los Angeles', 'Corp Corp', NULL, NULL, NULL, NULL, NULL, '23456', '2022-12-08 16:16:00', '2023-06-05 23:59:00', NULL, NULL);
INSERT INTO sdpid VALUES ('300', 1, 'client', 'US', 'California', 'Los Angeles', 'Corp Corp', NULL, NULL, NULL, NULL, NULL, '34567', '2022-12-08 16:16:00', '2023-06-05 23:59:00', NULL, NULL);
INSERT INTO sdpid VALUES ('400', 1, 'client', 'FI', 'Uusimaa', 'Helsinki', 'Distance Corp', NULL, NULL, NULL, NULL, NULL, '45678', '2022-12-08 16:16:00', '2023-06-05 23:59:00', NULL, NULL);
Taulu 11:
INSERT INTO sdpid_service VALUES ('1', '1', '2');
INSERT INTO sdpid_service VALUES ('2', '1', '3');
INSERT INTO sdpid_service VALUES ('3', '2', '1');
INSERT INTO sdpid_service VALUES ('4', '2', '2');
INSERT INTO sdpid_service VALUES ('5', '2', '3');
INSERT INTO sdpid_service VALUES ('6', '3', '1');
INSERT INTO sdpid_service VALUES ('7', '3', '2');
INSERT INTO sdpid_service VALUES ('8', '3', '3');
INSERT INTO sdpid_service VALUES ('9', '4', '1');
INSERT INTO sdpid_service VALUES ('10', '4', '2');
INSERT INTO sdpid_service VALUES ('11', '4', '3');
Taulu 12:
INSERT INTO service VALUES ('1', 'controller', 'controller service');
INSERT INTO service VALUES ('2', 'data_service1_lin', 'VIP asset Linux');
INSERT INTO service VALUES ('3', 'data_service2_win', 'VIP asset Windows ');
Taulu 13:
INSERT INTO service_gateway VALUES ('1', '1', '1', 'udp', '443', '0.0.0.0', '443');
INSERT INTO service_gateway VALUES ('2', '2', '1', 'tcp', '443', '0.0.0.0', '443');
INSERT INTO service_gateway VALUES ('3', '2', '1', 'tcp', '443', '0.0.0.0', '443');

Sitten vain exit.

9.	Config.js-tiedoston asetukset

Seuraavaksi katsotaan kansiossa home/samik/SDPcontroller olevaa määritystiedostoa:
nano config.js
Itse muutin tämän asetuksia vasta myöhemmin, mutta toki jo nyt voi löytämiään asetuksia muuttaa.
10.	OpenSSL-asennus

Asenna tarvittaessa openssl, vaikka yleensä se tulee Linux-asennuksen mukana. Tämän voi tarkistaa:
openssl version
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)

11.	OpenSSL-varmenteet ja PKI-infrastruktuurin rakentaminen

Sitten siirrymme vaiheeseen, jossa oma rajallinen kokemukseni tuli vastaan. Annetut OpenSSL-asennusohjeet johtivat useaan virhetilanteeseen, joiden ratkomisessa päädyin seuraamaan useita JavaScript-funktiokutsuja tiedostosta toiseen. Jos saat varmenteet normaalisti toimimaan, hienoa! Valitettavasti ohjeen tämä kohta aiheutti kuitenkin itselleni suuria ongelmia. Mennään kuitenkin eteenpäin. Hyvä OpenSSL-ohje on esimerkiksi Ivan Ristićin OpenSSL Cookbook: https://www.feistyduck.com/books/openssl-cookbook/
Waverley Labsin SDP-ohjeen PDF-versiossa on kohdassa 11 kaksi komentoa ja Controller-moduulin GitHub-kuvauksen README.md-tiedostossa puolestaan yksi koottu komento. Seurataan aluksi ensimmäistä PDF-versiota.
openssl genrsa -des3 -out ca.key 4096
Tämä luo RSA-salausalgoritmiin perustuvan yksityisen avaimen, joka salataan symmetrisellä Triple DES -salausalgoritmilla. Lopussa RSA:n avainparin laskemiseen tarvittavan moduluksen pituus määritellään kaksinkertaiseksi (= 4096) ohjeen tekoaikana ilmeisesti OpenSSL:ssä tyypilliseen 2048 bittiin nähden. Määre 
-out ohjaa luotavan yksityisen avaimen kirjoitettavaksi tiedostoon nimeltä ca.key.
Seuraava komento 
openssl req -new -x509 -days 365 -key ca.key -out ca.crt
luo varmenteen luontipyynnön. Varmenne on voimassa 365 päivää, sen yksityinen avain on äsken luotu ca.key, ja varmenne kirjoitetaan tiedostoon ca.crt. 
RSA-algoritmi ei ole enää OpenSSL-projektin suositus, vaan sen on korvannut Ed25519. Muitakin versiomuutoksia tapahtuu jatkuvasti. Itse sain tässä vaiheessa virheen:
Error: RSA private key not found from openssl output
Tätä selviteltiin pitkään ja hartaasti. Tässä ohjeessa ei valitettavasti voida syventyä OpenSSL:n komentojen kirjoon, koska kyseessä on todellinen kryptografian Swiss Army Knife. Lopulta sain varmenteet luotua.

Toinen vaihtoehto SDP-ohjeessa on käyttää komentoa 
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:4096 -keyout ca.key -out ca.crt
jota voi halutessaan käyttää myös (kumpi sinulla toimii paremmin?).

12. Palvelinvarmenteiden ja -avainten luonti

Tässä luodaan esimerkkiavaimet SDP:n Client-, Gateway- ja Controller-komponenteille. Itse käytin SDPID-arvoja 100 (Controller), 200 (Gateway), 300 (sisäinen Client) ja 400 (ulkoinen Client). Kaikki komennot näkyvät tässä:

KUVA


Palvelinvarmenteet näkyvät kansiolistauksessa:

KUVA

13. Controllerin käynnistys

Controller käynnistetään komennolla
node ./sdpController.js
Viimeistään tässä vaiheessa voi olla syytä tarkistaa config.js-tiedoston asetuksia, esim. varmenteiden sijaintipolkuja.


KUVA

Jos kaikki sujuu hyvin, Controller käynnistyy:

KUVA

Ohjeen kohdan 12 mukaisesti pitää vielä muistaa siirtää crt- ja key-tiedostot Gateway- ja Client-koneille ja toimia muiden ohjeiden mukaan.







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
