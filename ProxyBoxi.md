# OpenWrt #


----------


##  Tärkeää tietää aiheesta  ##

OpenWRT = OPEN Wireless RouTer

Käyttöjärjestelmän dokumnetaatio on julkaistu seuraavassa osoitteessa:
https://openwrt.org/docs/start


Edellä mainitut ohjeet ovat tällä hetkellä hyvin suntaa antavia tietyissä tapauksissa sillä **Raspberry Pi 3b +** kehitysalustan toiminnallisuus poikkeaa perinteisestä reitittimestä, joihin OpenWRT on perinteisesti tarkoitettu asennettavaksi.

https://en.wikipedia.org/wiki/OpenWrt
LEDE



Lede/OpenWRT yhdistyneet
Lähde: https://www.theregister.co.uk/2017/05/10/openwrt_and_lede_peace_plan/


OpenWrt is configured using a command-line interface (ash shell)

----------

## Esivalmistelut ##

Ratkaisun käyttöönottoa ennen tarvitsee suorittaa esivalmisteluita ja hankkia sopivat osat.

- Tarvitset laitteen **Raspberry Pi 3b +**, koska se sisältää oman WiFI:n
- Tarvitset myös USB-näppäimistön ja 
- Tarvitset Raspberry Pi yhteensopivan **verkkovirta-adapterin (suositus 5.1V 2.5A)** tai sopivan **akun (suositus 5V 2.4 - 3A)**. Akun saa kiinnittää parhaaksi katsomallaan tavalla, joko käyttämällä micro-USB liitäntää tai vetämällä virtajohdot suoraan GPIO-pinneille Raspberry Pi:ssä. Jälkimmäinen vaihtoehto vaatii tarkkuutta ja tietämystä, joten se on tämän projektin rajojen ulkopuolella.
- **microSD-kortti**, jossa vähintään **UHS-1**, mutta mielellään **UHS-3** nopeusluokitus.  (UHS = Ultra-High Speed suorituskykyluokka, joka korvaa jatkossa vanhemmat nopeusmerkinnät)
- Tarvitset ohjelman, jolla voit purkaa toimivan imagen boottaavaan muotoon **microSD-kortille**. Suosittelemme käytettäväksi **BelanaEtcher**iä tai **Win32 Disk Imager 1.0**
- Tarvitset Raspberry Pi:lle suunnatun OpenWRT:n, joka on ladattavissa osoitteesta: https://downloads.openwrt.org/releases/19.07.0-rc2/targets/brcm2708/bcm2710/
- Kannattaa valita tällä hetkellä tuo **rpi-3-ext4-factory.img.gz**, jonka versio on **19.07.0-rc2**. Tämä on toimivin paketti raportin työstämisen aikaan
- Kun image on purettu microSD-kortille, on yleensä ext-osoi kortilla liian pieni. Helpoiten tämän osion voi laajentaa **gparted**-nimisellä Linux ohjelmalla, mutta mikä tahansa muu osiointiohjelma ja käyttöjärjestelmä kelpaa tähän. On myös teoreettisesti mahdollista uudelleenmäärittää osointi suoraan Raspberry Pi:ssä kun järjestelmä on käytössä, muttta tuolloin prosessi on erittäin monimutkainen ja tuloksia ei voi taata.
- Ohjelmisto on nyt valmis ensimmäiseen käyttnistykseen.
- Muista liittää ennen käynnistystä Raspberry Pi:hin **USB-näppäimistö** ja yhteensopiva **WLAN-tikku**. Yhteensopivuus on hankala määrittää etukäteen ja jouduimme itsetestaamaan usean laitteen ennen sopivan löytymistä, mutta nyrkkisääntönä 5GHz tikut eivät usein toimi tai ylikuumenevat firmware ongelmien takia. Parhaat tulokset saimme WLAN-tikuilla jotka käyttävät RealTek:n **RTL8188CUS** piirisarjaa tai jotain sen variantteja. 

## Käyttöönotto ##

Kun laite käynnistetään ensimmäisen kerran, on syytä jättää ethernet johto irti, koska kaikissa OpenWRT versioissa `dhcp` tai esimääritetty staattinen IP ei toimi oikein, ellei seuraavaa kodia syötetä ensin komentoriville.

Ensin on syytä turvata laitteen käyttö asettamalla pääsalasana root tilille komennolla `passwd`, jonka jälkeen voidaan siirtyä itse verkkoyhteyden asetuksiin seuraavilla koodeilla

    uci set network.lan.proto=dhcp
    uci commit
    /etc/init.d/network restart

Selite:
- ensimmäinen komento `uci set network.lan.proto=dhcp` määrittää verkon `network` käyttämään `lan` lähiverkossa `proto=dhcp` protokollaa DHCP. Tämän korvaa laitteen oman esiasetetun verkko osoitteen `192.168.1.1`.
- `uci commit` suorittaa ja tallettaa edellä ohjelmoidut muutokset ja kuuluu OpenWRT:n perus synksitoiminnallisuuteen vastaavissa säädöissä. 

Toinen vaihtoehtoinen tapa on viimeisen restart komennon korvaava koko laitteen uudelleenkäynnistäminen.

    reboot

Edeltävä koodi on aikaisempaa esimerkkiä varmempi toteutustapa, mutta vie enemmän aikaa. Tämä aika riippuu hyvin paljon käytettävästä ohjelmistoversiosta ja käytettävstä microSD-kortista.

Tässä vaiheessa on syytä kiinnittää ethernet piuhalaitteeseen, olettaen toki, että käytössäsi on normaali reititin, joka jakaa IP-osoitteen laitteelle automaattisesti.

DHCP ja IP-osoitteen tarkistus kun ethernet on liitetty laitteeseen:

    ifconfig

