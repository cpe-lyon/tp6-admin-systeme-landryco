# Exercice 1. Disques et partitions

3. __Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),
et une seconde partition de 3 Go en NTFS (n°7)__<br>
```
sudo fdisk /dev/sdb
n
p
1
entrer
+2G

n
p
2
enter
+3G

t
1
83

t
2
7

w
```

4. __A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.
A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)__<br>
```
sudo mkfs /dev/sdb1
sudo mkfs.ntfs /dev/sdb2
```

5. __Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle
pas sur notre disque ?__<br>
Celui-ci n'est pas encore monté.

6. __Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la
machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des
UUID en raison de l’impossibilité d’effectuer des copier-coller)__<br>
```
/UUID=0537b2bc-7886-447e-afbb-5a40ed4ceb09 / ext4 defaults 0 0
/dev/sdb1       /data   ext4    defaults        0       0
/dev/sdb2       /win    ntfs-3g permissions,users,auto,locale=fr_FR.utf8        0       0
```

8. __Montez votre clé USB dans la VM__<br>
```
sudo mkdir /media/USB
sudo mount -t vfat /dev/sdb1 /media/USB -o [securityoption]
```

9. __Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer
les Additions invité de VirtualBox__<br>

# Exercice 2. Partitionnement LVM

3. __A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en
utilisant la commande pvdisplay.__<br>
`sudo pvcreate /dev/sdb`
```
user@serveur:~$ sudo pvdisplay
  "/dev/sdb" is a new physical volume of "5,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name
  PV Size               5,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               BupfVY-7jkH-h78b-XFC8-t61o-F5KD-N9wcv4
```

4. __A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que
le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.__<br>
`sudo vgcreate vg01 /dev/sdb`
```
user@serveur:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               vg01
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5,00 GiB
  PE Size               4,00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0
  Free  PE / Size       1279 / <5,00 GiB
  VG UUID               5h505n-uTM1-34no-XaZG-0rWq-6Wp2-xWb4em
```

5. __Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.__<br>
`sudo lvcreate -l 100%VG vg01 -n lvData`

6. __Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans
l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.__<br>
`sudo mkfs -text4 /dev/vg01/lvData`

8. __Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de
volumes__<br>
`sudo vgextend vg01 /dev/sdc`

9. __Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas
oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.__<br>
`sudo lvextend -l 100%FREE /dev/vg01/lvData`

`New size given (511 extents) not larger than existing size (1279 extents)`

# Exercice 3. Exécution de commandes en différé : at et cron

1. __Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez
entre temps que la tâche est bien programmée.__<br>
`echo "Réunion" | at 17:03`

2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par
exemple en vous aidant des logs, du manuel...)

Non, pas de service de mail

3. __Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,
l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.__<br>
`*/3 * * * * echo "Il faut réviser l'examen !"`

4. __Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure__<br>
`0 0 * * * echo "Il faut réviser l'examen !"`
`0 0 1 1 * echo "Il faut réviser l'examen !"`
`*/15 * * * * echo "Il faut réviser l'examen !"`

5. __Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18
heures les 1er et 15 du mois :__<br>

6. __Programmez l’exécution d’une commande du lundi au vendredi à 17 heures__<br>
`0 17 * * 1-5 echo "Il faut réviser l'examen !"`

7. __Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un
fichier de log situé dans votre dossier personnel__<br>

8. __Videz votre crontab__<br>
