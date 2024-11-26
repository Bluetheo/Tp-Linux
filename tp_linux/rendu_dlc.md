# DLC

# Étape 1 : Analyse avancée et suppression des traces suspectes

1. Rechercher des utilisateurs récemment ajoutés :

Identifiez les utilisateurs ajoutés récemment en inspectant les logs de sécurité.
````
sudo grep "new user" /var/log/secure
````

2. Trouver les fichiers récemment modifiés dans des répertoires critiques :

Analysez les fichiers modifiés dans /etc, /usr/local/bin ou /var au cours des 7 derniers jours :
````
sudo find /etc /usr/local/bin /var -type f -mtime -7
````

3. Lister les services suspects activés :

Vérifiez tous les services activés au démarrage pour identifier ceux qui ne devraient pas être présents :
````
sudo systemctl list-unit-files --state=enabled
````

4. Supprimer une tâche cron suspecte :

Identifiez et supprimez une tâche cron malveillante ajoutée à un utilisateur spécifique, par exemple attacker :
````
sudo crontab -u attacker -r
````

# Étape 2 : Configuration avancée de LVM

1. Créer un snapshot du volume logique 

````
[root@localhost ~]# sudo lvdisplay
````
````
[root@localhost ~]# sudo lvcreate --size 1G --snapshot --name secure_data_snapshot /dev/vg_secure/secure_data
````

2. Tester le snapshot 
````
[root@localhost ~]# sudo mkdir /mnt/secure_data_snapshot
````
````
[root@localhost ~]# sudo mount /dev/vg_secure/secure_data_snapshot /mnt/secure_data_snapshot
````
````
[root@localhost ~]# ls /mnt/secure_data_snapshot
````

3. Simuler une restauration
````
[root@localhost ~]# sudo rm /mnt/secure_data/sensitive1.txt
````
````
[root@localhost ~]# sudo cp /mnt/secure_data_snapshot/sensitive1.txt /mnt/secure_data/
````
````
ls /mnt/secure_data/
````

# Étape 3 : Renforcement du pare-feu avec des règles dynamiques
