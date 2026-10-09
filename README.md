# Déploiement GLPI Agent — v1.0.0

Cette publication contient un lanceur Windows et les scripts d’installation Windows, macOS et Linux pour **GLPI Agent 1.21**. **Aucune clé ni configuration client n’est intégrée aux fichiers publics.** Le serveur et le tag sont fournis lors de l’exécution ou par un lanceur client remis en privé.

## Windows : téléchargement automatique

Windows x64, PowerShell 5.1 ou supérieur **64 bits**, en **administrateur** ou **SYSTEM**. Windows x86 et ARM64 ne sont pas pris en charge. Internet doit autoriser GitHub et le serveur GLPI.

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

Le lanceur télécharge **la version v1.0.0**, vérifie son SHA256 intégré puis exécute le script d’installation. Le paquet MSI officiel GLPI est lui aussi téléchargé depuis GitHub et vérifié par SHA256 avant le retrait des anciens agents.

En PowerShell interactif, le Client ID puis le Client Secret GLPI sont demandés si aucune clé n’est fournie ; le secret est saisi de façon masquée. Avec un lanceur client privé contenant sa clé dédiée, aucune saisie n’est requise. Les secrets ne sont pas passés dans les arguments de l’installateur.

L’option `-ExecutionPolicy Bypass` s’applique uniquement au processus lancé ; elle ne modifie pas la politique de la machine. Une stratégie de groupe peut néanmoins interdire l’exécution.

## NinjaOne

- Importer le contenu du lanceur client privé pour Windows, ou celui du lanceur public avec `-Tag` et `-ServerUrl`.
- Windows : SYSTEM, PowerShell 64 bits. Créer les champs **Secure** `glpiOauthClientId` et `glpiOauthClientSecret`, lisibles par les automatisations, si les clés ne sont pas déjà dans le lanceur privé.
- macOS/Linux : importer le script correspondant, exécuter en root et Bash. Fournir `GLPI_TAG` et `GLPI_SERVER_URL`, plus les mêmes champs sécurisés NinjaOne.
- Hors NinjaOne : les scripts acceptent aussi `GLPI_OAUTH_CLIENT_ID` et `GLPI_OAUTH_CLIENT_SECRET` dans l’environnement. Ne pas écrire les clés dans l’historique PowerShell, une ligne de commande ou les journaux.

## macOS et Linux

Les scripts sont disponibles individuellement et dans `glpi-agent-deployment-v1.0.0-sans-cles.zip`. Utiliser `--check-only` pour valider les paramètres avant installation.

macOS prend en charge Intel et Apple Silicon avec l’installation PKG officielle. Linux prend en charge Debian/Ubuntu ou RPM avec systemd. Les installations Homebrew, Snap, AppImage, CPAN ou personnalisées nécessitent une migration spécifique.

## Remplacement et contrôle du pilote

FusionInventory est retiré par défaut. Les anciens GLPI Agent standard sont remplacés ; la même version peut être réparée, et une version plus récente est conservée. Windows accepte `-KeepFusionInventory` pour conserver exceptionnellement FusionInventory. Les configurations anciennes sont sauvegardées dans un dossier réservé aux administrateurs/root. Aucun redémarrage forcé.

Ces scripts remplacent les agents des appareils ; ils ne migrent pas les tickets ni l’historique du serveur GLPI. Si l’installation échoue après retrait, l’ancien logiciel n’est pas réinstallé automatiquement.

**Avant généralisation, tester un poste par système** et contrôler dans GLPI : nom du poste, numéro de série, entité attendue et nouvelle date d’inventaire. Vérifier le service après un redémarrage planifié. Un code local 0 ne prouve pas la réception dans GLPI.

Syntaxe et simulations de remplacement contrôlées ; aucun déploiement réel de parc n’est certifié par cette release.

## Intégrité et téléchargements

`SHA256SUMS.txt` contient les empreintes des scripts, de la notice et de l’archive. Le lanceur contient l’empreinte attendue du script Windows qu’il exécute. Il utilise une release précise et ne suit pas automatiquement `latest`.

[Téléchargements de cette version](https://github.com/axyrios/glpi-agent-deployment-releases/releases/tag/v1.0.0). Les lanceurs avec clés sont remis séparément par un canal privé et ne doivent pas être publiés ici.
