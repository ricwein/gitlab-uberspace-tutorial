# Installation

Diese Anleitung beruht auf der offiziellen Installationsanleitung [hier](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md). Für Uberspace sind jedoch einige Dinge unwichtig, andere zusätzlich nötig. Genauere Beschreibungen sind in der offiziellen Anleitung zu finden.

## Dependencies

### Python

Python wird in einer Version 2.5+ (nicht 3.0+) benötigt.

Python ist auf den Uberspaceservern bereits aktiviert. Jedoch manchmal noch in der Version 2.4. Prüft das mit dem Befehl `python -V`. Falls noch die alte Version aktiv ist, könnt ihr nach der Anleitung [hier](http://uberspace.de/dokuwiki/development:python) eine neuere aktivieren.

### Git 

Git wird in der Version 1.7.10+ benötigt. Nicht zu verwechseln mit 1.7.1, was auf meinem Uberspace die Version war.

Git ist auch bereits auf den Servern installiert. Prüft mit `git --version` eure Version. Falls sie zu alt ist könnt ihr über Toast eine neuere Version installieren. Sucht dazu [hier](http://code.google.com/p/git-core/downloads/list) eine Version und kopiert den Link zum Tarball. Mit `toast arm [URL zum Tarball]` wird diese installiert und eingerichtet.

### Ruby

Ruby wird in der Version 1.9.3+ benötigt.

Auf den Uberspaceservern wird standartmäßig eine ältere Version genutzt. [Hier](http://uberspace.de/dokuwiki/development:ruby) wird erklärt wie die neueren zur Verfügung stehenden Versionen aktiviert werden.

####.bashrc vs. .bash_profile
SSH Keys werden innerhalb GitLab über die GitLab Shell verwaltet. Da diese SSH Keys direkt auf das GL Shell Script verweisen wird .bash_profile nicht geladen. Deshalb müssen die `$PATH` Angaben die den neuen Rubypfad, sowie den Ruby Gem Pfad hinzufügen aus der `.bash_profile` in `.bashrc` kopiert werden.