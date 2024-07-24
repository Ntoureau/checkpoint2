# **Exercice 1**
## Q1.1
D'abord, il faut vérifier l'adresse IP de chaque machine avec ```ipconfig```

On obtient les adresses suivantes :
![01adressesip.png](https://github.com/Ntoureau/checkpoint2/blob/main/01adressesip.png?raw=true)

On constate que les deux machines ne sont pas sur le même réseau : le 3è octets est différent avec un masque 255 : l'une est sur le réseau 172.1.10 et l'autre sur le réseau 172.16.100.
![02pingechec.png](https://github.com/Ntoureau/checkpoint2/blob/main/02pingechec.png?raw=true)

Pour modifier l'adresse IP du client, il faut aller dans les **Paramètres** > **Réseau et Internet** > **Centre réseau et partage** > **Modifier les paramètres de la carte** puis sélectionner Ethernet (notre carte réseau) et cliquer sur Propriétés. On double clic sur Protocole Internet Version 4 (TCP/IPv4) et on modifie l'adresse IP de la machine pour la mettre dans le même réseau que le serveur, par exemple 172.16.10.50. On coche Valider les paramètres en quittant puis OK, OK, Fermer. Un diagnostic réseau va se lancer, on peut annuler. Notre machine cliente a désormais l'adresse ip 172.16.10.50.

![03parametrescarte.png](https://github.com/Ntoureau/checkpoint2/blob/main/03parametrescarte.png?raw=true)
![04ethernet.png](https://github.com/Ntoureau/checkpoint2/blob/main/04ethernet.png?raw=true)
![05proprietescarte.png](https://github.com/Ntoureau/checkpoint2/blob/main/05proprietescarte.png?raw=true)
![06proprieteipv4.png](https://github.com/Ntoureau/checkpoint2/blob/main/06proprieteipv4.png?raw=true)
![07modifipv4.png](https://github.com/Ntoureau/checkpoint2/blob/main/07modifipv4.png?raw=true)

On vérifie les pings (bien vérifier que l'on indique la nouvelle adresse depuis le serveur) : le ping fonctionne.

![08pingfonctionnel.png](https://github.com/Ntoureau/checkpoint2/blob/main/08pingfonctionnel.png?raw=true)

## Q1.2
Lorsque l'on ping la machine client depuis le serveur avec son nom ```ping CLIENT1 ```, le ping est fonctionnel, mais on constate qu'il se fait sur l'adresse IPv6 au lieu d'IPv4.
Le serveur dispose du rôle DNS, la machine cliente est enregistrée sur le domaine ce qui permet de la ping directement par son nom plutôt que par son adresse IP ce qui facilite les commandes.

![09pingnommachine.png](https://github.com/Ntoureau/checkpoint2/blob/main/09pingnommachine.png?raw=true)

## Q1.3
Pour désactiver le protocole IPv6, on suit le même chemin que pour modifier l'adresse IPv4, mais on décoche Protocole IPv6.

![11modifipv6.png](https://github.com/Ntoureau/checkpoint2/blob/main/11modifipv6.png?raw=true)
Lorsque l'on fait le ping, il se fait désormais en IPv4 au lieu d'IPv6.

![10pingnommachine2.png](https://github.com/Ntoureau/checkpoint2/blob/main/10pingnommachine2.png?raw=true)

## Q1.4
On peut déjà vérifier s'il est configuré sur le serveur. On voit dans le gestionnaire que le rôle est installé. Pour vérifier si une plage d'adresse est configurée, on va dans le menu et on recherche DHCP. Dans la fenêtre qui s'ouvre, on déroule les menus **winserv > IPv4 > Scope [172.16.10.0] vlan 10** et dans **Address Pool**, on peut voir la plage d'adresses distribuées par le dhcp.

![12pooladressesdhcp.png](https://github.com/Ntoureau/checkpoint2/blob/main/12pooladressesdhcp.png?raw=true)

Nous n'aurons donc normalement pas de manipulation à faire sur le serveur pour l'instant.
Sur le client, on retourne sur les modifications de la carte mère comme précédemment, mais au lieu d'employer une adresse statique, on coche "Obtenir une adresse IP automatiquement".

![13activationdhcpclient.png](https://github.com/Ntoureau/checkpoint2/blob/main/13activationdhcpclient.png?raw=true)

## Q1.5
Pour vérifier l'adresse obtenue via le DHCP, on tape ```ipconfig```sur une session powershell du client.
![14nouvelleadresseipclient.png](https://github.com/Ntoureau/checkpoint2/blob/main/14nouvelleadresseipclient.png?raw=true)

Si l'on retourne sur le serveur, en dessous de la plage d'adresses distribuées, on constate qu'il y a une plage d'exclusion de l'attribution automatique des adresses entre 172.16.10.0 et 172.16.10.19, ainsi que des adresses 172.16.10.241 et 172.16.10.254.
![15plageexclusion.png](https://github.com/Ntoureau/checkpoint2/blob/main/15plageexclusion.png?raw=true)

Cela explique pourquoi l'adresse récupérée par notre client est le 172.16.10.20 et pas le 172.16.10.1 

## Q1.6
On peut attribuer une adresse réservée à notre client via le DHCP. Pour cela, il faut déjà récupérer son adresse physique (adresse MAC). On peut l'afficher avec ```ipconfig /all``` (ici le 08-00-27-49-93-6A)

Sur le serveur DHCP, on clique droit sur **Reservations > New Reservation** . Puis on indique le nom de la réservation, par exemple "CLIENT1", l'IP que l'on souhaite réserver, ici 172.16.10.15, puis l'adresse MAC, puis Add.
![16reservationdhcp.png](https://github.com/Ntoureau/checkpoint2/blob/main/16reservationdhcp.png?raw=true)

Sur le client, on libère notre adresse IP avec ```ipconfig /release```et on la renouvelle avec ```ipconfig /renew```
Lorsqu'on vérifie notre nouvelle adresse IP, il s'agit de l'adresse que l'on vient de réserver.

![17adressereservee.png](https://github.com/Ntoureau/checkpoint2/blob/main/17adressereservee.png?raw=true)

## Q1.7
L'intérêt de passer en IPv6 réside dans plusieurs éléments : cela permet de passer outre le problème d'épuisement des adresses IPv4 du fait des adresses de 32 bits, là où les adresses IPv6 en 128 bits en permettent plus (3,4 x 10^38 adresses). Cela permet également une autoconfiguration des adresses via SLAAC juste via l'adresse du réseau donnée par le routeur.

## Q1.8
Non, le serveur DHCP n'est pas obsolète, on a pu voir sur les captures précédentes que désormais, le rôle DHCP peut prendre en compte des plages d'adresses IPv6. Il faudra le configurer de la même manière qu'on l'aurait fait pour de l'adressage IPv4, mais en adaptant les plages.

# **Exercice 2**
## Q2.1
Pour déplacer les fichiers sans utiliser VirtualBox, on ouvre l'explorateur de fichiers, puis on clique droit sur le dossier que l'on souhaite partager (ici Scripts) et on sélectionne Propriétés.

![18proprietesdossier.png](https://github.com/Ntoureau/checkpoint2/blob/main/18proprietesdossier.png?raw=true)

Dans l'onglet Partage (Share) on sélectionne Partager (Share...)

![19partageonglet.png](https://github.com/Ntoureau/checkpoint2/blob/main/19partageonglet.png?raw=true)

On indique ensuite les utilisateurs auxquels on souhaite partager le dossier. On peut sélectionner les utilisateurs ou le rendre disponible à tout le monde en read/write. Puis Partager et enfin Done.

![20choixdestination.png](https://github.com/Ntoureau/checkpoint2/blob/main/20choixdestination.png?raw=true)

Sur le client, on ouvre l'explorateur de fichiers et dans la barre de chemins, on rentre \\172.16.10.10\Scripts (le chemin vers le dossier partagé du serveur) et on récupère les fichiers par un copier/coller.

![21recupdossier.png](https://github.com/Ntoureau/checkpoint2/blob/main/21recupdossier.png?raw=true)

## Q2.2
"Impossible de charger le fichier C:\Scripts\Main.ps1, car l'exécution de scripts est désactivée sur ce système."
Il va falloir modifier la politique d'exécution avec
```Set-ExecutionPolicy RemoteSigned```et cliquer sur oui.

![22remotesigned.png](https://github.com/Ntoureau/checkpoint2/blob/main/22remotesigned.png?raw=true)

Désormais quand on exécute le script, il ouvre une console PowerShell et exécute un script, mais il rencontre une erreur.

On peut aussi ajouter à la liste d'arguments -ExecutionPolicy Bypass. Pour qu'il s'exécute correctement, il faut corriger le chemin indiqué pour qu'il trouve le script AddLocalUsers.ps1

## Q2.3 
Cette  option sert à exécuter le script avec les privilèges administrateurs.

## Q2.4
Elle sert à afficher la fenêtre PowerShell qui va s'ouvrir en plein écran.

## Q2.5
En ouvrant le fichier csv, on voit qu'il dispose d'une ligne d'entête. On peut donc supprimer l'option -Header et ne pas utiliser l'option -Skip 2.

## Q2.6
A la ligne 40, la ¤UserInfo n'utilise pas le champ "Description". On peut le lui ajouter.

## Q2.7 
On peut supprimer à la ligne 27 les en têtes inutilisées.

## Q2.8
A la ligne 50, on ajoute "avec le mot de passe $Pass" -ForegroundColor Green."
Cela permet d'employer la variable contenant le mot de passe avant qu'il soit en Securestring.

## Q2.9
On peut importer le module contenant la fonction avec ```Import-Module "C:\Scripts\Functions.psm1" ```
Ou définir la fonction dans un script externe et utiliser le Dot-source en ajoutant au debut du script "C:\Scripts\Functions.ps1"

## Q2.10
Ajout des lignes
else
    {
        #indique que l'utilisateur existe déjà
        $LogMessage = "Le compte $Prenom.$Nom existe déjà"
        Log -FilePath $LogFile -Content $LogMessage
        Write-Host "Le compte $Prenom.$Nom existe déjà" -ForegroundColor Red
        }

## Q2.11
Ajout d'un "s" car le nom de groupe était mal orthographié (vérifié via la commande net localgroup)

## Q 2.12
Remplacement de la variable $Prenom.$Nom dans le script par "$Name = $Prenom.$Nom", la variable a déjà été créée.

## Q2.13
Dans $UserInfo , on remplace $false à la ligne "PasswordNeverExpires" par $true

## Q2.14
Dans la fonction Random-Password, on remplace $length = 6 par $length = 12

## Q2.15
A la fin du script, on retire la ligne sleep. Avant fin du script, on ajoute deux lignes :
Write-Host "Appuyer sur Entrée pour mettre fin au script"
Read-Host

## Q2.16
Cette fonction va supprimer les accents et convertir les majuscules en minuscules afin de s'assurer qu'aucune erreur n'apparaissent dans les noms.
Les noms "Styrbjörn" ou encore "Anaïs" pourraient par exemple poser problème.

voici le script final

```Write-Host "--- Début du script ---"

Function Random-Password ($length = 12)
{
    $punc = 46..46
    $digits = 48..57
    $letters = 65..90 + 97..122

    $password = get-random -count $length -input ($punc + $digits + $letters) |`
        ForEach -begin { $aa = $null } -process {$aa += [char]$_} -end {$aa}
    Return $password.ToString()
}

Function ManageAccentsAndCapitalLetters
{
    param ([String]$String)
    
    $StringWithoutAccent = $String -replace '[éèêë]', 'e' -replace '[àâä]', 'a' -replace '[îï]', 'i' -replace '[ôö]', 'o' -replace '[ùûü]', 'u'
    $StringWithoutAccentAndCapitalLetters = $StringWithoutAccent.ToLower()
    $StringWithoutAccentAndCapitalLetters
}

$Path = "C:\Scripts"
$CsvFile = "$Path\Users.csv"
$LogFile = "$Path\Log.log"

#Importer le module de la fonction log
Import-Module "$Path\Functions.psm1"

#Importer les champs nécessaires uniquement
#$Users = Import-Csv -Path $CsvFile -Delimiter ";" -Header "prenom","nom","societe","fonction","service","description","mail","mobile","scriptPath","telephoneNumber" -Encoding UTF8  | Select-Object -Skip 2
$Users = Import-Csv -Path $CsvFile -Delimiter ";" -Header "prenom","nom","description" -Encoding UTF8  | Select-Object

foreach ($User in $Users)
{
    $Prenom = ManageAccentsAndCapitalLetters -String $User.prenom
    $Nom = ManageAccentsAndCapitalLetters -String $User.Nom
    $Name = "$Prenom.$Nom"
    If (-not(Get-LocalUsers -Name "$Name" -ErrorAction SilentlyContinue))
    {
        $Pass = Random-Password
        $Password = (ConvertTo-secureString $Pass -AsPlainText -Force)
        $Description = "$($user.description) - $($User.fonction)"
        $UserInfo = @{
            Name                 = "$Prenom.$Nom"
            FullName             = "$Prenom.$Nom"
            Password             = $Password
            Description          = $Description
            AccountNeverExpires  = $true
            PasswordNeverExpires = $true
        }

        New-LocalUser @UserInfo
        Add-LocalGroupMember -Group "Utilisateurs" -Member "$Name"
        Write-Host "L'utilisateur $Prenom.$Nom a été créé avec le mot de passe $Pass" -ForegroundColor Green
    }
    else
    {
        #indique que l'utilisateur existe déjà
        $LogLine = "Le compte $Name existe déjà"
        Log -FilePath $LogFile -Content $LogLine
        Write-Host "Le compte $Name existe déjà" -ForegroundColor Red
        }
}

# pause
Write-Host "Appuyer sur Entrée pour mettre fin au script"
Read-Host
Write-Host "--- Fin du script ---"
```

# **Exercice 3**

## Q3.1
Il s'agit d'un switch ou commutateur. Il permet aux ordinateurs de communiquer entre eux dans un réseau. Il leur permet également d'être relié au routeur B afin de communiquer avec les réseaux extérieurs.

## Q3.2
Il s'agit d'un routeur, c'est la passerelle qu'utilise le réseau 10.10.0.0/16 pour communiquer avec les autres réseaux.

## Q3.3
Il s'agit des interfaces utilisées du routeur. Chacune dispose d'une adresse IP propre.

## Q 3.4
Il s'agit du masque de réseau, il permet de déterminer quelle est la partie du réseau interne. Ici, la plage utilisée sera la plage 10.11.0.0.

## Q 3.5
Il s'agit normalement de sa passerelle vers l'extérieur du réseau. Toutefois, ils ne sont pas sur le même sous réseau (le PC2 est sur le réseau 10.11.0.0 alors que le routeur est sur 10.10.0.0)

## Q 3.6
PC 1
adresse de réseau : 10.10.0.0/16
première adresse : 10.10.0.1
dernière adresse : 10.10.255.254
adresse de diffusion : 10.10.255.255

PC 2
adresse de réseau : 10.11.0.0/16
première adresse : 10.11.0.1
dernière adresse : 10.11.255.254
adresse de diffusion : 10.11.255.255

PC 5
adresse de réseau : 10.10.0.0/15
première adresse : 10.10.0.1
dernière adresse : 10.11.255.254
adresse de diffusion : 10.11.255.255

## Q 3.7
Les PC 1, 3 et 4 sont sur le même sous réseaux ils pourront donc communiquer entre eux. Le PC 5 a un masque qui couvre la plage des 3 pc précédents, ils pourront donc communiquer également, toutefois cela perturbera certains services car le port du switch est configuré pour un réseau particulier.
Toutefois, le PC 2 n'est pas sous le même réseau, il ne pourra communiquer avec aucun.

## Q3.8
Les PC 1, 3, 4 et 5 pourront joindre le réseau 172.16.5.0/24 car ils ont tous accès au routeur.
Le PC 2 ne le pourra pas car pas sur la plage du routeur.

## Q3.9
Il n y aura qu'une incidence mineure : les deux pc peuvent perdre temporairement leur connexion au réseau le temps que la table du switch se mette à jour. Cela n'entrainera aucune modification du réseau.

**Réalisé après 12h30**

## Q3.10
On peut configurer les IP en dynamique par l'installation d'un serveur DHCP. Il atttribuera de nouvelles adresses aux machines (il faudra configurer chaque machine en adressage dynamique sur la carte réseau). On sera certain que toutes les machines branchées à ce switch se verront alors attribuer une adresse sur la plage distribuée par le serveur.

## Q3.11
L'adresse MAC de la source est 00:50:79:66:68:00

## Q3.12
La communication enregistrée a réussi, car la "request" est suivie d'une "reply" donc l'autre appareil a répondu. L'autre appareil est à l'adresse MAC 00:50:79:66:68:03

## Q3.13
Le request correspond au premier appareil 
IP 10.10.4.1
MAC 00:50:79:66:68:00
Nom PC1

Le reply correspoond au second appareil
IP 10.10.4.2
MAC 00:50:79:66:68:03
Nom PC4

## Q3.14
Il s'agit du protocole ARP, Adress Resolution Protocol. Il permet de lier une adresse IP à une adresse MAC.

## Q3.15 
Le matériel A (switch) reçoit les paquets et les redirige vers le bon destinataire. Il va stocker les adresses mac et va les associer à ses ports. C'est donc lui qui va diriger les paquets de PC 1 vers PC 4 et inversement.

## Q3.16
C'est l'adresse IP 10.10.80.2 (PC2) qui initie la communication. 

## Q3.17
C'est un protocole ICMP, un ping. Il permet de tester la connectivité entre deux appareils : le premier envoie un Echo Request, et s'il reçoit un Echo Reply, c'est que la connectivité avec l'appareil destinataire est effective.

## Q3.18
La communication n'a pas réussi, le second paquet indique que l'hôte (PC 3) n'est pas accessible (host unreachable). Les adresses IP des deux appareils ne sont pas sur le même réseau.

## Q3.19
La ligne du paquet n°2 indique que le paquet a été transmis au routeur. En effet, le switch n'ayant pas pu transférer le paquet, celui ci a été envoyé à la passerelle pour qu'elle puisse le transmettre en dehors du réseau. Mais le routeur n'est pas non plus sur le même réseau donc il n'est pas joignable par le PC 2.

## Q3.20
Le switch a relayé les paquets vers la passerelle du réseau car il ne pouvait établir le lien entre les deux appareils. Mais la passerelle n'est pas sur le même réseau que PC 2, donc le paquet ne peut l'atteindre. 

## Q3.21
Le premier matériel est 10.10.4.2, c'est à dire PC 4. Sa destination est 172.16.5.253, le second routeur.

## Q3.22
L'adresse MAC de PC 4 est ca:01:da:d2:00:1c
L'adresse MAC de destination est ca:03:9e:ef:00:38
L'adresse MAC de PC4 n'est plus la même car le paquet est relayé par le routeur, c'est donc l'adresse MAC du routeur qui est affichée, et non celle de PC4.

## Q3.23
Cette communication a été enregistrée entre le routeur B et le second routeur.
