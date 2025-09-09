# SoftwareentwicklungsProzess mit Github

Git erlaubt es allen Entwicklern gleichzeitig Features zu entwickeln.  
Hierzu existieren diverse Techniken Git zu nutzen.
Die bekanntesten hierbei sind:

* Git Flow
* Git(Hub) Flow
* Trunk-based 

Der Github Flow ist eine vereinfachte Version des von der CMI verwendeten Git Flows.  
Für grössere Organisationen ist dieser Workflow völlig ungeeignet, weshalb hier nicht weiter auf diesen eingegangen wird.  
Infos zu diesem Flow lassen sich [direkt bei Github](https://docs.github.com/en/get-started/quickstart/github-flow) finden.

# Terminologie
Die in der CMI verwendete Versionierungs-Software lautet git, bzw. Github in seiner libgit2 Implementation.  
Git bietet hier viele Funktionalitäten.  
In diesem Dokument werden die Folgenden genutzt:

## Branch
Ein Branch bildet einen Zustand des vollständigen Codes ab und fungiert damit quasi als Zeiger auf einen bestimmten [Commit](#commit).  
Releases werden stets aus einem Branch heraus erstellt.  
Ein Branch wird in den folgenden Graphen als Linie dargestellt

## Commit
Ein Commit bildet eine Art Snapshot, der die Änderungen des Entwicklers enthält.  
Wenn ein Release gebaut wird, geschieht dies im Regelfall auf dem aktuellsten Commit des [Branches](#branch).  
Ein Commit wird in den folgenden Graphen als Knoten dargestellt.

## Merge
Als merge wird der Vorgang bezeichnet, der alle Commits ( und somit alle Änderungen ) eines Branches in einen anderen Branch übernimmt.  
Im Regelfall passiert das, sobald die Entwicklung eines Youtrack issues abgeschlossen ist.  
Der initiierende Branch des Merges wird im Anschluss an den Vorgang gelöscht.

## CherryPick
Der CherryPick erlaubt es, einen beliebigen [Commit](#commit) an der Spitze eines anderen [Branches](#branch) zu duplizieren.  
Während der [merge](#merge) es noch erzwingt, dass beide Branches den gleichen Ursprung haben ( zB aus dem gleichen Branch heraus entstanden sind ),  
muss für einen CherryPick diese Bedingung nicht erfüllt sein.  
In der Praxis geht das Brechen dieser Bedingung oft mit MergeKonflikten oder unerwünschten Seiteneffekten einher.  
Mit der steigenden Anzahl an CherryPicks wächst dieses Gefahrenpotential exponentiell.

# Git Flow
Der Gitflow ist der älteste Workflow und ist zusammen mit Git und Github gewachsen.  
In der CMI wird dieser Workflow ebenfalls verwendet und kann dann beispielsweise so aussehen:  

```mermaid
%%{init: { 'logLevel': 'debug', 'theme': 'base', 'gitGraph': {'showBranches': true, 'showCommitLabel':true,'mainBranchName': 'master'}} }%%
gitGraph
        commit
        branch develop
        branch featureA
        branch featureB
        branch v.23.0
        branch release/v23.0.1
        branch v.23.1
        branch release/v.23.1.1
        checkout featureA
        commit
        commit
        checkout featureB
        commit
        commit
        checkout featureA
        commit
        checkout featureB
        commit
        checkout develop
        merge featureA tag: "0.1.0"
        checkout featureB
        commit
        merge develop
        commit
        commit
        checkout develop
        merge featureB tag: "0.1.0"
        checkout v.23.0
        merge develop
        checkout master
        merge v.23.0 tag: "v.23.0.0"
        branch featureC
        merge develop
        commit id:"cc1"
        commit id: "cc2"
        checkout develop
        merge featureC
        checkout release/v23.0.1
        cherry-pick id: "cc1"
        cherry-pick id: "cc2"
        branch featureD
        merge develop
        commit id:"dc1"
        commit id: "dc2"
        checkout develop
        merge featureD
        checkout release/v23.0.1
        cherry-pick id: "dc1"
        cherry-pick id: "dc2"
        checkout release/v.23.1.1
        cherry-pick id: "dc1"
        cherry-pick id: "dc2"
        checkout v.23.0
        merge release/v23.0.1
        checkout master
        merge v.23.0 tag: "v.23.0.1"
        checkout v.23.1
        merge release/v.23.1.1
        checkout master
        merge release/v.23.1.1 tag: "v.23.1.1"
```

Die wichtigsten Branches tragen hierbei den Namen **master** und **develop**.  
Den Branch **master** ( neuerdings auch gerne **main** ) bezeichnet man ebenfalls als "Trunk".  
So wie bei einem echten Baum wird der Baumstumpf nie entfernt, sollte nicht der ganze Baum entfernt werden.  

Umkreist werden diese Branches dann von FeatureBranches, Bugfixbranches, Hotfix branches.  
Zusätzlich existieren allerdings viele trunk-ähnlichen release Branches, die in der CMI dann nach dem Entwicklungsjahr benannt sind ( e.g v.23.X )


## Vorteile
* Entwickler können vollständig dezentralisiert arbeiten und müssen nur hin und wieder ihren die Trunks in ihre Branches mergen
* Einfacher Einstieg nach Git für Junior Entwickler
* Klare Trennung zwischen Entwicklungs und Produktionsständen

## Nachteile
* Die gewaltige Menge an Releases und Patch Branches macht es fast unmöglich noch eine Übersicht zu behalten
* Cherrypicks in die ReleaseBranches werden benötigt, wodurch unerwünschte Seiteneffekte durch picken in der falschen Reihenfolge oder Merge Konfltikte passieren
* Merge Konflikte sind gross und Komplex und benötigen des Öfteren einen "Git Profi", der diese dann löst
* Sehr träge ReleaseFrequenz

## Nachteile ( CMI )
* Features müssen gebackported werden und erzeugen die Gefahr unerwünschter Seiteneffekte
* FeatureBackporting ist oft durch die veränderte Codebasis nicht möglich und benötigt dadurch veränderten Code zwischen dem Trunk, der auf der KV getestet wird,  
  und dem Release Stand, der Schlussendlich an die Kunden verteilt wird
* Features werden vor der Freigabe eines Releases kaum getestet ( hängt allerdings nur indirekt mit dem ReleaseModell zusammen )
* Viele verschiedene ReleaseStände ( Projektreleases !), obwohl der GitFlow traditionell nur einen einzigen Release Branch vorsieht
* Dll patches nehmen selbst den freigegebenen Ständen ihre Homogenität

# Trunkbased Development
Eine Alternative ist hierzu das sog. TrunkBasedDevelopment.  
Der grosse Vorteil dieser Methodik liegt darin, dass deutlich weniger Branches existieren.  
Dadruch werden weniger Cherrypicks und Tests älterer Stände benötigt.  
Als Ergebnis erzeugt dies mehr Zeit für die tatsächliche Entwicklung des Produkts.  

Dieses Verahren wird in der CMI bereits für die **meisten Microservices** ( STS, WebDav, Push, etc.) verwendet.  
Weiterhin wurde zB das **GWR3.0** Projekt nach diesem Stil erfolgreich entwickelt.  

```mermaid
%%{init: { 'logLevel': 'debug', 'theme': 'base', 'gitGraph': {'showBranches': true, 'showCommitLabel':true,'mainBranchName': 'master'}} }%%
gitGraph
        commit
        branch Task1
        branch Task2
        checkout Task1
        commit
        commit
        
        checkout Task2
        commit
        checkout master
        merge Task2 id: "23.4.0" type: HIGHLIGHT
        checkout Task1
        merge master
        commit
        checkout master
        merge Task1 id: "23.5.0" type: HIGHLIGHT
        branch featureA
        branch A/Task1
        branch A/Task2
        checkout featureA
        commit id: "initial"
        checkout A/Task1
        commit
        commit
        checkout A/Task2
        commit
        checkout featureA
        merge A/Task2
        checkout A/Task1
        merge featureA
        checkout master
        branch Task3
        commit
        checkout master
        branch bugfix1
        commit
        checkout master
        merge bugfix1 id: "23.5.1"
        merge Task3 id: "23.6.0" type: HIGHLIGHT
        checkout featureA
        merge master
        checkout A/Task1
        merge featureA
        commit
        checkout featureA
        merge A/Task1
        checkout master
        merge featureA
```

## Vorteile
* Immens weniger Wartungsaufwand durch weniger Branches
* Sehr gute Integration in CI/CD Workflows erlaubt es zB dem PM ein Feature wöhrend der Entwicklung tatsächlich "wachsen zu sehen"
* Schnellere Releases
* Nur selten Konflikte während des [Merges](#merge)
* Keine "Feature Freezes" benötigt
* Entwickler eines Features erhalten etwas mehr Freiheiten während der Entwicklung
* Entwickler müssen weniger Zeit für die Reproduktion von Bugs aufwenden

## Nachteile
* In der Realität nur zusammen mit CI/CD hilfreich ( Ist im CMI System bereits vorhanden )
* Langlebige Feature Branches müssen ca alle 1-2 Wochen gerebased werden um merge Konflikte zu vermeiden
* Entwickler eines Features erhalten etwas mehr Freiheiten während der Entwicklung, wodurch das PM  
  dementsprechend weniger Kontrolle über die Features eines Releases hat

## Nachteile ( CMI )
* Neues Versionierungsmodell muss erst kommuniziert werden
* Möglicherweise fehlt bei manchen Kunden die Akzeptanz für dieses Model

# FeatureFreeze

Besonders der Release zum Jahreswechsel ist in seiner aktuellen Form sehr mühsam und fehleranfällig.  
Gegenwärtig werden **die meisten PRs in den release Branch gepickt**.
Aus diesem Grund können wir hier den Aufwand und die Komplexität minimieren,
indem wir die Ordnung umkehren.
Ein neuer branch **next** wird zum Featurefreeze erstellt.
Alles was **nicht** im neuen Release landen soll, wird nach **next** gemerged.

```mermaid

%%{init: { 'logLevel': 'debug', 'theme': 'base', 'gitGraph': {'showBranches': true, 'showCommitLabel':true,'mainBranchName': 'develop'}} }%%
gitGraph
  commit

  branch feature/A
  checkout feature/A
  commit tag:"Fix thing 1"
  commit tag:"Fix thing 2"
  checkout develop
  merge feature/A tag:"[STX-110] Fix bug before FF"

  branch feature/B
  checkout feature/B
  commit tag:"Implement new feature"
  commit tag:"fix tests"

  checkout develop
  commit id:"26.0" tag:"Feature Freeze"

  branch next
  checkout next
  commit

  checkout feature/B
  commit tag:"Fix thing 1"
  commit tag:"Whoops didnt finish before ff"
  checkout develop
  merge feature/B tag:"[STX-111] new feature"

  checkout next
  merge develop tag:"weekly rebase"

  branch feature/C
  commit tag:"new feature"
  commit
  commit
  checkout next
  merge feature/C tag:"[STX-112] Something new"

  checkout develop
  commit id:"26.0.0" tag:"New Release 26.0.0.1000"

  branch v.26.0
  checkout v.26.0
  commit tag:"New Release 26.0.0.1000"

  checkout develop
  merge next tag:"Implement as usual"

  checkout v.26.0
  branch release/v.26.0.1
  checkout release/v.26.0.1
  commit tag:"[STX-113] Cherrypick fix"
 
```
