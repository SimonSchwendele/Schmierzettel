# SoftwareentwicklungsProzess mit Github

Git erlaubt es allen Entwicklern gleichzeitig Features zu entwickeln.  
Hierzu existieren diverse Techniken Git zu nutzen.
Die bekanntesten hierbei sind:

* Git Flow
* Git(Hub) Flow
* Trunk-based 

Der Github Flow ist eine vereinfachte Version des von der CMI verwendeten Git Flows.  
Für grössere Organisationen ist dieser Workflow völlig ungeeignet, weshalb dieser Aufschrieb nicht weiter darauf eingehen wird.

# Git Flow
Der Gitflow ist der älteste Workflow und ist zusammen mit Git und Github gewachsen.  
In der CMI wird dieser Workflow ebenfalls verwendet und sieht hierbei ungefähr so aus:

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

Der Grundbaustein sind hier der trunks **master** und **develop**. 
Umkreist werden diese dann von FeatureBranches, Bugfixbranches, Hotfix branches.

Zusätzlich existieren dann allerdings viele trunk-ähnlichen release branches, die in der CMI dann nach dem Entwicklungsjahr benannt sind ( e.g v.23.X )

## Vorteile
* Entwicklker können vollständig dezentralisiert arbeiten und müssen nur hin und wieder ihren jeweiligen Trunk mergen
* Einfacher Einstieg nach Git
* Klare Trennung zwischen Entwicklungs und Produktionsständen

## Nachteile
* Die gewaltige Menge an Releases und Patch branches macht es fast unmöglich noch eine Übersicht zu behalten
* Cherrypicks in die ReleaseBranches werden benötigt, wodurch unerwünschte Seiteneffekte durch picken in der falschen Reihenfolge oder Merge Konfltikte passieren
* Merge Konflikte sind gross und Komplex und benötigen des Öfteren einen "Git Profi", der diese dann löst
* Sehr träge ReleaseFrequenz

## Nachteile ( CMI )
* Features müssen gebackported werden und erzeugen die Gefahr unerwünschter Seiteneffekte
* FeatureBackporting oft nicht möglich durch eine veränderte Codebasis und benötigt dadurch veränderten Code zwischen KV und Kundenstand
* Features werden vor der Freigabe eines Releases kaum getestet
* Viele verschiedene ReleaseStände ( Projektreleases !), obwohl der GitFlow traditionell nur einen release branch vorsieht
* Dll patches nehmen selbst den freigegebenen Ständen ihre Homogenität

# Trunkbased Development
Eine alternative ist hierzu das sog. TrunkBasedDevelopment.  
Der grosse Vorteil dieser Methodik liegt darin, dass deutlich weniger Branches existieren.  
Dadruch werden weniger cherrypicks und Tests älterer Stände benötigt und erlauben mehr Zeit für die tatsächliche Entwicklung des Produkts.

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

