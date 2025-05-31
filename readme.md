# Rapport : Création, Déploiement et Test d'un Web Service SOAP

Ce document détaille les étapes nécessaires pour créer, déployer, et tester un web service SOAP en utilisant Java et JAX-WS, basé sur le projet fourni.

## 1. Création du Web Service

La première étape consiste à définir la logique métier et l'interface du service web. Cela implique la création des objets de données  et de la classe de service elle-même.
--
### 1.1. Définition de l'entité Compte

Une classe simple `Compte.java` est utilisée pour représenter un compte bancaire. Elle contient généralement des attributs comme un identifiant (ou code), le solde, et potentiellement la date de création. Cette classe sert de type de données échangé par le web service. Dans le projet fourni, cette classe se trouve dans le package `ws` (`src/main/java/ws/Compte.java`). Elle est annotée avec `@XmlRootElement` et `@XmlAccessorType(XmlAccessType.FIELD)` pour faciliter la sérialisation/désérialisation XML par JAXB, qui est utilisé par JAX-WS.

--
### 1.2. Implémentation du Service BanqueService

La classe `BanqueService.java` (`src/main/java/ws/BanqueService.java`) implémente la logique métier du web service. Elle est annotée avec `@WebService` pour l'exposer comme un service web SOAP. Chaque méthode publique destinée à être une opération du service est annotée avec `@WebMethod`. Les paramètres de ces méthodes peuvent être annotés avec `@WebParam` pour spécifier leur nom dans le message SOAP et le WSDL.

Les opérations implémentées sont typiquement :

*   **`conversionEuroToDH(double montant)`**: Convertit un montant donné en euros vers des dirhams marocains (DH) en appliquant un taux de change fixe.
*   **`getCompte(Long code)`**: Récupère les informations d'un compte bancaire spécifique en fonction de son code.
*   **`listComptes()`**: Retourne une liste de tous les comptes bancaires gérés par le service.

```java
package ws;

import jakarta.jws.WebMethod;
import jakarta.jws.WebParam;
import jakarta.jws.WebService;

import java.time.LocalDate;
import java.util.Date;
import java.util.List;

@WebService(serviceName = "BanqueWS")
public class BanqueService {

    @WebMethod(operationName = "conversionEuroToDH")
    public double conversion(@WebParam(name = "montant") double mt) {
        return mt * 11;
    }

    @WebMethod
    public Compte getCompte(@WebParam(name = "code") int code) {
        return new Compte(code, Math.random() * 60000, new Date());
    }

    @WebMethod
    public List<Compte> listComptes() {
        return List.of(
                new Compte(1, Math.random() * 60000, new Date()),
                new Compte(2, Math.random() * 60000, new Date()),
                new Compte(3, Math.random() * 60000, new Date())
        );
    }
}
```
---
## 2. Déploiement du Web Service avec JAX-WS

Une fois le service implémenté, il doit être déployé pour être accessible. JAX-WS fournit un moyen simple de publier un service web en utilisant un serveur HTTP léger intégré.

La classe `ServeurJWS.java` (`src/main/java/ServeurJWS.java`) est responsable de ce déploiement. Elle utilise la méthode statique `Endpoint.publish()` pour démarrer un serveur sur une adresse IP et un port spécifiques (par exemple, `http://0.0.0.0:9090/`) et y publier l'instance de `BanqueService`.

L'adresse `0.0.0.0` est utilisée pour rendre le service accessible depuis n'importe quelle interface réseau de la machine hôte.

```java
// Extrait simplifié de ServeurJWS.java
import jakarta.xml.ws.Endpoint;
import ws.BanqueService;

public class ServeurJWS {
    public static void main(String[] args) {
        String url = "http://0.0.0.0:9090/";
        Endpoint.publish(url, new BanqueService());
        System.out.println("Web service déployé sur " + url);
    }
}

```

Pour lancer le serveur, il suffit d'exécuter la méthode `main` de cette classe. Une fois lancé, le serveur écoute les requêtes SOAP entrantes à l'adresse spécifiée.

![image](https://github.com/user-attachments/assets/17529b3f-4dee-49f3-a7ae-3805ee8cff16)



---

## 3. Consultation et Analyse du WSDL

Le WSDL (Web Services Description Language) est un document XML qui décrit le contrat du web service : les opérations disponibles, les types de données échangés, le format des messages et l'adresse du service (endpoint).

Une fois le service déployé avec JAX-WS, le WSDL est automatiquement généré et accessible via une URL spécifique. Si le service est déployé à l'adresse `http://<adresse>:<port>/<nom_service>`, le WSDL est généralement disponible en ajoutant `?wsdl` à cette URL. Dans notre cas, avec le déploiement via `ServeurJWS` à l'adresse `http://0.0.0.0:9090/`, l'URL du WSDL sera `http://localhost:9090/BanqueWS?wsdl` (en supposant que vous accédez depuis la même machine où le serveur tourne, sinon remplacez `localhost` par l'adresse IP appropriée).

Pour consulter le WSDL, ouvrez simplement cette URL dans un navigateur web. Le navigateur affichera le contenu XML du WSDL.