Päivitetään käyttöjärjrestelmän ohjelmisto repository ja asennetaan alustavat tarvittavat paketit seuraavilla komennoilla:

    opkg update
    opkg installa luci
    opkg install nano

Selitteet:
- Opkg = paketinhallintajärjestelmä (lisätitetoja, sekä käyttöohjeet löytyvät osoitteesta https://openwrt.org/docs/guide-user/additional-software/opkg)
- `opkg update` päivittää repositoryn
- `opkg install luci` asentaa luci:n
- LuCI = Lua Configuration Interface, jokatoimii frontend ratkaisuna webselaimen kautta OenWRT:ssä
- `opkg install nano` asentaa ohjelman nimeltä nano, joka on erittäin helppokäyttöinen tekstieditori (käytetään myöhemmin asetusten luomisessa ja muokkaamisessa)


Tässä vaiheessa on mahdollista tehdä vaihtoehtoisia mutta ei vielä pakollisia asioita, kuten esimerkiksi SSL suojauksen käyttöönotto.

    kkkokpk

kfksopfkskfpsjsgisjijgo

## Perusasetukset graafisessa näkymässä ##

Firmware


# Squid Proxy #

Ottamalla käyttöön Squidin proxy:n (Squid 4.6) ProxyBoxi:ssa on loppukäyttäjän tietoturvan kannalta oleellista käyttää ACL-sääntöjä. Tämä vaihtoehto otetaan käyttöön mahdollisen verkkokäytön profiloinnin estämiseksi. Squid:n proxy toimii tässä tapauksessa cache-proxyna (transparent proxy), jolla on oma cache-kansio microSD-kortilla. 

Asennus:

Asennetaan Squid proxy komennolla:

```
opkg install squid
```
Proxy-ohjelmiston asentamisen jälkeen asennetaan vielä cache manageri ja LuCI hallinta moduuli squidille:

```
opkg install luci-app-squid squid-mod-cachemgr
```

Tämän jälkeen säädetään palomuuria CLI-pohjaisesti:

```
nano /etc/config/firewall
```

Tänne laitetaan säännös sallimaan liikenne lähtöportista 80/tcp kohdeporttiin 3128.

```
config redirect 

            option name 'Allow-transparent-Squid' 

            option enabled '1' 

            option proto 'tcp' 

            option target 'DNAT' 

            option src 'lan' 

            option src_ip '!192.168.1.1' 

            option src_dip '!192.168.1.1' 

            option src_dport '80' 

            option dest 'lan' 

            option dest_ip '192.168.1.1' 

            option dest_port '3128' 
```

Seuraavaksi pitää säätää squid.conf:ia.

```
nano /etc/squid/squid.conf
```

Tänne laitetaan seuraavat asetukset:

```
acl localnet src 10.0.0.0/8 
acl localnet src 172.16.0.0/12 
acl localnet src 192.168.0.0/16 
acl localnet src fc00::/7 
acl localnet src 192.168.1.111/16 
acl localnet src fe80::/10 

acl ssl_ports port 443 
acl safe_ports port 80 
acl safe_ports port 21 
acl safe_ports port 443 
acl safe_ports port 70 
acl safe_ports port 210 
acl safe_ports port 1025-65535 
acl safe_ports port 280 
acl safe_ports port 488 
acl safe_ports port 591 
acl safe_ports port 777 
acl connect method connect 
acl blocksites dstdomain "/etc/squid/Ads.conf" 

http_access deny !safe_ports 
http_access deny connect !ssl_ports 
http_access deny localhost manager 
http_access deny manager 
http_access deny to_localhost 
http_access allow localnet 
http_access allow localhost 
http_access deny all 
refresh_pattern ^ftp: 1440 20% 10080 
refresh_pattern ^gopher: 1440 0% 1440 
refresh_pattern -i (/cgi-bin/|\?) 0 0% 0 
refresh_pattern . 0 20% 4320 

access_log none 
cache_log /dev/null 
cache_store_log stdio:/dev/null 

logfile_rotate 0 
logfile_daemon /dev/null 

http_port 3128 intercept 

cache_dir aufs /tmp/squid/cache 600 16 512 

cache_mem 8 MB              
maximum_object_size_in_memory 100 KB 
maximum_object_size 32 MB 
```
Squid:n asetusten säätämisen jälkeen pitää tehdä komennot:

```
Squid –k reconfigure #uudelleen konfiguroi squid:n 

Squid –z #rakentaa tiedosto puun uudelleen squid:lle

squid #käynnistää squidin uudelleen
```
Seuraavaksi täytyy muuttaa /etc/sysupgrade.conf tiedostoa, jotta asetukset Squid:n cache:lle säilyvät. Seuraavat rivit on lisättävä aiemmin mainittuun tiedostoon: 

```
nano /etc/squid/squid.conf 
```

Tämän jälkeen cache proxyn asennus pitäisi olla suoritettu onnistuneesti. Vielä pitää kuitenkin tehdä tiedosto, josta Squid hakee estettävät sivustot. Tämä tiedosto luodaan komennolla: 

```
nano /etc/squid/Ads.conf  
```

Ads.conf:n laitetaan esimerkiksi seuraavat:

```
Mainos.fi 
Testisivu.net 
123.123.123.123 
```
Tämän jälkeen annetaan Ads.conf:lle vielä oikeudet:

```
chmod 755 Ads.conf
```
Tämän jälkeen käynnistetään OpenWrt uudelleen:

```
reboot
```
Nyt pitäisi olla proxy käytössä ja tämän voit kokeilla proxy:ä käytännössä laittamalla seuraavat asetuksen esimerkiksi selaimeen:

```
http proxy IP: 192.168.1.1
Portti: 3128
```




Selite:
- adasdasdas
- dsadsadas
- grghrdhdh
















