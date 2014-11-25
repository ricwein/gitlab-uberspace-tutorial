# Installation von GitLab 7.5 (mit *https* - yay!)

Diese Anleitung bezieht sich direkt auf die offiziellen Installationsanleitung [hier](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md). Für Uberspace sind jedoch einige Dinge unwichtig, andere zusätzlich nötig. Genauere Beschreibungen sind in der offiziellen Anleitung zu finden. Viele der Befehle aus der offiziellen Anleitung laufen jedoch auch ohne das sudo.


## Abhängigkeiten

### Python

Python wird in einer Version 2.5+ (nicht 3.0+) benötigt.

Python ist auf den Uberspace-Servern bereits aktiviert. Jedoch manchmal noch in der Version 2.4. Prüft das mit dem Befehl `python -V`. Falls noch die alte Version aktiv ist, könnt ihr nach der Anleitung [hier](https://wiki.uberspace.de/development:python) eine neuere aktivieren.


### Git

Git wird in der Version 1.7.10+ benötigt. *Nicht zu verwechseln mit 1.7.1!*

Git ist auch bereits auf den Servern installiert. Prüft mit `git --version` eure Version. Falls sie zu alt ist könnt ihr über Toast eine neuere Version installieren. Sucht dazu [hier](https://code.google.com/p/git-core/downloads/list) eine Version und kopiert den Link zum Tarball. Mit `toast arm [URL zum Tarball]` wird diese installiert und eingerichtet.


### Redis

Installiere Redis wie [hier](https://wiki.uberspace.de/database:redis) beschrieben. Redis akzeptiert auf Uberspace nur Verbindungen zu seinem Socket, was in allen Konfigurationsfiles von GitLab zu beachten ist.


### cmake

Ab Version 7.2 benötigt Gitlab cmake. Dies ist aber auf Uberspace nicht standardmäßig vorinstalliert!

Abhilfe schaffen wir uns wieder mittels toast:

```bash
toast arm cmake
```


### Ruby

Ruby wird in der Version 2.0+ benötigt.

Auf den Uberspace Servern wird standardmäßig eine ältere Version genutzt. [Hier](https://wiki.uberspace.de/development:ruby) wird erklärt wie die neueren zur Verfügung stehenden Versionen aktiviert werden.

```bash
cat <<'__EOF__' >> ~/.bashrc
export PATH=/package/host/localhost/ruby-2.1.1/bin:$PATH
export PATH=$HOME/.gem/ruby/2.1.0/bin:$PATH
__EOF__
```


#### .bashrc vs. .bash_profile
SSH Keys werden innerhalb GitLab über die GitLab Shell verwaltet. Da diese SSH Keys direkt auf das GL Shell Script verweisen wird `.bash_profile` nicht geladen.
Seid ihr der Anleitung auf [Uberspace](https://wiki.uberspace.de/development:ruby) gefolgt, müssen daher die `$PATH` Angaben aus der `.bash_profile` in `.bashrc` verschoben (oder kopiert) werden.


#### Bundler Gem ####

```bash
gem install bundler --user-install --no-ri --no-rdoc
```

`--user-install` sorgt dafür, dass der Gem im Nutzerverzeichnis statt global installiert wird.

**.gemrc**

Alternativ lässt sich diese option auch dauerhaft aktivieren. Dafür einfach `gem: --user-install --no-rdoc --no-ri` in die ~/.gemrc eintragen. Falls die Datei noch nicht existiert erstellen.

```bash
touch ~/.gemrc
echo "gem: --user-install --no-rdoc --no-ri" > ~/.gemrc
```


## System User

Auf den Uberspace Servern gibt es nicht die Möglichkeit einen extra User `git` anzulegen. Der Normale Nutzer geht auch. Jedoch muss das in fast allen Konfigurationsfiles beachtet werden.


## GitLab Shell

Unten die Shellbefehle nach Anleitung.

```bash
cd ~
git clone https://github.com/gitlabhq/gitlab-shell.git -b v2.3.1
cd gitlab-shell
cp config.yml.example config.yml
nano config.yml
```
Wichtig in der `config.yml`:

Änderung aller Pfade von `/home/git/...` zu `/home/[Nutzername]/...`

Außerdem sind folgende Änderungen durchzuführen:

```ruby
user: [Nutzername]
gitlab_url: "https://[Nutzername].[Host].uberspace.de"

#[...]Redis Einstellungen

bin: /usr/local/bin/redis-cli
# Der Pfad sollte identisch mit der Ausgabe von 'which redis-cli' sein

# auskommentieren:
# host: ...
# port: ...

socket: /home/[Nutzername]/.redis/sock
# der Pfad findet sich auch in der Datei '~/.redis/conf' um sicherzugehen.
```

**Für die gitlab_url mit https muss der komplette Pfad inklusive Server und .uberspace.de angegeben werden, damit das Zertifikat auch passt. Auch wenn ihr eine eigene Domain haben solltet!**

Nachdem die Konfigurationdatei geändert ist.

```bash
./bin/install
```


## GitLab

```bash
cd ~
git clone https://github.com/gitlabhq/gitlabhq.git -b 7-5-stable gitlab
cd gitlab

# Clone a few config
cp config/gitlab.yml.example config/gitlab.yml
cp config/unicorn.rb.example config/unicorn.rb
cp config/resque.yml.example config/resque.yml
cp config/database.yml.mysql config/database.yml

cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb #No need to edit this later

#Make a few Direktories and make sure the chmod is right
mkdir /home/[Nutzername]/gitlab-satellites
mkdir tmp/pids/
mkdir tmp/sockets/
mkdir public/uploads

chmod o-rwx config/database.yml

#Muss nicht aber ist nützlich
git config --global user.name "GitLab"
git config --global user.email "gitlab@localhost"
git config --global core.autocrlf input
```


### gitlab.yml Konfiguration

`nano config/gitlab.yml`

```ruby
host: [Nutzername].[Host].uberspace.de
https: true
[...]
user: [Nutzername] # Auskommentierung muss entfernt werden ("#" am Anfang der Zeile entfernen)!
```

**Interessant** für alle, die Gitlab in einer Subdomain (zB. für die [SSH-Keys](#eindeutige-logins-durch-subdomains)) oder in einem Unterordner verwenden wollen:
Mit dem Eintrag `ssh_host` lässt sich für SSH ein anderer Host angeben, als für http (`host`).

Unter 3: Advanced Settings:

alle `/home/git/...` ändern in `/home/[Nutzername]/...`

```ruby
git:
bin_path: /home/[Nutzername]/.toast/armed/bin/git
```

**Der `git_path` sollte stimmen wenn git per Toast installiert wurde. Trotzdem sicherheitshalber per `which git` den Pfad auf seine Richtigkeit überprüfen!**


### unicorn.rb Konfiguration

`nano config/unicorn.rb`

alle `/home/git/...` ändern in `/home/[Nutzername]/...`
`listen "127.0.0.1:8080"...` port in einen noch freien port ändern, z.B. 9765 `listen "127.0.0.1:9765"...`

**Diesen Port am besten merken oder irgendwo notieren. Wir brauche ihn später nochmal!**

> **Um Verwirrungen vorzubeugen:**

> Laut [Uberspace-Wiki](https://wiki.uberspace.de/system:ports) sind Ports nur im Bereich von 61000 bis 65535 erlaubt. Dies bezieht sich aber nur auf Ports, die wir später nach Außen auf dem Server öffnen wollen!
> Wir hingegen wollen den Port aber nur intern nutzen, um [später](#apache-redirect) den Webserver per .htaccess vom externen Port 80 auf unseren lokalen Port weiterzuleiten.
> Es empfiehlt sich also vermutlich ein Port irgendwo zwischen 1024 und 61000 zu nehmen. Eventuell aufpassen, dass man nicht gerade einen von den [well-known Ports](https://de.wikipedia.org/wiki/Liste_der_standardisierten_Ports) erwischt.


### resque.yml Konfiguration

Richtigen redis Zugang einfügen
`production: 'unix:/home/[Nutzername]/.redis/sock'`
Socket ändern, falls er bei der GitLab Shell schon anders war.


### database.yml Konfiguration

Unter `production: ` die MySQL Nutzerdaten eintragen:

```ruby
database: [Datenbank]
username: [Nutzername]
password: [MySQL Passwort] #Wenn es nicht geändert wurde, dann unter ~/.my.cnf zu finden
```


### statische Files ausliefern

`nano config/environments/production.rb`

```ruby
config.serve_static_assets = true
```


## Install Bundle Gems

**Achtung:** [Gabriel Bretschner][1] Hat darauf hingewiesen, dass es auf Servern unter CentOS 5 zu Problemen mit *Charlock Holmes* kommen kann. Die Lösung ist recht einfach und stammt aus dem [Uberspace-Wiki](https://wiki.uberspace.de/development:ruby#charlock_holmes):

```bash
bundle config build.charlock_holmes --with-icu-dir=/package/host/localhost/icu4c
```

**Install:**

```bash
bundle install --deployment --without development test postgres aws
```


## Init Database

```bash
bundle exec rake gitlab:setup RAILS_ENV=production
```

## Precompile assets ##

```bash
bundle exec rake assets:precompile RAILS_ENV=production
```

Dieser Vorgang kann eine Weile dauern...

# Tipp 'yes' zum erstellen der Datenbank
# Wenn ihr fertig seid, sollte so etwas kommen:
Administrator account created:

login.........admin@local.host
password......5iveL!fe

```

**Den Benutzernamen und das Passwort brauchen wir später für den erstmaligen Login noch!**


### GitLab als Uberspace-Service verwalten ###

Gabriel Bretschner hat die ultimative Lösung für Uberspace parat!
In seinem [Blogeintrag][1] erklärt er, wie sich GitLab als Service verwalten lässt.

Eine kurze Anleitung und die Service-Skripte findet ihr in seiner eigenen [GitLab-Installation](https://git.kanedo.net/kanedo/gitlab-uberspace/tree/master/services)

... und als Kopie auch nochmal bei mir:
- [Anleitung](services/Readme.md)
- [sidekiq-Service](services/sidekiq)
- [gitlab-Service](services/gitlab)


## Apache Redirect

In `~/html` oder einem Subdomain-Ordner eine `.htaccess` erstellen und damit füllen

```htaccess
<IfModule mod_rewrite.c>
   RewriteEngine On

   RewriteCond %{HTTPS} !=on
   RewriteCond %{ENV:HTTPS} !=on
   RewriteRule .* https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]

   RewriteBase /
   RewriteRule ^(.*)$ http://127.0.0.1:[Der vorher gewählte Port]/$1 [P]
</IfModule>

RequestHeader set X-Forwarded-Proto https
```

> siehe Beispiel-.htaccess: [.htaccess](_.htaccess)

Die Zeile mit `RequestHeader` behebt den Fehler "Can't verify CSRF token authenticity" beim Login mit https.

## Check Status

```bash
bundle exec rake gitlab:env:info RAILS_ENV=production

bundle exec rake gitlab:check RAILS_ENV=production
```

Falls alles passt, bis auf das nicht kopierte init.d script, dann ..

`bundle exec rake assets:precompile RAILS_ENV=production`


## Fertig

Jetzt sollte erstmal alles funktionieren.


## GitLab-Shell und die SSH-keys

Ein großes Problem bei GitLab und uberspace ist das fehlen eines separaten Users. Loggt ihr euch für gewöhnlich per Key über SSH ein, wird dies nach der Installation der GitLab-Shell nicht mehr möglich sein. Diese blockt nämlich den Shell-Zugriff für alle auf GitLab registrierten Keys! Trotzdem wollen wir aber gerne die GitLab-Pfade und Nutzerrechte zum clonen, pushen etc. benutzen.
Im Grunde genommen gibt es dafür zwei Mögliche Lösungen. Beide haben den Vorteil, dass durch die Nutzung der ssh-config die angelegten Host-Aliases Systemweit zu Verfügung stehen (inklusive SFTP)!

> Zur Nutzung unter Windows kann ich leider keine klare Aussage treffen. Wenn hier jemand Erfahrung hat würde ich mich sehr über Hinweise freuen.


### Separates Keypaar

Ihr legt euch ein separates Key-Paar für den Shellzugriff an.

```bash
ssh-keygen -f ~/.ssh/shellAccess
# optional auch mit custom-Mail/Kommentar:
ssh-keygen -f ~/.ssh/shellAccess -C [aussagekrätiger-Name]@[server]
```

und kopiert den Inhalt des Public-Keys (`.pub`) in die `~/.ssh/authorized_keys` eures Servers.

Anschließend müsst ihr allerdings beim Login noch deutlich machen, mit welchem Keypaar ihr euch einloggen wollt. Das geht am Besten, indem ihr euch in eure `~/.ssh/config` einen Host-*Alias* anlegt, der dann in etwa wie folgt aussehen sollte:

```apache
Host Servername.ShellKey
HostName [Host]
User [Nutzername]
IdentityFile ~/.ssh/shellAccess
IdentitiesOnly yes
```

Nun sollten wir uns direkt mit `ssh Servername.ShellKey` einloggen können!

Der `Host` Eintrag ist dabei als Alias zu verstehen. Ihr könnt im Prinzip benutzen, was euch gefällt.

Alternativ lässt sich ssh auch zur einmaligen Nutzung ohne ssh-config überreden. Das sieht dann in etwa wie folgt aus:

```bash
ssh -i ~/.ssh/shellAccess [Nutzername]@[Host]
```


### per Passwort

Alternative Zwei funktioniert so ähnlich, verzichtet aber auf ein weiteres Keypaar. Stattdessen loggen wir uns old-school mäßig via Passwort ein.
Hierfür muss allerdings ssh konkret der Login mit einem Key verboten werden.

Das geht einmalig mit einem Konstrukt wie:

```bash
ssh -o PreferredAuthentications=keyboard-interactive -o PubkeyAuthentication=no [Nutzername]@[Host]
```

Zum Dauerhaften deaktivieren erstellen wir uns wieder einen Eintrag in die ssh-config `~/.ssh/config`.

```apache
Host Servername.NoKey
HostName [Nutzername].[Host].uberspace.de
User [Nutzername]
PubkeyAuthentication no
```

Der Befehl zum Verbinden lautet nun `ssh Servername.NoKey`.

### Eindeutige Logins durch Subdomains

Gabriel Bretschner hat vor kurzem eine super Ergänzung [veröffentlicht][1].
Er erklärt darin wie sich das Problem mit den SSH-Keys durch eine separate Subdomain für GitLab lösen lässt.

Im Grunde genommen ganz einfach. Ihr lasst GitLab und die GitLab-Shell in einer Subdomain laufen (zB.: git.[Nutzername].[Host].uberspace.de) und erstellt euch einen Host-*Alias* ähnlich wie im obigem [Beispiel](#separates-keypaar).
Natürlich müssen auch die Pfade in den *Configs* angepasst werden.

Besonders wichtig sind hier `gitlab_url` in der *gitlab-shell/config.yml*, sowie `host` in der *gitlab/config/gitlab.yml*.
Damit anschließend die Pfade für SSH noch stimmen sollte auch der Eintrag `ssh_host` in der *gitlab.yml* ergänzt werden ([siehe](#gitlab-yml-konfiguration)).

Dann erstellt ihr euch ein neues Keypaar und den passenden Eintrag in die ssh-config.

```bash
ssh-keygen -f ~/.ssh/shellAccess
```

```apache
Host git.[Nutzername].[Host].uberspace.de
User [Nutzername]
IdentityFile ~/.ssh/shellAccess
IdentitiesOnly yes
```


> **Einziges Problem** an dieser Lösung sind die SSL-Zertifikate.
Uberspace biete selber zwar Wildcard-Zertifikate an, diese sind aber natürlich nicht für eigene Domains oder Sub-Subdomains der User gültig.
> Im Allgemeinen lässt Uberspace zwar [eigene Zertifikate](https://wiki.uberspace.de/webserver:https#nutzung_eigener_ssl-zertifikate) zu. Anbieter wie [StartCom](https://www.startssl.com/) bieten sogar einfache *Class 1* Zertifikate gratis an! Subdomains decken diese jedoch nicht ab (**Ausnahme**: StartSSL Class 1 beinhaltet eine Subdomain!) . Entsprechende *Class 2* Zertifikate kosten bei allen Stellen etwas.

### ControlMaster

Falls Ihr für SSH einen [ControlMaster](https://wiki.uberspace.de/faq?s[]=controlmaster#ich_baue_viele_ssh-verbindungen_auf_und_komm_ploetzlich_nicht_mehr_rein) verwendet, solltet ihr diesen für die entsprechenden Einträge deaktivieren, um eine Verwirrung des Shell-Logins und des GitLab-Logins zu vermeiden!

Dazu einfach `ControlMaster no` noch zum Host in die ssh-config hinzufügen. Fertig!


## Upgraden

### Gitlab-Shell

Manche Gitlab-Upgrades benötigen auch eine aktuellere Version von Gitlab-Shell. Keine Panik, das ist ganz einfach - z.B.: auf 2.3.1:

```bash
cd gitlab-shell
git fetch
git checkout v2.3.1
```

### GitLab

Zuerst sicherheitshalber ein Backup erstellen. Anschließend einfach den Prozess stoppen und das Upgrade-Skript durchlaufen lassen.

```bash
cd gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
svc -d ~/service/run-gitlab && svc -d ~/service/run-sidekiq
./lib/support/init.d/gitlab stop
ruby script/upgrade.rb
```

Änderungen in der production.rb müssen erneut gesetzt werden.

`nano config/environments/production.rb`

```ruby
config.serve_static_assets = true
```

Falls alles erfolgreich verlief kann GitLab nun wieder gestartet werden.

```bash
svc -u ~/service/run-sidekiq && svc -u ~/service/run-gitlab
```

## Upgraden auf 7.x

Die GitLab-Shell ist der einfachste Part:

```bash
cd gitlab-shell
git fetch
git checkout v2.3.1
```

Nun folgt GitLab itself. Im wesentlich habe ich mich dabei an die offizielle Anleitung gehalten: [Docu 6.9 to 7.0](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/update/6.9-to-7.0.md)

Kurz: Backup. Abhängigkeiten installieren. GitLab stoppen. Git pullen. Checkout auf 7.4. Installieren. Daten migrieren. Assets kompilieren und aufräumen. GitLab starten.

*Gitlab 7-2-stable* benötigt cmake als Abhängigkeit. Details [siehe hier](#cmake)

```bash
toast arm cmake
```

Nun Gitlab:

```bash
cd gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
svc -d ~/service/run-gitlab && svc -d ~/service/run-sidekiq
# stoppen noch laufender gitlabinstancen
./lib/support/init.d/gitlab stop

git fetch --all
git checkout 7-5-stable

bundle install --without development test postgres aws --deployment
bundle exec rake db:migrate RAILS_ENV=production
bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production

# siehe unten:
nano config/environments/production.rb

svc -u ~/service/run-sidekiq && svc -u ~/service/run-gitlab
```

wobei für unsere *"Hacks"* wieder gilt:

`nano config/environments/production.yml`

```ruby
config.serve_static_assets = true
```

Nach dem Start schadet ein erneuter Check nicht:

```bash
bundle exec rake gitlab:env:info RAILS_ENV=production
bundle exec rake gitlab:check RAILS_ENV=production
```


## Impressum

Nach einem Tutorial von: [Benjamin Milde](http://kobralab.centaurus.uberspace.de/benni/uberspace/blob/master/install.md)

Aktualisierungen, Fehlerbehebungen und Ergänzungen: [Richard Weinhold](Readme.md)

Support und Feedback von: [Gabriel Bretschner](http://kanedo.net), [Christian Raunitschka](http://ch.rauni.me), [Jan Beck](http://jancbeck.com)

[1]: https://blog.kanedo.net/1925,gitlab-7-0-auf-einem-uberspace-installieren.html
