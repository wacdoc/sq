Futni depon e konfigurimit ops.soft, ekzekutoni `./ssl.sh` dhe do të krijohet një dosje `conf` në **drejtorinë e sipërme** .

## parathënie

SMTP mund të blejë drejtpërdrejt shërbime nga shitësit e cloud, të tilla si:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Shtytje e postës elektronike në renë Ali](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Ju gjithashtu mund të ndërtoni serverin tuaj të postës - dërgim i pakufizuar, kosto e përgjithshme e ulët.

Më poshtë, ne demonstrojmë hap pas hapi se si të ndërtojmë serverin tonë të postës.

## Zgjedhja e serverit

Serveri SMTP i vetë-strehuar kërkon një IP publike me portat 25, 456 dhe 587 të hapura.

Retë publike të përdorura zakonisht i kanë bllokuar këto porte si parazgjedhje dhe mund të jetë e mundur hapja e tyre duke lëshuar një urdhër pune, por në fund të fundit është shumë e mundimshme.

Unë rekomandoj të blini nga një host që i ka këto porte të hapura dhe mbështet vendosjen e emrave të domenit të kundërt.

Këtu, unë rekomandoj [Contabo](https://contabo.com) .

Contabo është një ofrues pritës me bazë në Mynih, Gjermani, i themeluar në vitin 2003 me çmime shumë konkurruese.

Nëse zgjidhni euron si monedhën e blerjes, çmimi do të jetë më i lirë (një server me memorie 8 GB dhe 4 CPU kushton rreth 529 juanë në vit, dhe tarifa fillestare e instalimit është falas për një vit).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Kur bëni një porosi, komentoni `prefer AMD` , dhe serveri me CPU AMD do të ketë performancë më të mirë.

Në vijim, unë do të marr VPS-në e Contabo si shembull për të demonstruar se si të ndërtoni serverin tuaj të postës.

## Konfigurimi i sistemit Ubuntu

Sistemi operativ këtu është Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Nëse serveri në ssh shfaq `Welcome to TinyCore 13!` (siç tregohet në figurën më poshtë), do të thotë që sistemi nuk është instaluar ende. Ju lutemi shkëputeni ssh dhe prisni disa minuta për t'u identifikuar përsëri.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kur shfaqet `Welcome to Ubuntu 22.04.1 LTS` , inicializimi ka përfunduar dhe mund të vazhdoni me hapat e mëposhtëm.

### [Opsionale] Inicializoni mjedisin e zhvillimit

Ky hap është fakultativ.

Për lehtësi, vendosa instalimin dhe konfigurimin e sistemit të softuerit ubuntu në [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Ekzekutoni komandën e mëposhtme për ta instaluar me një klik.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Përdoruesit kinezë, ju lutemi përdorni komandën e mëposhtme dhe gjuha, zona kohore, etj. do të vendosen automatikisht.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo mundëson IPV6

Aktivizo IPV6 në mënyrë që SMTP të mund të dërgojë gjithashtu email me adresa IPV6.

redakto `/etc/sysctl.conf`

Ndryshoni ose shtoni rreshtat e mëposhtëm

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Vazhdoni me [tutorialin contabo: Shtimi i lidhjes IPv6 në serverin tuaj](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Ndryshoni `/etc/netplan/01-netcfg.yaml` , shtoni disa rreshta siç tregohet në figurën më poshtë (skedari i konfigurimit të paracaktuar të Contabo VPS tashmë i ka këto rreshta, thjesht hiqni komentin).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Më pas `netplan apply` për të vënë në fuqi konfigurimin e modifikuar.

Pasi konfigurimi të jetë i suksesshëm, mund të përdorni `curl 6.ipw.cn` për të parë adresën ipv6 të rrjetit tuaj të jashtëm.

## Klononi funksionet e depove të konfigurimit

```
git clone https://github.com/wactax/ops.soft.git
```

## Gjeneroni një certifikatë falas SSL për emrin tuaj të domenit

Dërgimi i postës kërkon një certifikatë SSL për kriptim dhe nënshkrim.

Ne përdorim [acme.sh](https://github.com/acmesh-official/acme.sh) për të gjeneruar certifikata.

acme.sh është një mjet i automatizuar i nënshkrimit të certifikatave me burim të hapur,

Futni depon e konfigurimit ops.soft, ekzekutoni `./ssl.sh` dhe do të krijohet një dosje `conf` në **drejtorinë e sipërme** .

Gjeni ofruesin tuaj DNS nga [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , modifikoni `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Pastaj ekzekutoni `./ssl.sh 123.com` për të gjeneruar certifikata `123.com` dhe `*.123.com` për emrin tuaj të domenit.

Ekzekutimi i parë do të instalojë automatikisht [acme.sh](https://github.com/acmesh-official/acme.sh) dhe do të shtojë një detyrë të planifikuar për rinovim automatik. Ju mund të shihni `crontab -l` , ekziston një rresht i tillë si më poshtë.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Rruga për certifikatën e krijuar është diçka si `/mnt/www/.acme.sh/123.com_ecc。`

Rinovimi i certifikatës do të thërrasë skriptin `conf/reload/123.com.sh` , modifikoni këtë skript, mund të shtoni komanda të tilla si `nginx -s reload` për të rifreskuar memorien e certifikatës së aplikacioneve të lidhura.

## Ndërtoni server SMTP me chasquid

[chasquid](https://github.com/albertito/chasquid) është një server SMTP me burim të hapur i shkruar në gjuhën Go.

Si një zëvendësim për programet e lashta të serverëve të postës si Postfix dhe Sendmail, chasquid është më i thjeshtë dhe më i lehtë për t'u përdorur, dhe është gjithashtu më i lehtë për zhvillimin sekondar.

Run `./chasquid/init.sh 123.com` do të instalohet automatikisht me një klikim (zëvendësoni 123.com me emrin e domenit tuaj dërgues).

## Konfiguro nënshkrimin e postës elektronike DKIM

DKIM përdoret për të dërguar nënshkrime me email për të parandaluar që letrat të trajtohen si spam.

Pasi komanda të ekzekutohet me sukses, do t'ju kërkohet të vendosni rekordin DKIM (siç tregohet më poshtë).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Thjesht shtoni një rekord TXT në DNS tuaj (siç tregohet më poshtë).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Shiko statusin dhe regjistrat e shërbimit

 `systemctl status chasquid` Shiko statusin e shërbimit.

Gjendja e funksionimit normal është siç tregohet në figurën më poshtë

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` ose `journalctl -xeu chasquid` mund të shikojë regjistrin e gabimeve.

## Konfigurimi i kundërt i emrit të domenit

Emri i kundërt i domenit është që të lejojë adresën IP të zgjidhet në emrin e domenit përkatës.

Vendosja e një emri të kundërt të domenit mund të parandalojë që emailet të identifikohen si spam.

Kur të merret posta, serveri marrës do të kryejë analizë të kundërt të emrit të domenit në adresën IP të serverit dërgues për të konfirmuar nëse serveri dërgues ka një emër të vlefshëm domeni të kundërt.

Nëse serveri dërgues nuk ka një emër domeni të kundërt ose nëse emri i kundërt i domenit nuk përputhet me adresën IP të serverit dërgues, serveri marrës mund ta njohë emailin si të padëshiruar ose ta refuzojë atë.

Vizitoni [https://my.contabo.com/rdns](https://my.contabo.com/rdns) dhe konfiguroni siç tregohet më poshtë

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Pas vendosjes së emrit të domenit të kundërt, mos harroni të konfiguroni rezolucionin përpara të emrit të domenit ipv4 dhe ipv6 në server.

## Redakto emrin e hostit të chasquid.conf

Modifiko `conf/chasquid/chasquid.conf` në vlerën e emrit të domenit të kundërt.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Pastaj ekzekutoni `systemctl restart chasquid` për të rifilluar shërbimin.

## Rezervoni konfigurimin në depon e git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Për shembull, bëj kopje rezervë të dosjes conf në procesin tim github si më poshtë

Fillimisht krijoni një depo private

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Futni direktorinë e konf. dhe dorëzojeni në magazinë

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Shto dërgues

vraponi

```
chasquid-util user-add i@wac.tax
```

Mund të shtojë dërgues

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Verifikoni që fjalëkalimi është vendosur saktë

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Pas shtimit të përdoruesit, `chasquid/domains/wac.tax/users` do të përditësohen, mos harroni ta dorëzoni në magazinë.

## DNS shton rekordin SPF

SPF (Sender Policy Framework) është një teknologji verifikimi me email që përdoret për të parandaluar mashtrimin me email.

Ai verifikon identitetin e një dërguesi të postës duke kontrolluar nëse adresa IP e dërguesit përputhet me të dhënat DNS të emrit të domenit që pretendon të jetë, duke parandaluar mashtruesit të dërgojnë email fals.

Shtimi i të dhënave SPF mund të parandalojë që emailet të identifikohen si të padëshiruara sa më shumë që të jetë e mundur.

Nëse serveri juaj i emrit të domenit nuk e mbështet llojin SPF, thjesht shtoni rekordin e tipit TXT.

Për shembull, SPF e `wac.tax` është si më poshtë

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF për `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Vini re se unë kam `include:_spf.google.com` , kjo ndodh sepse më vonë do të konfiguroj `i@wac.tax` si adresën e dërgimit në kutinë postare të Google.

## Konfigurimi DNS DMARC

DMARC është shkurtesa e (Vërtetimi, Raportimi dhe Përputhja e Mesazheve të bazuara në Domen).

Përdoret për të kapur kërcime me SPF (ndoshta shkaktohet nga gabime konfigurimi, ose dikush tjetër pretendon se jeni ju për të dërguar mesazhe të padëshiruara).

Shto regjistrimin TXT `_dmarc` ,

Përmbajtja është si më poshtë

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Kuptimi i secilit parametër është si më poshtë

### p (Politika)

Tregon se si të trajtohen emailet që dështojnë në verifikimin SPF (Sender Policy Framework) ose DKIM (DomainKeys Identified Mail). Parametri p mund të vendoset në një nga tre vlerat:

* asnjë: Nuk është ndërmarrë asnjë veprim, vetëm rezultati i verifikimit i kthehet dërguesit përmes mekanizmit të raportimit të emailit.
* Karantina: Vendosni postën që nuk e ka kaluar verifikimin në dosjen e postës së padëshiruar, por nuk do ta refuzojë drejtpërdrejt postën.
* refuzo: Refuzo direkt emailet që dështojnë në verifikim.

### fo (Opsionet e dështimit)

Përcakton sasinë e informacionit të kthyer nga mekanizmi raportues. Mund të vendoset në një nga vlerat e mëposhtme:

* 0: Raportoni rezultatet e vlefshmërisë për të gjitha mesazhet
* 1: Raportoni vetëm mesazhet që dështojnë në verifikimin
* d: Raportoni vetëm dështimet e verifikimit të emrit të domenit
* s: raportoni vetëm dështimet e verifikimit të SPF
* l: Raportoni vetëm dështimet e verifikimit DKIM

### rua & ruf

* rua (URI raportuese për raportet e përmbledhura): Adresa e emailit për marrjen e raporteve të përmbledhura
* ruf (URI raportuese për raportet mjekoligjore): adresa e emailit për të marrë raporte të detajuara

## Shto regjistrime MX për të përcjellë email-et në Google Mail

Për shkak se nuk mund të gjeja një kuti postare falas të korporatës që mbështet adresat universale (Catch-All, mund të marrë çdo email të dërguar në këtë emër domeni, pa kufizime në prefikset), përdora chasquid për të përcjellë të gjitha emailet në kutinë time postare të Gmail.

**Nëse keni kutinë tuaj postare të biznesit me pagesë, ju lutemi mos e modifikoni MX dhe kapërceni këtë hap.**

Redakto `conf/chasquid/domains/wac.tax/aliases` , vendos kutinë postare të përcjelljes

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` tregon të gjitha emailet, `i` është prefiksi i adresës së emailit të përdoruesit dërgues të krijuar më sipër. Për të përcjellë postën, çdo përdorues duhet të shtojë një rresht.

Pastaj shtoni rekordin MX (unë tregoj direkt adresën e emrit të domenit të kundërt këtu, siç tregohet në rreshtin e parë në figurën më poshtë).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Pasi të përfundojë konfigurimi, mund të përdorni adresa të tjera emaili për të dërguar email te `i@wac.tax` dhe `any123@wac.tax` për të parë nëse mund të merrni email në Gmail.

Nëse jo, kontrolloni regjistrin e chasquid ( `grep chasquid /var/log/syslog` ).

## Dërgoni një email te i@wac.tax me Google Mail

Pasi Google Mail mori postën, natyrisht shpresoja të përgjigjesha me `i@wac.tax` në vend të i.wac.tax@gmail.com.

Vizitoni [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) dhe klikoni "Shto një adresë tjetër email".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Më pas, futni kodin e verifikimit të marrë nga emaili që është përcjellë.

Së fundi, mund të vendoset si adresa e paracaktuar e dërguesit (së bashku me opsionin për t'u përgjigjur me të njëjtën adresë).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Në këtë mënyrë, ne kemi përfunduar krijimin e serverit të postës SMTP dhe në të njëjtën kohë përdorim Google Mail për të dërguar dhe marrë email.

## Dërgoni një email provë për të kontrolluar nëse konfigurimi është i suksesshëm

Fut `ops/chasquid`

Ekzekuto `direnv allow` instalimin e varësive (direnv është instaluar në procesin e mëparshëm të inicializimit me një çelës dhe një goditje është shtuar në guaskë)

pastaj vraponi

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Kuptimi i parametrave është si më poshtë

* përdoruesi: emri i përdoruesit SMTP
* kalim: fjalëkalim SMTP
* tek: marrësi

Mund të dërgoni një email provë.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Rekomandohet të përdorni Gmail për të marrë email testimi për të kontrolluar nëse konfigurimet janë të suksesshme.

### Kriptimi standard TLS

Siç tregohet në figurën më poshtë, ekziston ky bllokim i vogël, që do të thotë se certifikata SSL është aktivizuar me sukses.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Pastaj klikoni "Trego emailin origjinal"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Siç tregohet në figurën më poshtë, faqja origjinale e postës së Gmail shfaq DKIM, që do të thotë se konfigurimi i DKIM është i suksesshëm.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Kontrolloni Received në kokën e emailit origjinal dhe mund të shihni se adresa e dërguesit është IPV6, që do të thotë se edhe IPV6 është konfiguruar me sukses.
