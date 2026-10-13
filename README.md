# Déploiement GLPI Agent — v1.0.3

Cette publication contient un lanceur Windows et les scripts d’installation Windows, macOS et Linux pour **GLPI Agent 1.21**. **Aucune clé ni configuration client n’est intégrée aux fichiers publics.** Le serveur et le tag sont fournis lors de l’exécution ou par un lanceur client remis en privé.

**Correction v1.0.3 :** tous les modules disponibles de l’agent sont installés et autorisés : inventaire, déploiement de paquets, collecte, découverte et inventaire réseau, ESX et inventaire distant. Windows utilise `ADDLOCAL=ALL`, Linux `--type=all`, macOS le PKG officiel complet. La configuration `Inventory,...` laisse toutes les tâches disponibles au service. Une vérification locale refuse une installation dont un module requis manque.

**Correction conservée de v1.0.2 :** les messages informatifs de l’agent sur la sortie d’erreur native n’interrompent plus l’inventaire sous Windows PowerShell. Le script contrôle ensuite le code de sortie réel et refuse toujours les erreurs de transfert ou d’authentification. Les sauvegardes, TLS et SHA256 restent actifs.

## Windows : téléchargement automatique

Windows x64, **Windows PowerShell 4.0 ou 5.1**, en **64 bits**, en **administrateur** ou **SYSTEM**. Windows x86 et ARM64 ne sont pas pris en charge. Internet doit autoriser GitHub et le serveur GLPI.

Cette version utilise une syntaxe compatible PowerShell 4.0 et 5.1. Aucune mise à jour de PowerShell ou de .NET n’est effectuée par les scripts. Le contrôle SHA256, TLS 1.2 et les protections des identifiants restent actifs.

1. Télécharger `install_from_github.ps1` dans les fichiers de cette release.
2. Lancer PowerShell administrateur. Remplacer les exemples ci-dessous par le tag et l’URL du client.

```powershell
$lanceur = "$env:USERPROFILE\Downloads\install_from_github.ps1"
powershell.exe -NoProfile -ExecutionPolicy Bypass -File $lanceur -Tag CLIENT -ServerUrl "https://glpi.example.org/plugins/glpiinventory/" -CheckOnly
```

`-CheckOnly` valide les paramètres : aucun téléchargement, changement ou inventaire.

Pour télécharger et vérifier le script sans installer et sans demander de clé :

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File $lanceur -Tag CLIENT -ServerUrl "https://glpi.example.org/plugins/glpiinventory/" -DownloadOnly
```

Pour installer :

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File $lanceur -Tag CLIENT -ServerUrl "https://glpi.example.org/plugins/glpiinventory/"
```

Le lanceur télécharge **la version v1.0.3**, vérifie son SHA256 intégré puis exécute le script d’installation. Le paquet MSI officiel GLPI est lui aussi téléchargé depuis GitHub et vérifié par SHA256 avant le retrait des anciens agents.

En PowerShell interactif, le Client ID puis le Client Secret GLPI sont demandés si aucune clé n’est fournie ; le secret est saisi de façon masquée. Avec un lanceur client privé contenant sa clé dédiée, aucune saisie n’est requise. Les secrets ne sont pas passés dans les arguments de l’installateur.

L’option `-ExecutionPolicy Bypass` s’applique uniquement au processus lancé ; elle ne modifie pas la politique de la machine. Une stratégie de groupe peut néanmoins interdire l’exécution.

## NinjaOne

- Importer le contenu du lanceur client privé pour Windows, ou celui du lanceur public avec `-Tag` et `-ServerUrl`.
- Windows : SYSTEM, PowerShell 64 bits. Créer les champs **Secure** `glpiOauthClientId` et `glpiOauthClientSecret`, lisibles par les automatisations, si les clés ne sont pas déjà dans le lanceur privé.
- macOS/Linux : importer le script correspondant, exécuter en root et Bash. Fournir `GLPI_TAG` et `GLPI_SERVER_URL`, plus les mêmes champs sécurisés NinjaOne.
- Hors NinjaOne : les scripts acceptent aussi `GLPI_OAUTH_CLIENT_ID` et `GLPI_OAUTH_CLIENT_SECRET` dans l’environnement. Ne pas écrire les clés dans l’historique PowerShell, une ligne de commande ou les journaux.

## macOS et Linux

Les scripts sont disponibles individuellement et dans `glpi-agent-deployment-v1.0.3-sans-cles.zip`. Utiliser `--check-only` pour valider les paramètres avant installation.

macOS prend en charge Intel et Apple Silicon avec l’installation PKG officielle. Linux prend en charge Debian/Ubuntu ou RPM avec systemd. Les installations Homebrew, Snap, AppImage, CPAN ou personnalisées nécessitent une migration spécifique.

## Correction des installations limitées à l’inventaire

Télécharger le **nouveau** lanceur de cette publication : les anciens lanceurs restent liés à leur ancienne release et ne changent pas automatiquement. Relancer la nouvelle version en administrateur/SYSTEM sur les agents déjà en 1.21 ajoute les modules manquants sous Windows ; macOS et Linux réinstallent le paquet complet après sauvegarde. Les identifiants doivent être fournis par les mêmes champs sécurisés ou le lanceur privé.

Le contrôle immédiat force seulement un inventaire. Ensuite, les autres tâches sont exécutées selon la configuration et les tâches planifiées dans GLPI Inventory. Le serveur HTTP local reste désactivé : les agents interrogent GLPI périodiquement, sans réveil immédiat par le port 62354 ni ouverture du pare-feu. WakeOnLan n’est pas une tâche supportée de l’agent Windows 1.21.

Contrôler dans la fiche de l’agent GLPI les **Tâches supportées**, en particulier « Déploiement de paquets ». La présence des modules ne certifie pas une installation de paquet réussie : effectuer ensuite un pilote de déploiement. Aucun poste existant n’a été corrigé pendant la publication.

## Remplacement et contrôle du pilote

FusionInventory est retiré par défaut. Les anciens GLPI Agent standard sont remplacés ; la même version peut être réparée, et une version plus récente est conservée. Windows accepte `-KeepFusionInventory` pour conserver exceptionnellement FusionInventory. Les configurations anciennes sont sauvegardées dans un dossier réservé aux administrateurs/root. Aucun redémarrage forcé.

Ces scripts remplacent les agents des appareils ; ils ne migrent pas les tickets ni l’historique du serveur GLPI. Si l’installation échoue après retrait, l’ancien logiciel n’est pas réinstallé automatiquement.

**Avant généralisation, tester un poste par système** et contrôler dans GLPI : nom du poste, numéro de série, entité attendue et nouvelle date d’inventaire. Vérifier le service après un redémarrage planifié. Un code local 0 ne prouve pas la réception dans GLPI.

Syntaxe et simulations de remplacement contrôlées ; aucun déploiement réel de parc n’est certifié par cette release.

## Intégrité et téléchargements

`SHA256SUMS.txt` contient les empreintes des scripts, de la notice et de l’archive. Le lanceur contient l’empreinte attendue du script Windows qu’il exécute. Il utilise une release précise et ne suit pas automatiquement `latest`.

[Téléchargements de cette version](https://github.com/axyrios/glpi-agent-deployment-releases/releases/tag/v1.0.3). Les lanceurs avec clés sont remis séparément par un canal privé et ne doivent pas être publiés ici.
