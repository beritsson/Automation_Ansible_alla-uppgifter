# Automation_Ansible_alla-uppgifter

Alla uppgifter för kursen Automation 

Detta fick vi från studentportalen

Automatisering med Configuration Management program

Kursen kommer lära ut hur man automatiserar miljöer och speciellt konfigurationer av miljöer med hjälp av verktyg för dels automatisering, men också orchestrering.
Den praktiska delen kommer ske i verktyget Ansible

Ansvarig utbildare: Calle Lejdbrandt

E-post: Carl.Lejdbrandt@nackademin.se

Datum för deadline för inlämningar av laborationer: 31/10 2019 kl 16:00

Kurslitteratur: Föreläsningspresentation samt online dokumentation för Ansible

    Kursplan och slides Fil

    Denna presentation är i Open Documents Format. Men ska fungera i MS PowerPoint.

    Den innehåller alla slides, både presentation, betygskriterier, kursplan samt föreläsningsslides.
    Frågor och diskussioner Forum
    Om det finns frågor eller punkter att diskutera, eller något som klassen vill ta upp.
    Chef föreläsning URL
    chef föreläsning PDF Fil
    Puppet presentation Fil

Examinerande laborationer

För denna kurs kommer det enda examinerande momentet vara laborationer.

Laborationerna kommer rikta sig både till de elever som siktar på Godkänd-nivå. Men det finns mycket utrymme för att visa upp Väl Godkänd nivå. 

Laborationerna kommer ske i virtuella maskiner skapade med Vagrant. Till dessa finns det laborationshandledningar som går igenom målet med övningen och vilket betygskriterium den riktar sig mot att uppfylla. Samt vad för moment man måste utföra för att nå de olika betygskriterierna.

Man får arbeta med laborationerna antingen enskilt, eller i grupper om två personer.

Efter varje moment, redovisa för att bli godkänd för det momentet av labben. Om man arbetar i grupp skall båda deltagarna kunna redovisa för labben. Momenten heter "Moment 1" och så vidare. Att sätta upp vagrantmiljön är grundläggande och är inte betygsgrundande.

Vagrantfiler för miljön

Först starta upp maskiner med vagrant

Här är filerna för att ladda ned till er labb-dator. Dessa instruktioner utgår ifrån att du kör Ubuntu eller välldigt snarlik ubuntu dist.

Skapa en katalog på datorn, och ladda ned denna zip-fil. Packa sedan upp zip-filen och en underkatalog med vagrant-filerna kommer skapas.

Se till att paketet qemu-system-common är installerat.

Gå in i katalogen och kör följande kommandon:

vagrant up

Trouble shooting

Vid problem försök köra följande:

Se till att du använder Ubuntu 18.04

    sudo apt purge -y libvirt-dev libvirt-bin vagrant
    sudo apt autoremove
    sudp apt install -y libvirt-dev libvirt-bin vagrant
    vagrant up (från vagrant-katalogen du zippade upp)

Fungerar inte det försöka köra följande och se om det lyckas:

    vagrant box add centos/7
    vagrant plugin install vagrant-libvirt
    vagrant plugin install vagrant-share

I annat fall fråga läraren
Sedan förbered så att ansible kan komma åt maskinerna

För att underlätta för ansible att komma åt de två vagrant maskinerna så bör man köra ett kommando för att lista maskinerna som kända.

När man kört `vagrant up` kommer det komma ett meddelande två gånger som ser ut nästan exakt som nedan:

===================
Detta är IP adressen för webb: 
172.28.128.116
===================

Ta de två ip-adresserna för dina maskiner (webb, och databas) och använd de i kommandot under

ssh-keyscan -t rsa 172.28.128.116 172.28.128.117 >> ~/.ssh/known_hosts

Byt ut 172.28.128.116 och 172.28.128.117 mot de två ip-adresserna som du fått när du körde `vagrant up`

Detta gör att ssh känner igen och litar på att dina vagrant maskiner är just dina vagrant maskiner.

    vagrant-filer

Grundläggande Laborationer (Godkändnivå)

Dessa steg av laborationerna är de mest grundläggande och är för Godkänd nivån av kursen.
Laborationer på Ansible

