---
layout: post
title:  "Svenska myndigheter och HTTPS"
date:   2018-04-23
---

I [myndighetsregistret](http://www.myndighetsregistret.scb.se) på SCB finns följande typer myndigheter:

1. Statliga förvaltningsmyndigheter (254 st)
2. Myndigheter under riksdagen (4 st)
3. Statliga affärsverk (3 st)
4. AP-fonder (6 st)
5. Sveriges domstolar samt Domstolsverket (84 st)
6. Svenska utlandsmyndigheter (107 st)

En sak man kan fråga sig om dessa är: i vilken utsträckning använder sig myndigheterna https på sina hemsidor? Svaret kommer här!

Från SCB kan man ladda ner en lista över myndigheterna i CVS-format, där en viss kolumn anger myndighetens hemsida. Hemsidan kan man sedan plocka ut genom att använda några standardprogram:

```
cat Forvaltningsmyndigheter.txt | cut -f 12 | tail -n +2 | sort | uniq
```
Efter lite filtrering och borttagning av dubletter, kan man få fram en [lista]({{ "/assets/Sites.txt" | absolute_url }}) över 299 unika webbplatser. När man väl har en hög med hemsidor man vill undersöka kan man använda följade bash-script:

{% highlight bash %}
while read site; do
  data=$(timeout 10 curl -sSv 'https://'$site 2>&1)

  if echo $data | grep -q 'issuer';then
    printf $site': '
    issuer=$(echo "$data" | grep 'issuer' | cut -f 2)
    subject=$(echo "$data" | grep 'subject' | cut -f 2)
    printf "%s %s\n" "$issuer" "$subject"
  else
    echo $site': does not have a certificate'
  fi

done <Sites.txt
{% endhighlight %}> 

Bash-scriptet försöker besöka hemsidan med ett "https://" inlagt först. Om den inte får något svar inom 10 sekunder så avbryter den. Seedan försöker den hitta strängen "issuer" i svaret, och skriver ut lite information om SSL/TLS-certifikatet om strängen hittades. Annars skriver den ut "[sidan]: does not have a certificate". Efter en stund får man fram lite data i det här formatet:

```
www.havochvatten.se:  issuer: C=US,O=DigiCert Inc,CN=DigiCert SHA2 Secure Server CA  subject: C=SE,L=Gotenburg,O=Havs- och vattenmyndigheten,CN=*.havochvatten.se
```

Hur många sidor är det så som inte har något certifikat alls? Svaret är 130 av 299. Dock så räknas "domstol.se" ganska många gånger i den siffran, då de flesta domstolar i sverige körs på underdomäner till domstol.se. Om man ignorerar domstol.se så är siffran 66 av 300. Följande hemsidor använder inte https:

* [www.ap1.se](http://www.ap1.se)
* [www.ap3.se](http://www.ap3.se)
* [www.ap4.se](http://www.ap4.se)
* [www.apfond6.se](http://www.apfond6.se)
* [www.arbetsdomstolen.se](http://www.arbetsdomstolen.se)
* [www.arn.se](http://www.arn.se)
* [www.bfn.se](http://www.bfn.se)
* [www.du.se](http://www.du.se)
* [www.finanspolitiskaradet.se](http://www.finanspolitiskaradet.se)
* [www.fmi.se](http://www.fmi.se)
* [www.formas.se](http://www.formas.se)
* [www.fortv.se](http://www.fortv.se)
* [www.fra.se](http://www.fra.se)
* [www.gih.se](http://www.gih.se)
* [www.gotahovratt.se](http://www.gotahovratt.se)
* [www.harpsund.se](http://www.harpsund.se)
* [www.hogstadomstolen.se](http://www.hogstadomstolen.se)
* [www.hogstaforvaltningsdomstolen.se](http://www.hogstaforvaltningsdomstolen.se)
* [www.hovrattenfornedrenorrland.se](http://www.hovrattenfornedrenorrland.se)
* [www.hsan.se](http://www.hsan.se)
* [www.hyresnamnden.se](http://www.hyresnamnden.se)
* [www.iaf.se](http://www.iaf.se)
* [www.kammarratten.goteborg.se](http://www.kammarratten.goteborg.se)
* [www.kammarrattenisundsvall.se](http://www.kammarrattenisundsvall.se)
* [www.karnavfallsfonden.se](http://www.karnavfallsfonden.se)
* [www.konkurrensverket.se](http://www.konkurrensverket.se)
* [www.kulturanalys.se](http://www.kulturanalys.se)
* [www.kulturradet.se](http://www.kulturradet.se)
* [www.levandehistoria.se](http://www.levandehistoria.se)
* [www.lsh.se](http://www.lsh.se)
* [www.mfd.se](http://www.mfd.se)
* [www.mfof.se](http://www.mfof.se)
* [www.mi.se](http://www.mi.se)
* [www.musikverket.se](http://www.musikverket.se)
* [www.nai.uu.se](http://www.nai.uu.se)
* [www.oks.se](http://www.oks.se)
* [www.radioochtv.se](http://www.radioochtv.se)
* [www.rattshjalp.se](http://www.rattshjalp.se)
* [www.rattshjalpsmyndigheten.se](http://www.rattshjalpsmyndigheten.se)
* [www.rekryteringsmyndigheten.se](http://www.rekryteringsmyndigheten.se)
* [www.rymdstyrelsen.se](http://www.rymdstyrelsen.se)
* [www.sakerhetspolisen.se](http://www.sakerhetspolisen.se)
* [www.sakint.se](http://www.sakint.se)
* [www.sameskolstyrelsen.se](http://www.sameskolstyrelsen.se)
* [www.sfhm.se](http://www.sfhm.se)
* [www.shmm.se](http://www.shmm.se)
* [www.sieps.se](http://www.sieps.se)
* [www.siun.se](http://www.siun.se)
* [www.sjofartsverket.se](http://www.sjofartsverket.se)
* [www.skolfi.se](http://www.skolfi.se)
* [www.sst.a.se](http://www.sst.a.se)
* [www.statensansvarsnamnd.se](http://www.statensansvarsnamnd.se)
* [www.statenssc.se](http://www.statenssc.se)
* [www.statskontoret.se](http://www.statskontoret.se)
* [www.stockholmstingsratt.se](http://www.stockholmstingsratt.se)
* [www.sundsvallstingsratt.se](http://www.sundsvallstingsratt.se)
* [www.sva.se](http://www.sva.se)
* [www.svea.se](http://www.svea.se)
* [www.sverige.fi](http://www.sverige.fi)
* [www.sverigesambassad.dk](http://www.sverigesambassad.dk)
* [www.swedenabroad.rwanda](http://www.swedenabroad.rwanda)
* [www.swedishembassy.ca](http://www.swedishembassy.ca)
* [www.tillvaxtanalys.se](http://www.tillvaxtanalys.se)
* [www.undom.se](http://www.undom.se)
* [www.uniarts.se](http://www.uniarts.se)
* [www.vetansvar.se](http://www.vetansvar.se)

Notera dock att en del hemsidor inte tycks finnas längre, och dyker upp på den här listan av den anledningen.

Vilka certifikat-utgivare de 169 hemsidorna använder går att se i tabellen nedan.

**Utgivare** | **Antal**
DigiCert | 49
TERENA | 36
GlobalSign | 23
Comodo | 21
Let's Encrypt | 14
GoDaddy | 9
GeoTrust | 6
Symantec | 5
ByPass | 1
TeliaSonera | 1
Starfield Technologies | 1
SSL.com | 1
cPanel | 1

Vad kan man dra för slutsatser av allt det här? Det är bra att en stor majoritet av myndigheternas hemsidor använder https, men det finns några som saknar. Det är ganska spritt varifrån myndigheterna får sina certifikat, vilket förmodligen har att göra med att sidorna ligger på olika webbhotell. Det är inte superviktigt att alla sidor har https, men några av de listade sidorna ovan har inloggningar som inte kan betraktas som säkra (vilket även Firefox numera varnar för).
