# Bundles give you Services

Tous les objets Service sont stockés dans un objet appelé **container**.  
Chaque Service a son nom interne (comme les routes).  
Les **bundles** correspondent au système de plugins de Symfony, notamment chargé de placer les 
Services dans le container. Par exemple, `MonologBundle` est le bundle chargé de fournir 
le service de logs.
En ouvrant le fichier `start/config/bundles.php`, on peut voir que notre application dispose 
actuellement de 7 bundles (donc 7 plugins de Services). Le système de Recipes met automatiquement 
à jour ce fichier lors d'un require d'un bundle.  