Det här dokumentet utgår från att Ansible redan finns installerad på den maskin där vi kör, att vi har tillgång till två maskiner där vi kan logga in med ssh, och att vi har en passande ssh-nyckel. Utöver det så börjar vi med en helt tom katalog.
Steg 1: Ansible-konfiguration

För att saker skall bli så lite magiska som möjligt så kommer vi ha precis all konfiguration för vår Ansible-setup i samma katalog. Första steget för att uppnå det är att skapa en fil som heter ansible.cfg. Om det finns en fil med det namnet i katalogen vi står i, så kommer Ansible alltid använda den för sin konfiguration. För våra enkla behov här behöver filen bara inehålla två rader:

[defaults]
inventory = hosts

Filen vi pekar ut på den andra raden är Ansibles inventory-fil. Den berättar vilka maskiner vi kan hantera, så den är rätt central. Här har vi döpt den till hosts. Öppna filen hosts och skriv in följande rader:

[webb]
<ip-adress från vagrant för webb> (utan <>)

[databas]
<ip-adress från vagrant för databas> (utan <>)

[all:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/deploy_id_rsa
ansible_become=yes
ansible_become_user=root
ansible_become_method=sudo

Bitarna mellan hak-parenteser är sektionsnamn. all är en sektion för saker som gäller alla maskiner. Sektionsnamn som slutar på :vars innehåller variabelvärden. Andra sektioner innehåller namn eller IP-adresser till maskiner som vi skall kunna hantera. För de flesta sektioner hittar vi på namnen själva, på det sätt som passar oss. I filen skapar vi först  två sektioner, webb och databas med en IP-adress i varje. Planen är att den ena maskinen skall köra en webb-server och den andra en databas-server. Sedan sätter vi fem variabler för alla maskiner, för att vi skall kunna logga in på dem. 

Med de här två filerna (och den utpekade filen med ssh-nyckeln i) så skall du kunna köra ansible -m ping all och få ett svar som ser ut ungefär så här:

192.168.121... | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.121... | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

Om det står saker som ERROR istället, be läraren om hjälp med att lista ut vad som gick snett.
Steg 2: Playbooks

Nu kan vi komma åt maskinerna vi skall administrera. Nästa steg är att göra något med dem. Det gör vi genom filer som kallas "playbooks".

Skapa en ny fil med något bra namn, till exempel tzone.yml (suffixet är för att playbook-filer skrivs i filformatet YAML). Skriv följande i den:

---
- hosts: all
  tasks:
    - name: Set timezone to UTC.
      timezone:
        name: UTC

Vi går inte in på hela filformatet här. Det vi just skrivit är ett recept på något som Ansible skall se till är sant. Först specificerar vi vilka maskiner som det här receptet gäller (alla). Sedan räknar vi upp en eller flera uppgifter (tasks). Varje uppgift får ett beskrivande namn, som information till inblandade människor. Under namnet anger vi en modul att köra, och vidare under den parametrar till modulen i fråga. Vilka moduler som finns, och vilka parametrar de behöver slår en upp i Ansibles dokumentation. I det här använder vi en modul som heter timezone, och ger som enda parameter namnet på den tidszon som vi vill att maskinerna skall ha.

När vi skrivit klart filen så kör vi den med kommandot ansible-playbook tzone.yml, och med lite tur får vi följande utmatning som resultat:

PLAY [all] ****************************************************************

TASK [Gathering Facts] ****************************************************
ok: [192.168.121...]
ok: [192.168.121...]

TASK [Set timezone to UTC.] ***********************************************
changed: [192.168.121...]
changed: [192.168.121...]

PLAY RECAP ****************************************************************
192.168.121...             : ok=2    changed=1    unreachable=0    failed=0
192.168.121...             : ok=2    changed=1    unreachable=0    failed=0

Notera biten med changed i sektionen "TASKS". Det berättar att Ansible gjorde en förändring. Om du kör samma kommando en gång till nu direkt, så kommer du se att det står ok istället på de raderna. Det indikerar att maskinen i fråga redan var inställd rätt, och Ansible behövde inte göra någon förändring.
Steg 3: Apache

Dags att installera något riktigt! Låt oss börja med webbservern. För det gör vi en ny playbook-fil, webb.yml.

---
- hosts: webb
  tasks:
    - name: Se till att senaste versionen av Apache-paketet är installerat.
      yum:
        name: httpd
        state: latest

Rätt likt förra playbook-filen, men med några signifikanta skillnader. Vi har angett att maskinerna som skall påverkas är webb istället för all. Och vi använder en annan modul, yum. Den modulen är till för att styra det paket-hanteringssystem som används på maskiner med bland annat Linux-distributionerna RedHat Enterprise Linux och CentOS (våra maskiner här kör CentOS). Parametrarna vi ger är namnet på ett paket vi är intresserade av (httpd), och vilket tillstånd vi vill att det skall vara i (senaste versionen installerad). När vi kör filen (ansible-playbook webb.yml) så ser saker ut ungefär som förut, med den inte så lilla skillnaden att den bara agerat på en maskin.

Men vi är inte nöjda bara med att ha installerat Apache-paketet. Vi vill också att Apache skall gå igång när maskinen startar. Det fixar vi genom att lägga till en uppgift till i playbook-filen:

---
- hosts: webb
  tasks:
    - name: Se till att senaste versionen av Apache-paketet är installerat.
      yum:
        name: httpd
        state: latest
    - name: Starta Apache vid boot.
      service:
        name: httpd
        enabled: yes

Modulen service används för att styra tjänster på maskinerna vi kontrollerar. I det här fallet säger vi att tjänsten httpd skall vara aktiverad (vilket innebär att den startas vid boot). Provkör, och se att du nu får två "TASK"-sektioner i utmatningen.

Fast det är ju lite fånigt att behöva starta om maskinen för att Apache skall gå igång efter att den installerats. Lyckligtvis är det enkelt åtgärdat. Vi lägger bara till ytterligare en parameter till service (håll ordning på indenteringen, den är viktig!):

state: started

Sådär. Nu kommer Ansible se till att Apache är installerad, aktiverad vid boot och att den är igång när Ansible körs.

Steg 4: MariaDB

Nästa steg är att installera databasen. Föga förvånande gör vi det genom ytterligare en playbook-fil, du kan själv välja ett lämpligt beskrivande namn. Innehållet blir hemskt lik den för Apache:

---
- hosts: databas
  tasks:
    - name: Se till att senaste versionen av MariaDB-paketet är installerat.
      yum:
        name: mariadb-server
        state: latest

Precis som med Apache vill vi att databasen skall startas vid boot, och direkt efter installation. Det gör vi också nästan exakt likadant som för Apache. Största skillnaden här är nog att själva paktet för MariaDB-servern heter mariadb-server, medan tjänsten som skall startas bara heter mariadb. Sådana detaljer rår inte Ansible över, utan vi måste tyvärr veta en del om de paket vi ber att få installerade. Till slut ser i alla fall hela filen ut så här:

---
- hosts: databas
  tasks:
    - name: Se till att senaste versionen av MariaDB-paketet är installerat.
      yum:
        name: mariadb-server
        state: latest
    - name: Starta MariaDB vid boot.
      service:
        name: mariadb
        enabled: yes
        state: started

Steg 5: En vhost i Apache

En webbserver är ganska tråkig utan någon konfiguration, så låt oss installera lite sådan. Mer specifikt, låt oss lägga till en definition för en VirtualHost i den Apache vi nyss installerade. På CentOS kan vi göra det genom att lägga till en fil med en lämplig bit Apache-konfiguration i en fil i katalogen /etc/httpd/conf.d. Vi kan be Ansible att flytta dit filen åt oss, men vi måste skriva den själva.

Det är praktiskt att ha alla sådana filer i en egen hög, så börja med att skapa en underkatalog kallad "filer" att ha dem i (mkdir filer). Skapa sedan en fil i den katalogen, förslagsvis med namnet "min_vhost.conf". Fyll den med något som ser ut som en konfiguration för en vhost:

<VirtualHost *:80>
    ServerName www.example.org
    DocumentRoot "/var/www/example.org"
</VirtualHost>

(Det får duga som exempel).

Nu skall vi få Ansible att lägga in filen. Vi gör det i webb.yml, eftersom de andra Apache-sakerna ligger där. Där lägger vi till en ny uppgift efter de andra två:

- name: Definiera en vhost.
  copy:
    src: "filer/min_vhost.conf"
    dest: "/etc/httpd/conf.d/min_vhost.conf"

Men vi har ett litet problem här. Apache plockar inte automatiskt upp förändringar i sin konfiguration, så filen vi kopierat dit kommer inte användas förrän nästa gång Apache startar. Vi kan lösa problemet genom att lägga till ytterligare en task efter filkopieringen:

- name: Starta om Apache.
  service:
    name: httpd
    state: restarted

Men om du tar och provkör webb.yml några gånger efter varandra så ser du att den lösningen har en nackdel. Det står changed varje gång du kör. Ansible startar om Apache varenda gång, vare sig det behövs eller ej. Rätt sätt att lösa problemet med att starta om en service enbart om Ansible modifierat dess konfiguration är att använda det som kallas handlers. De kommer vi till senare.

Laborationer forts (Godkändnivå)

Dessa labbar förväntar sig att du är färdig med och förstår vad du gör i steg 1 till och med 5.
Steg 6: En sida att visa i apache

En webbserver utan en hemsida att titta på är också ganska trist. Så låt oss skapa en enkel hemsida och låta ansible installera den på webbservern åt oss.

Skapa en ny fil i mappen filer som vi skapade tidigare i Steg 5. Filen skall heta index.html då det är en vedertagen standard att döpa första filen som presenteras på en hemsida till.

Skriv sedan in följande html-kod för att ha något enkelt att visa:

<html>
<head>
<title>Hello world</title>
<body>
<p>Min första sida på denna webbserver!<br />Hello World!</p>
<p>Länk till <a href="www.google.com">Google!</a></p>
</body>
</html>

Se till att spara filen. Öppna därefter den playbook som ordnat med webbservern innan, den vi kallade webb.yml. Skriv sedan ett nytt stycke sist i den filen som skapar en katalog som heter /var/www/example.org. Modulen du vill använda heter förvirrande nog file och det som gör att det blir en katalog är att sätta state: directoy som argument. Se till att katalogen ägs av användaren apache och gruppen apache. Argumenten för dessa är owner: apache samt group: apache. Läs denna documentation för att få hjälp: https://docs.ansible.com/ansible/2.8/modules/file_module.html#file-module

Detta efterseom vi sagt åt ansible att installera en VirtualHost på vår webbserver. Och den Virtual Hosten har confiugrerats att leta efter hemsidan i katalogen /var/www/example.org.

Under blocket för att skapa en katalog ska du också lägga till att ansible kopierar in den statiska hemsidan vi just skrev. För detta används modulen copy. Det är samma som användes för att kopiera in configurationsfilen för vår VirtualHost. Men här ska det vara src: filer/index.html samt dest: /var/www/example.org/index.html istället för tidigare argument.

spara filen, och kör sedan playbooken med ansible som innan.

Ännu är vi inte helt klara. För att kunna se hemsidan och använda den VirtualHost vi skapade när vi inte har satt upp DNS behöver vi ändra en grej i /etc/hosts på den fysiska kontrollnoden. Så på den fysiska maskinen du kör ansible på. Editera filen /etc/hosts (glöm inte sudo här) och och under raden som heter något med

127.0.0.1 <någonting>

Lägg till en ny rad där du skriver

<ip adress till webb maskinen> example.org

Som till exempel

127.0.0.1     localhost localhost.localdomain
192.168.121.4 example.org

Gå sedan till en webbläsare och skriv in example.org i adress-fältet och så borde din sida komma fram.

Laborationer fort. (upp till VG nivå)

Efter att ha gått igenom laborationerna och förstått vad som hände där, så är det dags att försöka göra lite saker själv. Till exempel lösa de problem som ges nedan.

Flera av övningarna hänvisar till Ansibles dokumentation. Den finns på Ansible dokumentationssida

1. Loopar

På webb-servern vill kunden kunna köra med bra säkerhet, och de tänker köra en webbapplikation skriven i Python. För att uppnå dessa så behöver det installeras ett gäng moduler till Apache. Dessa finns i paketen mod_security, mod_ssl, mod_wsgi, mod_ldap, mod_auth_kerb och mod_auth_gssapi. Det fungerar naturligtvis att skriva en task för varje paket i vår playbook, men det blir en attans massa repetitivt skrivande. För att slippa upprepandet, läs på om loopar i Ansibles dokumentation och skriv en enda task som installerar alla sex paketen.

För bra information om hur loopar fungerar, samt hur man ska välja sina loopar för vad för uppgift man har så läs denna dokumentation

Laborationer fort. (upp till VG nivå)

2. Templates

Laborations-receptet för webb installerade en konfigurationsfil för en vhost i Apache. Kunden vill nu lägga till ett Listen-direktiv i den filen, som säger åt Apache att bara lyssna på den IP-adress som vi loggar in genom. Vi kan naturligtvis bara redigera filen lokalt och skriva in IP-adressen för hand där. Men det är klumpigt, och stor risk för att vi glömmer ändra den dag vi kör mot en annan maskin. Automatisera istället uppgiften genom att använda Ansibles template-modul och Ansibles variabler.

Dokuementation för variabler innehåller också grundläggande information om hur dessa används i templates. Det är reckomenderat att kopiera filer/min_vhost.conf till en ny mapp och nytt filnamn templates/min_vhost.conf.j2 för att visa att det nu är en template. Kom ihåg att ändra din playbook också!

Laborationer fort. (upp till VG nivå)

3. Handlers

I den ursprungliga laborationen startade vi om Apache vid varje körning, med en not om att detta inte är det bästa sättet att göra på. Istället bör något som kallas handlers användas. Nu är det dags att läsa på om dem, och att fixa laborationens playbook så att den gör på det rekommenderade sättet.

Laborationer advancerat (VG nivå)

4. Roller

En funktion som Ansible har för att organisera recept är det som kallas roles, roller. I våra exempel hittills skulle vi till exempel troligen ha en roll för webbservrar, och en roll för databasservrar. Leta reda på biten om roles i Ansible-dokumentationen, läs den, och skriv sedan om de tidigare exemplen så att huvud-playbook-filen bara ser ut så här:

---
- hosts: webb
  roles:
    - webbserver

- hosts: databas
  roles:
    - dbserver


Laborationer fort. (upp till VG nivå)

5. Använd en modul för att konfigurera databasen

Försök använda någon eller några lämpliga modul(er) för att modifiera databas-rollen vi nyss skapat så att den också skapar en databas-instans med namnet "webappdb" och en databas-användare med namnet "webappuser". Sätt att "webappuser" har lösenordet "secretpassword" i databasen.

Databasen vi använder (MariaDB) är en variation av MySQL, och moduler i ansible som kontrollerar MySQL kan också användas utan problem till MariaDB. För att administrera MariaDB via Mysql-modulen behöver databas hosten ha paketet "MySQL-python" installerat.

Laborationer fort. (upp till VG nivå)

6. Sätt ett lösenord utan att använda klartext

Databas-användaren vi skapade ovan skall naturligtvis ha ett lösenord, och det lösenordet skall lika naturligtvis inte stå i klartext i våra Ansible-filer. För att hjälpa oss att åstadkomma detta finns ett verktyg kallat Ansible Vault. Ansible Vault dokumentation.

Uppgiften här är tvådelad. Modifiera först det task från förra uppgiften som hanterar databas-användaren till att sätta ett lösenord, och att det lösenordet hämtas in till task-filen via en Ansible-variabel. När det fungerar, använd Vault för att lagra själva lösenordet i krypterad form.

Laborationer fort. (uppt till VG nivå)

7. Konfigurera databasen till något användbart

Förut sattes det upp en databas, en användare och ett lösenord till användaren. Dock om man bara sätter upp en användare så kommer denne bara kunna ansluta via localhost. Detta är inget bra. Så som första steg, se till att användaren i databasen kan ansluta från den IP adress webb-servern har som INTE står med i den Virtual Host som är uppsatt på webbservern. Kom ihåg att båda maskinerna har två serier av ip adresser, en 192.xxx.xxx.xxx och en 172.xxx.xxx.xxx. IP adressen skall EJ hårdkodas (skrivas fast satt) i playbooken. Använd ansible facts för att hämta ut den!

Nästa steg är att flytta över den sql fil som finns med denna laboration. Den heter här webbapp.sql. Se till att den flyttas över till databas servern och där körs mot webappdb databasen som installerades innan. Städa sedan bort webbapp.sql från filsystemet på databas servern så den inte ligger kvar och skräpar.

När du är klar ska det finnas en databas i MariaDB på databas servern. Den ska innehålla 1 tabell och den tabellen ska innehålla 2 columner.

    sql-filen till labben

Laborationer fort. (upp till VG nivå)

8. Installera en webapp

Utgående från det vi gjort hittills skall vi få igång en webb-applikation. Här finns flera knepigheter att lösa.

För att kunna styra vilka hostar som ansible har hand om som kör denna applikation så skapa en ny roll för denna som kallas "webbapp". Se sedan till att använda denna nya roll på korrekt sätt enligt vad vi hittills lärt oss om ansible.

Till att börja med skall själva applikationen hämtas från https://github.com/initab/demoapp och läggas på någon lagom plats på webb-maskinen (till exempel /var/www/demo). Förslagsvis används ansible-modulen `git` till detta.

För att applicationen ska fungera behövs en del andra paket, samt ett paket från cpan. 

Först måste man lägga till EPEL så att paketen blir tillgänliga. Det kan man göra genom enkelt först installera nedan paket, ange hela sökvägen som name till yum modulen.

http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

State ska vara present.

För att underlätta så inkluderas här paketlista:

Paket för yum:

    "@Development tools"
    perl
    "perl-App*"
    "perl-DBD*"
    "perl-Template*"
    "perl-YAML*"
    "perl-Capture-Tiny*"

Paket för modulen cpanm:

    Dancer

Sedan skall applikationens databasaccess konfigureras. Det görs genom att se till att det i applikationens fil environments/production.yml finns tre rader på följande mönster:

dbconnect: 'dbi:mysql:database=webappdb;host=8.8.8.8'
dbuser: webappuser
dbpass: secret!

IP-addressen på första raden skall vara den som vår databas-server har. Där skall naturligtvis Ansible fylla i den IP-adress som databas-maskinen har fått, snarare än att vi hårdkodar in något. Här gäller också att kopplingen mot databasen skall ske mot den ip adress som användaren "webappuser" får ansluta från, dvs den som valdes i föregående laboration. Samma sak gäller lösenords-raden, där skall lösenordet som vi lagt in i Vault användas.

Sedan skall en virtuell host konfigureras i Apache. Det enklaste här, eftersom vi inte har några prestandakrav, är att köra webappens egen server på en hög port och sedan reläa dit trafiken via Apache med mod_proxy. Konfigurationen för det kan se ut så här, till exempel:

<VirtualHost {{ korrekt_ip_adress }}:80>
    ServerName demoapp.example.org

    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>

Kom här ihåg att ändra "{{ korrekt_ip_adress }}" till den variabeln som bör användas i ansible. `korrekt_ip_adress` är bara alltså inte ett variabelnamn som kommer fungera av sig självt.

För att det skall fungera så måste vi som sagt starta webappens server. Och eftersom rätt skall vara rätt så bör detta göras genom att komnfigurera systemd att starta, stoppa och övervaka den åt oss. För det behöver vi en unit-fil, som läggs på lagom ställe. Filen kan se ut så här:

[Unit]
Description=Ansible demo app.

[Service]
ExecStart=/var/www/demo/bin/app.pl --port=8080 --environment=production

[Install]
WantedBy=multi-user.target

Ett lagom ställe att lägga den på kan vara /etc/systemd/system/demod.service.

CentOS 7 kör med SELinux Enforcing som default. Detta är ur säkerhetsperspektiv bra. Men det förhindrar apache att skapa nya anslutningar. Men det finns dokumentation finns att läsa på https://docs.ansible.com/ansible/latest/modules/seboolean_module.html och variabeln som ni måste sätta till "yes" heter `httpd_can_network_connect`. Se till att den är satt permanent.

När alla dessa bitar är på plats skall det gå att surfa till port 80 på webb-maskinen och få se lite info om de upp till 100 senast inkomna HTTP-requesten.
Trouble shooting

Det kan vara så att applicationen får problem med databasen eftersom ni redan satt upp en databas i laboration 7. Om det skulle ske, logga in som root användaren i databasen:

mysql -u root

och kör sedan nedan kommando:

drop database webappdb;

Efter det, kör en ny play med de nya playbooksen och detta bör ordna upp problemet.
