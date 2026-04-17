Mon analyse : Compromission Google Cloud (LetsDefend)

Le scénario en deux mots

Je me suis penché sur un cas de compromission chez une startup.
Un attaquant a réussi à récupérer une identité Google Cloud pour accéder à un bucket contenant du code sensible.
<img width="871" height="667" alt="image" src="https://github.com/user-attachments/assets/a345c5f5-115a-4e89-a909-da93b501d39f" />





Ma démarche d’investigation

En analysant les logs, j’ai pu reconstituer le déroulement de l’attaque.

La cible
Le service account `cloud-storage-helper` a été compromis.
C’est un cas assez classique : ce type de compte est souvent utilisé pour automatiser des tâches et peut avoir des permissions trop larges.


L’alerte
L’activité provenait de l’IP `178.132.108.38`, localisée en Roumanie.
Ce qui m’a immédiatement paru suspect, c’est qu’il s’agit d’une IP de Data Center. Ce n’est pas le comportement attendu d’un utilisateur légitime.



Les actions de l’attaquant
L’attaquant a d’abord tenté d’activer des services (échec), puis il s’est concentré sur le bucket `importantbucket`.

À l’aide de l’outil `gsutil`, il a :

listé les fichiers du bucket
puis téléchargé le fichier `secretcode.java`

On voit clairement la progression :

storage.objects.list` → phase de reconnaissance
 `storage.objects.get` → exfiltration



🚩 Ce qui a trahi l’attaquant

Un point intéressant ici est la corrélation entre :

 le User-Agent (indiquant macOS via "darwin")
 et l’IP de Data Center

Ce décalage est typique d’une activité malveillante : on n’attend pas un poste utilisateur classique derrière une infrastructure serveur.



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
