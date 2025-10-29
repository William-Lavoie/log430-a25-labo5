<img src="https://upload.wikimedia.org/wikipedia/commons/2/2a/Ets_quebec_logo.png" width="250"> \
William Lavoie \
Rapport de laboratoire \
LOG430 — Architecture logicielle \
28 Octobre 2025
École de technologie supérieure

## Questions
### Question 1
#### Quelle réponse obtenons-nous à la requête à POST /payments ? Illustrez votre réponse avec des captures d'écran/terminal.

En envoyant la requête à `POST http://localhost:8080/payments-api/payments` par Postman, ce qui correspond à la route utilisé par `store-manager`, à l'exception que `localhost` est remplacé par le nom du conteneur dans le réseau Docker, on obtient la réponse suivante. Le service de paiement retourne l'id du paiement qui a été exécuté, avec un code 200 pour signifier que l'opération s'est bien passé.

![Question 1 image 1](./images/1.1.png)


### Question 2
#### Quel type d'information envoyons-nous dans la requête à POST payments/process/:id ? Est-ce que ce serait le même format si on communiquait avec un service SOA, par exemple ? Illustrez votre réponse avec des exemples et captures d'écran/terminal.

![Question 2 image 1](./images/2.1.png)

### Question 3
#### Quel résultat obtenons-nous de la requête à POST payments/process/:id?


### Question 4
#### Quelle méthode avez-vous dû modifier dans log430-a25-labo05-payment et qu'avez-vous modifié ? Justifiez avec un extrait de code.

J'ai modifié la méthode `update_status_to_paid` car c'est elle qui s'occupe de faire passer l'état des paiements à `is_valid = True`, donc il est logique que ce soit cette méthode qui appelle `store_manager` afin de transmettre cette information. J'ai ajouté la partie suivante à la méthode:

![Question 4 image 1](./images/4.1.png)

Dans les logs de `store_manager`, on voit les lignes suivantes qui démontre que les commandes sont mises à jour correctement.

![Question 4 image 2](./images/4.2.png)



### Question 5
#### À partir de combien de requêtes par minute observez-vous les erreurs 503 ? Justifiez avec des captures d'écran de Locust.

Les erreurs 503 commencent à partir de 2 RPS comme on peut le voir ci-dessous.

![Question 5 image 1](./images/5.1.png)

Ceci est dû au fait que la configuration du `endpoint` dans krakenD permet 10 requêtes par minute, cependant krakenD converti en requête par seconde, ce qui revient à 1.6 de permise, ainsi, il peut y avoir maximum 10 requêtes par minute, et au plus une par seconde. 

![Question 5 image 2](./images/5.2.png)


Le nombre d'utilisateurs maximum n'a essentiellement pas changé avec l'introduction de Nginx, ce qui était attendu car le *bottleneck* est MySQL, or les 3 instances de `store_manager` utilisent toute la même base de données, ainsi le nombre de requêtes envoyé à MySQL n'a pas changé.
Par contre, la latence a augmenté car le traffic est partagé par les instances de `store_manager`. La latence moyenne est maintenant de 5ms.

![Question 5 image 3](./images/5.3.png)


### Question 6
#### Que se passe-t-il dans le navigateur quand vous faites une requête avec un délai supérieur au timeout configuré (5 secondes) ? Quelle est l'importance du timeout dans une architecture de microservices ? Justifiez votre réponse avec des exemples pratiques.

On obtient une erreur 500 ce qui signifie que le serveur a rencontré une erreur. Cela est causée par le timeout configuré dans KrakenD. Le but de mettre un timeout est d'éviter un "callback hell", soit d'attendre une requête qui ne viendra jamais. Par exemple, dans le cadre d'une architecture microservices il est possible qu'un service ne fonctionne plus, ainsi si les autres services l'appellent ils ne reçevront jamais de réponse, il est donc essentiel de détermine un temps d'attente maximum.

## Observations additionnelles

Le script d'intégration/déploiement continu (CI/CD) est presque identique à celui du laboratoire 3, je n'ai donc pas vraiment rencontré de problème avec ça. J'ai utilisé un `self-hosted runner` sur la VM et le script roule sur Github Actions.

J'ai seulement ajouté la ligne `docker network rm labo04-network || true` afin de détruire le réseau Docker à chaque déploiement sur la VM pour toujours commencer avec une installation fraîche. La seconde partie évite de lancer une erreur si le réseau n'existe pas déjà.

Autrement, j'ai rencontré un problème dans la section 9 en utilisant les méthodes suggérées, soit `hset` de Redis, car celle-ci s'attend à reçevoir une clé et une valeur et non une liste, j'ai donc utilisé `set` à la place. J'ai remplacé `hget` par `get` afin que cela concorde. 