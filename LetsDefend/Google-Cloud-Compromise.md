Mon analyse : Compromission Google Cloud (LetsDefend)

Le scénario en deux mots

Je me suis penché sur un cas de compromission chez une startup.
Un attaquant a réussi à récupérer une identité Google Cloud pour accéder à un bucket contenant du code sensible.


Ma démarche d’investigation

En analysant les logs, j’ai pu reconstituer le déroulement de l’attaque.

La cible
Le service account `cloud-storage-helper` a été compromis.
C’est un cas assez classique : ce type de compte est souvent utilisé pour automatiser des tâches et peut avoir des permissions trop larges.
<img width="737" height="276" alt="image" src="https://github.com/user-attachments/assets/d2c41bda-2af8-4c9a-b63b-12117475a580" />



L’alerte
L’activité provenait de l’IP `178.132.108.38`, localisée en Roumanie.
Ce qui m’a immédiatement paru suspect, c’est qu’il s’agit d’une IP de Data Center. Ce n’est pas le comportement attendu d’un utilisateur légitime.
<img width="761" height="251" alt="image" src="https://github.com/user-attachments/assets/27b63ecb-ef03-4b17-a99b-d668a0d0cb7f" />
<img width="667" height="392" alt="image" src="https://github.com/user-attachments/assets/b263f70f-baa4-40dc-9434-59410476f5f0" />




Les actions de l’attaquant
L’attaquant a d’abord tenté d’activer des services (échec), puis il s’est concentré sur le bucket `importantbucket`.
<img width="792" height="227" alt="image" src="https://github.com/user-attachments/assets/b4e83a87-4cc6-4b2b-a4ab-02eca6cee51f" />
<img width="677" height="247" alt="image" src="https://github.com/user-attachments/assets/4c9b540d-491a-4ff7-86cd-64df02fe74c2" />



À l’aide de l’outil `gsutil`, il a :

listé les fichiers du bucket
puis téléchargé le fichier `secretcode.java`
<img width="713" height="128" alt="image" src="https://github.com/user-attachments/assets/3067aae9-59ba-4d54-ade9-3343cb9de552" />


On voit clairement la progression :

storage.objects.list` → phase de reconnaissance
 `storage.objects.get` → exfiltration

 <img width="742" height="280" alt="image" src="https://github.com/user-attachments/assets/7552395d-0097-40dd-b166-a1764381b703" />
On observe ici un appel API `storage.objects.get` permettant le téléchargement du fichier `secretcode.java` confirmant l’exfiltration de données.



🚩 Ce qui a trahi l’attaquant

Un point intéressant ici est la corrélation entre :

 le User-Agent (indiquant macOS via "darwin")
 et l’IP de Data Center

Ce décalage est typique d’une activité malveillante : on n’attend pas un poste utilisateur classique derrière une infrastructure serveur.
<img width="722" height="107" alt="image" src="https://github.com/user-attachments/assets/bb714dfd-23d7-411e-9796-05ad673f34db" />

On observe ici un User-Agent indiquant un système macOS ("darwin") combiné à une adresse IP de Data Center, ce qui constitue un comportement atypique et suspect.




🧠 Ce que je retiens de ce challenge

Ce lab m’a permis de me mettre dans la peau d’un analyste SOC sur un environnement cloud.

Deux points m’ont marqué :

Les service accounts sont une cible critique
  S’ils ne sont pas correctement sécurisés (permissions, restrictions IP), ils deviennent un point d’entrée idéal.

Comprendre les appels API est essentiel
  Savoir distinguer une simple énumération (`list`) d’une exfiltration (`get`) permet de comprendre rapidement l’intention de l’attaquant.



🔐 Mesures de sécurité à mettre en place

Pour éviter ce type d’incident :

* Limiter les permissions des service accounts (principe du moindre privilège)
* Restreindre les accès par IP lorsque c’est possible
* Surveiller les appels API sensibles
* Mettre en place des alertes sur les comportements anormaux
* Faire une rotation régulière des clés



Conclusion

Ce challenge montre bien comment une attaque cloud peut être retracée uniquement à partir des logs.
On passe d’un accès suspect à une exfiltration de données en quelques étapes, avec des indices assez clairs quand on sait où regarder.

C’est exactement le type de scénario que l’on peut rencontrer en environnement SOC.
