---
layout: post
title:  "Asennusympäristöstä huomioitavaa"
date:   2023-01-24 13:51:34 +0200
categories: jekyll update
---
Tämä ohje on toteutettu Linux Ubuntu Desktop -virtuaalikoneympäristössä. Ohjetta varten oli olemassa kaksi ympäristöä, joita yhdisti yhteinen Client-kone:

•	Windows 11 -isäntäkoneessa VirtualBox-alustalla Ubuntu-virtuaalikone nimeltä SDP Client, joka simuloi mitä tahansa etäympäristöstä SDP-yhdyskäytävän suojaamaan ympäristöön yhteyttä ottavaa asiakasta. Tämä SDP Client otti SSH-yhteyden internetin yli rakennettuun etäympäristöön, jossa oli palomuurin takana useampi kone: SDP Gateway, Controller, Client ja lisäksi Gatewayn suojaama ns. VIP Asset -kone, joka edusti organisaation arvo-omaisuutta.
•	Edellisen VirtualBox-ympäristön sisäinen SDP-ympäristö, jossa oli Client-koneen lisäksi Gateway- ja Controller-koneet. Näiden avulla pystyi kokeilemaan toteutusta ilman verkon yli menevää SSH-etäyhteyttä.

SDP-toteutuksen koodi on testattu vain *nix-tyyppisissä järjestelmissä. Ohjeen oletuksena on Linux Ubuntu -ympäristö.



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