![image](https://github.com/user-attachments/assets/ec45d185-e9f5-4877-9266-300d2800797c)


L'analyse de ce fichier WSDL permet de comprendre :

*   **`<types>`**: Définit les schémas XML (XSD) pour les types de données complexes utilisés, comme l'objet `Compte`.
*   **`<message>`**: Décrit les messages échangés (requête et réponse) pour chaque opération.
*   **`<portType>` (ou `<interface>` en WSDL 2.0)**: Définit l'ensemble des opérations abstraites proposées par le service (par exemple, `ConversionEuroToDH`, `getCompte`, `listComptes`).
*   **`<binding>`**: Spécifie le protocole concret (SOAP sur HTTP dans ce cas) et le style d'encodage (par exemple, Document/Literal) pour un `portType` donné.
*   **`<service>`**: Regroupe les différents points d'accès (ports) où le service est disponible. Chaque `<port>` associe un `binding` à une adresse réseau concrète (endpoint URL).

Comprendre le WSDL est essentiel pour les développeurs qui souhaitent interagir avec le web service, car il fournit toutes les informations nécessaires pour construire des requêtes SOAP correctes et interpréter les réponses.




## 4. Test du Web Service avec SoapUI

Pour vérifier le bon fonctionnement du web service déployé, des outils comme SoapUI (open source ou Pro) ou Oxygen XML Editor (avec ses fonctionnalités de test SOAP) sont couramment utilisés. Ces outils permettent d'envoyer des requêtes SOAP au service et d'inspecter les réponses reçues, en se basant sur le WSDL.

La procédure générale est la suivante :

1.  **Créer un nouveau projet SOAP** : Dans SoapUI (ou l'outil choisi), créez un nouveau projet en fournissant l'URL du WSDL (`http://localhost:91/BanqueWS?wsdl`). L'outil va automatiquement importer la définition du service, y compris les opérations disponibles.

    ![image](https://github.com/user-attachments/assets/07248899-6be8-4c36-9b12-90ea409604bf)
    ![image](https://github.com/user-attachments/assets/6c052a30-78d1-49d9-933b-6582f6f758cb)


3.  **Générer des requêtes exemples** : L'outil génère des modèles de requêtes SOAP pour chaque opération définie dans le WSDL (`ConversionEuroToDH`, `getCompte`, `listComptes`).

4.  **Modifier et envoyer les requêtes** : Adaptez les requêtes exemples en fournissant les valeurs de paramètres nécessaires. Par exemple, pour `ConversionEuroToDH`, spécifiez une valeur pour le paramètre `montant`. Pour `getCompte`, indiquez un `code` de compte.

    ![image](https://github.com/user-attachments/assets/ae0cc7c1-d97d-470d-917f-0e7382a55bbb)



5.  **Analyser les réponses** : Envoyez la requête au service web. L'outil affichera la réponse SOAP reçue du serveur. Vérifiez que la réponse est conforme à ce qui est attendu. Par exemple, pour `ConversionEuroToDH`, la réponse doit contenir le montant converti. Pour `getCompte`, elle doit contenir les détails du compte demandé (ou une indication si le compte n'existe pas). Pour `listComptes`, elle doit contenir une liste d'éléments `compte`.

    

    ![image](https://github.com/user-attachments/assets/9c2c9ae4-5f51-4606-995f-1263907c7f6e)


    ![image](https://github.com/user-attachments/assets/515af9be-252b-436c-abf6-d44154227eab)


Ces tests permettent de valider que chaque opération du service fonctionne correctement, que les données sont correctement sérialisées et désérialisées, et que le service est accessible et répond comme prévu avant de passer au développement d'un client applicatif.




## 5. Création d'un Client SOAP Java

Pour interagir avec le web service depuis une application Java, il est courant de générer un "stub" client à partir du WSDL. Ce stub est un ensemble de classes Java qui masquent la complexité de la communication SOAP et permettent d'appeler les opérations du web service comme s'il s'agissait de méthodes Java locales.

### 5.1. Génération du Stub Client

Il existe plusieurs façons de générer le stub :


*   ** en choisisant le dossier java du client-soap-java puis cliquer sur help -> find action -> generate javacode from wsdl :
  ![image](https://github.com/user-attachments/assets/a5099d60-3153-459b-93ec-86a94894a307)

Les classes générées incluent généralement :
*   Une interface de service (ex: `BanqueService.java` dans le package `proxy`) représentant le `portType` du WSDL.
*   Une classe d'implémentation du service (ex: `BanqueWS.java`) qui fournit une méthode pour obtenir une instance du port (le point d'accès au service).
*   Des classes correspondant aux types de données complexes (ex: `Compte.java` dans `proxy`).
*   Des classes représentant les opérations et leurs paramètres/retours (ex: `ConversionEuroToDH.java`, `ConversionEuroToDHResponse.java`, `GetCompte.java`, etc.).

### 5.2. Création du Client Java

Une fois le stub généré, la création d'un client Java est simple. Il suffit d'instancier la classe de service générée et d'obtenir une référence au port du service. Ensuite, les opérations du web service peuvent être appelées directement via les méthodes de cet objet port.

Le fichier `client-soap-java/src/main/java/org/example/Main.java` dans le projet fourni montre un exemple de client :

```java
// Extrait simplifié de Main.java
import proxy.BanqueService;
import proxy.BanqueWS;
import proxy.Compte;

public class Main {
    public static void main(String[] args) {
        BanqueService proxy = new BanqueWS().getBanqueServicePort();
        System.out.println(proxy.conversionEuroToDH(0));
        Compte compte = proxy.getCompte(0);
        System.out.println("------------");
        System.out.println(compte.getCode());
        System.out.println(compte.getSolde());
        System.out.println(compte.getDateCreation());

        proxy.listComptes().forEach(cp -> {
            System.out.println("------------");
            System.out.println(cp.getCode());
            System.out.println(cp.getSolde());
            System.out.println(cp.getDateCreation());
        });
    }
}
```

Pour exécuter ce client, assurez-vous que le serveur JAX-WS (`ServeurJWS`) est en cours d'exécution. Compilez et exécutez ensuite la classe `Main`. La sortie dans la console montrera les résultats des appels aux différentes opérations du web service.

![image](https://github.com/user-attachments/assets/2ea165e9-4c92-444b-b2f7-f61c35324a23)




