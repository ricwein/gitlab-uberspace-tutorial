# Installation

Diese Anleitung bezieht sich direkt auf die offiziellen Installationsanleitung [hier](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md). Für Uberspace sind jedoch einige Dinge unwichtig, andere zusätzlich nötig. Genauere Beschreibungen sind in der offiziellen Anleitung zu finden. Viele der Befehle aus der offiziellen Anleitung laufen jedoch auch ohne das sudo.

## Dependencies

### Python

Python wird in einer Version 2.5+ (nicht 3.0+) benötigt.

Python ist auf den Uberspaceservern bereits aktiviert. Jedoch manchmal noch in der Version 2.4. Prüft das mit dem Befehl `python -V`. Falls noch die alte Version aktiv ist, könnt ihr nach der Anleitung [hier](http://uberspace.de/dokuwiki/development:python) eine neuere aktivieren.

### Git 

Git wird in der Version 1.7.10+ benötigt. Nicht zu verwechseln mit 1.7.1, was auf meinem Uberspace die Version war.

Git ist auch bereits auf den Servern installiert. Prüft mit `git --version` eure Version. Falls sie zu alt ist könnt ihr über Toast eine neuere Version installieren. Sucht dazu [hier](http://code.google.com/p/git-core/downloads/list) eine Version und kopiert den Link zum Tarball. Mit `toast arm [URL zum Tarball]` wird diese installiert und eingerichtet.

### Redis

Installiere Redis wie [hier](http://uberspace.de/dokuwiki/database:redis) beschrieben. Redis akzeptiert auf Uberspace nur Verbindungen zu seinem Socket, was in allen Konfigurationsfiles von GitLab zu beachten ist.

### Ruby

Ruby wird in der Version 1.9.3+ benötigt.

Auf den Uberspaceservern wird standartmäßig eine ältere Version genutzt. [Hier](http://uberspace.de/dokuwiki/development:ruby) wird erklärt wie die neueren zur Verfügung stehenden Versionen aktiviert werden.

#### .bashrc vs. .bash_profile
SSH Keys werden innerhalb GitLab über die GitLab Shell verwaltet. Da diese SSH Keys direkt auf das GL Shell Script verweisen wird `.bash_profile` nicht geladen. Deshalb müssen die `$PATH` Angaben die den neuen Rubypfad, sowie den Ruby Gem Pfad hinzufügen aus der `.bash_profile` in `.bashrc` kopiert werden.

#### Bundler Gem

`gem install bundler --no-ri --no-rdoc`

## System User

Auf den Uberspace Servern gibt es nicht die Möglichkeit einen extra User `git` anzulegen. Der Normale Nutzer geht auch. Jedoch muss das in allen Konfigurationsfiles beachtet werden.

## Gitlab Shell

Unten die Shellbefehle nach Anleitung.

```bash
    cd ~
    git clone https://github.com/gitlabhq/gitlab-shell.git -b v1.7.9
    cd gitlab-shell
    cp config.yml.example config.yml
    vim config.yml
```
Wichtig in der `config.yml`:
`user: [Nutzername]`
`gitlab_url: "https://[Nutzername].[Host].uberspace.de"`
Änderung aller Pfade von `/home/git/...` zu `/home/[Nutzername]/...`

Die Redis Einstellungen sind zu ändern. 
`bin: /usr/local/bin/redis-cli` Der Pfad sollte der selbe sein wie die Ausgabe von `which redis-cli`.
`host: ...` & `port: ...` mit einem `# ` auskommentieren
`socket: /home/[Nutzername]/.redis/sock`, der Pfad findet sich auch in der Datei `~/.redis/conf` um sicherzugehen.


```bash
    ./bin/install
```