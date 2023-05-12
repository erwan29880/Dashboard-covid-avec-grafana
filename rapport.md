# **Un tableau de bord sur la covid**

> Dataviz avec Grafana et mysql


## **Back-end**


### **Persistance**     

Trois volumes sont créés pour mysql :

- le volume mysql pour la persistance des bases de données,   
- le volume mysql_conf pour la sauvegarde des informations de connexion,
- le volume créé dans un dossier accessible de l'OS hôte ; la base de donnée téléchargée 'vaccination.sql' y est copiée.

Un volume est initié au montage de grafana. Le dossier sur la machine hôte doit être créé au préalable.

### **Réseau**    

Le réseau 'mysqlnet' est créé ; tous les containers l'utilisent pour communiquer. 


### **Base de données**   

un container mysql est amorcé pour sourcer le fichier vaccination.sql.


### **Code :**

Téléchargement des images nécessaires :
- docker pull grafana/grafana   
- docker pull mysql  
  
  
création du réseau :   
- docker network create mysqlnet
    
      
création des images hors run :
- docker volume create mysql
- docker volume create mysql_config 
    
   
création du dossier pour grafana (mode root)
- mkdir /var/lib/grafana -p
- chown -R 472:472 /var/lib/grafana
     
  
Amorçage du container mysql sous le nom mysqldb :
> docker run -d -v mysql:/var/lib/mysql \
  -v mysql_config:/etc/mysql -p 3306:3306 \
  -v /home/erwan/exercices/docker/grafana/fichiers:/home \
  --network mysqlnet \
  --name mysqldb \
  -e MYSQL_ROOT_PASSWORD=*** \
  mysql
     
      
Démarrage du container :
- docker start mysqldb
    
        
Import du fichier sql et lancement de l'invite de commande sql :
- docker exec -it mysqldb mysql -u root -p
- source /home/vaccination.sql
    
        
**Vérification :** 

> describe country_vaccinations;
 
![Getting Started](./images/bdd.png)


Démarrage du Container Grafana :
- docker run -d -p 3000:3000 --network mysqlnet --name graf -v /var/lib/grafana:/var/lib/grafana -e "GF_SECURITY_ADMIN_PASSWORD=***" grafana/grafana

Le container est accessible à l'adresse localhost:3000 sur un navigateur web.


## **Front-end**

Sur l'interface graphique de grafana sur localhost:3000, il faut renseigner l'utilisateur ; ici le login est 'admin', le mot de passe est à définir dans le run de containers.  

L'interface propose une connexion à une base de données mysql ; les données suivantes sont renseignées :
- host : mysqldb:3306
- database : vaccinations

La connexion étant opérationnelle, deux graphiques sont créés à partir de deux interfaces 'panels' de grafana à partir de requêtes sql.


### **Pays dans lesquels il y a plus de 100.000.000 de personnes vaccinées :**  

![Getting Started](./images/graphique1.png)

### **Evolution de la vaccination sous forme de série temporelle :** 

![Getting Started](./images/graphique3.png)
