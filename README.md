# XML TV Fr

XML TV Fr est un service permettant de récupérer un guide des programmes au format XMLTV.


# Prérequis

PHP >=7.1.0 avec les extensions
 - curl
 - zip
 - mbstring
 - xml
 - json

# Configuration

Cette partie va vous permettre de configurer XML TV Fr.

## Liste des chaines (channels.json)

La liste des chaines doit être indiquée dans le fichier channels.json au format JSON. Chaque chaine correspond à l'ID d'une chaine (Exemple : France2.fr) présente dans les fichiers de chaines par services (dossier channels_by_providers).
La structure d'un item se fait comme ceci :

    "IdDelaChaine":{"name":"Nom de la chaine","icon":"http://icone de la chaine","priority":["Service1","Service2"]}
Les champs name, icon et priority sont optionnels. 
Le champ priority donne un ordre de priorité différent de celui par défaut en indiquant les noms des services (nom des classes dans le dossier classes). Dans l'exemple, Service1 sera appelé en premier et Service2 ne sera appelé que si Service1 échoue. Par exemple si on met en priorité Télérama puis Orange, Télérama sera lancé. Si aucun programme n'est trouvé sur Télérama, Orange est lancé, sinon on continue. Si aucun programme n'est trouvé sur tous les services, la chaine est indiquée HS pour le jour concerné.

## Configuration du programme (config.json)

Le fichier config.json est au format JSON. 
```json
{
  "days" : 1, // Nombre de jours de l'EPG
  "cache_max_days": 8, // Nombre de jours de cache
  "output_path": "./xmltv", // Chemin de destination du XML final
  "time_limit": 0, // Temps d'éxécution max du script (0=illimité)
  "memory_limit": -1, // Quantité de mémoire vive max (-1=illimité)
  "delete_raw_xml": false, // Supprimer le XML brut après génération (true|false)
  "enable_gz": false, // Activer la compression gz (true|false)
  "enable_zip": true // Activer la compression zip (true|false)
}
```

# Lancer le script
Pour démarrer la récupération du guide des programmes, lancez cette commande dans votre terminal (dans le dossier du programme).

    php script_all.php
    
# Générer le fichier channels.json
Il est possible de générer depuis votre navigateur le fichier channels.json. Pour cela, placez vous dans le dossier de travail du programme et lancez cette commande

    php -S localhost:8080
Note : le port 8080 peut être changé par un autre.

Ouvrez ensuite dans votre navigateur http://localhost:8080/tools/ (port à modifier en fonction de celui indiqué dans la commande au dessus).
# Sortie

## Logs
Les logs sont stockés dans le dossier logs au format JSON. Les derniers logs sont accessibles via le navigateur à l'adresse http://localhost:8080/tools/logs.php (à condition d'avoir lancé la commande précédente).
## XML TV
Les fichiers de sorties XML sont stockés dans le dossier xmltv au format XML, ZIP et GZ.
Cette commande indiquera si le dernier fichier XML généré est valide.

# Ajouter des services

Il est possible d'ajouter des services autres que ceux fournis. Pour cela, il faut ajouter une classe dans le dossier classes qui implémente la classe Provider et étende la classe AbstractProvider. 

Le constructeur aura le chemin des XML temporaires d'indiqué.

La méthode `getPriority()` renverra un flottant de préférence entre 0 et 1 pour indiquer la priorité par rapport à d'autres services (comparez les valeurs des autres scripts pour vous situer).

La méthode   `constructEPG(channel,date)` construira un fichier XML pour une chaine à une date donnée. Elle retourne `true` si la tâche s'est déroulée avec succès, sinon `false`.

L'instance de chaque `Provider` par date possédera un attribut `channelObj` étant une instance de la classe Channel (si `constructEPG` appelle la classe parente). A cette instance de classe `Channel`, vous pourrez ajouter des programmes (instances de la classe `Program`) avec la méthode `addProgram($start, $end)` (`$start` et `$end` étant des timestamp UNIX) et sur l'instance de chaque programme, vous pourrez définir les infos telles que le titre, les catégories, ... Une fois l'ajout des programmes terminé, il suffira d'appeller la méthode `save()` de l'attribut `channelObj` pour enregistrer le fichier XML pour la chaine et la date en question.
Exemple :
```php

    function constructEPG($channel, $date)
    {
        parent::constructEPG($channel, $date);
        foreach($results as $result) {
            $program = $this->channelObj->addProgram(strtotime($result['start']), strtotime($result['end']));
            $program->addTitle($result["title"], "en"); // argument langue optionnel, par defaut = "fr"
            $program->setIcon("myIconUrl");
            $program->addCategory(...)
            $program->addSubtitle(...)
            ...
        }   
        return $this->channelObj->save(); // sauvegarde le programme de la journée en XML
    }
```

Attention, le nom de la classe du service doit correspondre à son nom de fichier. Bien que PHP, contrairement à Java autorise des noms différents, le programme ici ne le permet pas.

