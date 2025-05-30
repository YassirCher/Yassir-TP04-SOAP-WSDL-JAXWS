# Rapport : Création, Déploiement et Test d'un Web Service SOAP

Ce document détaille les étapes nécessaires pour créer, déployer, et tester un web service SOAP en utilisant Java et JAX-WS, basé sur le projet fourni.

## 1. Création du Web Service

La première étape consiste à définir la logique métier et l'interface du service web. Cela implique la création des objets de données (POJOs) et de la classe de service elle-même.

### 1.1. Définition de l'entité Compte

Une classe simple `Compte.java` est utilisée pour représenter un compte bancaire. Elle contient généralement des attributs comme un identifiant (ou code), le solde, et potentiellement la date de création. Cette classe sert de type de données échangé par le web service. Dans le projet fourni, cette classe se trouve dans le package `ws` (`src/main/java/ws/Compte.java`). Elle est annotée avec `@XmlRootElement` et `@XmlAccessorType(XmlAccessType.FIELD)` pour faciliter la sérialisation/désérialisation XML par JAXB, qui est utilisé par JAX-WS.

```java
// Extrait simplifié de Compte.java
package ws;

import jakarta.xml.bind.annotation.XmlAccessType;
import jakarta.xml.bind.annotation.XmlAccessorType;
import jakarta.xml.bind.annotation.XmlRootElement;
import jakarta.xml.bind.annotation.XmlTransient;

import java.util.Date;

@XmlRootElement(name = "compte") // Définit le nom de l'élément racine XML
@XmlAccessorType(XmlAccessType.FIELD) // Indique que JAXB doit utiliser les champs pour le binding
public class Compte {
    private Long code;
    private double solde;
    @XmlTransient // Indique que ce champ ne doit pas être inclus dans le XML
    private Date dateCreation;

    // Constructeurs, Getters et Setters...
}
```

### 1.2. Implémentation du Service BanqueService

La classe `BanqueService.java` (`src/main/java/ws/BanqueService.java`) implémente la logique métier du web service. Elle est annotée avec `@WebService` pour l'exposer comme un service web SOAP. Chaque méthode publique destinée à être une opération du service est annotée avec `@WebMethod`. Les paramètres de ces méthodes peuvent être annotés avec `@WebParam` pour spécifier leur nom dans le message SOAP et le WSDL.

Les opérations implémentées sont typiquement :

*   **`conversionEuroToDH(double montant)`**: Convertit un montant donné en euros vers des dirhams marocains (DH) en appliquant un taux de change fixe.
*   **`getCompte(Long code)`**: Récupère les informations d'un compte bancaire spécifique en fonction de son code.
*   **`listComptes()`**: Retourne une liste de tous les comptes bancaires gérés par le service.

```java
// Extrait simplifié de BanqueService.java
package ws;

import jakarta.jws.WebMethod;
import jakarta.jws.WebParam;
import jakarta.jws.WebService;

import java.util.Date;
import java.util.List;

@WebService(serviceName = "BanqueWS") // Définit le nom du service dans le WSDL
public class BanqueService {

    @WebMethod(operationName = "ConversionEuroToDH") // Définit le nom de l'opération dans le WSDL
    public double conversionEuroToDH(@WebParam(name = "montant") double mt) {
        // Logique de conversion...
        return mt * 11.3; // Exemple de taux de change
    }

    @WebMethod
    public Compte getCompte(@WebParam(name = "code") Long code) {
        // Logique pour retrouver un compte par son code...
        return new Compte(code, Math.random() * 9000, new Date());
    }

    @WebMethod
    public List<Compte> listComptes() {
        // Logique pour retourner une liste de comptes...
        return List.of(
                new Compte(1L, Math.random() * 9000, new Date()),
                new Compte(2L, Math.random() * 9000, new Date()),
                new Compte(3L, Math.random() * 9000, new Date())
        );
    }
}
```

## 2. Déploiement du Web Service avec JAX-WS

Une fois le service implémenté, il doit être déployé pour être accessible. JAX-WS fournit un moyen simple de publier un service web en utilisant un serveur HTTP léger intégré.

La classe `ServeurJWS.java` (`src/main/java/ServeurJWS.java`) est responsable de ce déploiement. Elle utilise la méthode statique `Endpoint.publish()` pour démarrer un serveur sur une adresse IP et un port spécifiques (par exemple, `http://0.0.0.0:9191/`) et y publier l'instance de `BanqueService`.

L'adresse `0.0.0.0` est utilisée pour rendre le service accessible depuis n'importe quelle interface réseau de la machine hôte.

```java
// Extrait simplifié de ServeurJWS.java
import jakarta.xml.ws.Endpoint;
import ws.BanqueService;

public class ServeurJWS {
    public static void main(String[] args) {
        String url = "http://0.0.0.0:9191/";
        Endpoint.publish(url, new BanqueService());
        System.out.println("Web service déployé sur " + url);
    }
}

```

Pour lancer le serveur, il suffit d'exécuter la méthode `main` de cette classe. Une fois lancé, le serveur écoute les requêtes SOAP entrantes à l'adresse spécifiée.

**(Insérer ici une capture d'écran de la console montrant le message "Web service déployé sur http://0.0.0.0:9191/")**




## 3. Consultation et Analyse du WSDL

Le WSDL (Web Services Description Language) est un document XML qui décrit le contrat du web service : les opérations disponibles, les types de données échangés, le format des messages et l'adresse du service (endpoint).

Une fois le service déployé avec JAX-WS, le WSDL est automatiquement généré et accessible via une URL spécifique. Si le service est déployé à l'adresse `http://<adresse>:<port>/<nom_service>`, le WSDL est généralement disponible en ajoutant `?wsdl` à cette URL. Dans notre cas, avec le déploiement via `ServeurJWS` à l'adresse `http://0.0.0.0:9191/`, l'URL du WSDL sera `http://localhost:9191/BanqueWS?wsdl` (en supposant que vous accédez depuis la même machine où le serveur tourne, sinon remplacez `localhost` par l'adresse IP appropriée).

Pour consulter le WSDL, ouvrez simplement cette URL dans un navigateur web. Le navigateur affichera le contenu XML du WSDL.

**(Insérer ici une capture d'écran du navigateur affichant le contenu XML du WSDL à l'adresse http://localhost:9191/BanqueWS?wsdl)**

L'analyse de ce fichier WSDL permet de comprendre :

*   **`<types>`**: Définit les schémas XML (XSD) pour les types de données complexes utilisés, comme l'objet `Compte`.
*   **`<message>`**: Décrit les messages échangés (requête et réponse) pour chaque opération.
*   **`<portType>` (ou `<interface>` en WSDL 2.0)**: Définit l'ensemble des opérations abstraites proposées par le service (par exemple, `ConversionEuroToDH`, `getCompte`, `listComptes`).
*   **`<binding>`**: Spécifie le protocole concret (SOAP sur HTTP dans ce cas) et le style d'encodage (par exemple, Document/Literal) pour un `portType` donné.
*   **`<service>`**: Regroupe les différents points d'accès (ports) où le service est disponible. Chaque `<port>` associe un `binding` à une adresse réseau concrète (endpoint URL).

Comprendre le WSDL est essentiel pour les développeurs qui souhaitent interagir avec le web service, car il fournit toutes les informations nécessaires pour construire des requêtes SOAP correctes et interpréter les réponses.




## 4. Test du Web Service avec SoapUI/Oxygen

Pour vérifier le bon fonctionnement du web service déployé, des outils comme SoapUI (open source ou Pro) ou Oxygen XML Editor (avec ses fonctionnalités de test SOAP) sont couramment utilisés. Ces outils permettent d'envoyer des requêtes SOAP au service et d'inspecter les réponses reçues, en se basant sur le WSDL.

La procédure générale est la suivante :

1.  **Créer un nouveau projet SOAP** : Dans SoapUI (ou l'outil choisi), créez un nouveau projet en fournissant l'URL du WSDL (`http://localhost:9191/BanqueWS?wsdl`). L'outil va automatiquement importer la définition du service, y compris les opérations disponibles.

    **(Insérer ici une capture d'écran de la création du projet SoapUI/Oxygen avec l'URL du WSDL)**

2.  **Générer des requêtes exemples** : L'outil génère des modèles de requêtes SOAP pour chaque opération définie dans le WSDL (`ConversionEuroToDH`, `getCompte`, `listComptes`).

3.  **Modifier et envoyer les requêtes** : Adaptez les requêtes exemples en fournissant les valeurs de paramètres nécessaires. Par exemple, pour `ConversionEuroToDH`, spécifiez une valeur pour le paramètre `montant`. Pour `getCompte`, indiquez un `code` de compte.

    **(Insérer ici une capture d'écran d'une requête SoapUI/Oxygen pour l'opération `ConversionEuroToDH` avant envoi, avec une valeur pour `montant`)**

4.  **Analyser les réponses** : Envoyez la requête au service web. L'outil affichera la réponse SOAP reçue du serveur. Vérifiez que la réponse est conforme à ce qui est attendu. Par exemple, pour `ConversionEuroToDH`, la réponse doit contenir le montant converti. Pour `getCompte`, elle doit contenir les détails du compte demandé (ou une indication si le compte n'existe pas). Pour `listComptes`, elle doit contenir une liste d'éléments `compte`.

    **(Insérer ici une capture d'écran de la réponse SOAP reçue dans SoapUI/Oxygen pour l'opération `ConversionEuroToDH`)**

    **(Insérer ici une capture d'écran de la réponse SOAP reçue dans SoapUI/Oxygen pour l'opération `getCompte`)**

    **(Insérer ici une capture d'écran de la réponse SOAP reçue dans SoapUI/Oxygen pour l'opération `listComptes`)**

Ces tests permettent de valider que chaque opération du service fonctionne correctement, que les données sont correctement sérialisées et désérialisées, et que le service est accessible et répond comme prévu avant de passer au développement d'un client applicatif.




## 5. Création d'un Client SOAP Java

Pour interagir avec le web service depuis une application Java, il est courant de générer un "stub" client à partir du WSDL. Ce stub est un ensemble de classes Java qui masquent la complexité de la communication SOAP et permettent d'appeler les opérations du web service comme s'il s'agissait de méthodes Java locales.

### 5.1. Génération du Stub Client

Il existe plusieurs façons de générer le stub :

*   **Avec l'outil `wsimport` (fourni avec le JDK)** : C'est un outil en ligne de commande qui prend l'URL du WSDL en entrée et génère les classes Java correspondantes.
    ```bash
    # Exemple de commande wsimport
    # Assurez-vous que le service est démarré
    wsimport -s src/main/java -p proxy http://localhost:9191/BanqueWS?wsdl
    ```
    Cette commande génère les sources Java (`-s src/main/java`) dans le package `proxy` (`-p proxy`) à partir du WSDL spécifié.

*   **Avec Maven (via un plugin)** : Si le projet utilise Maven (comme c'est le cas dans le sous-dossier `client-soap-java` du projet fourni), la génération du stub peut être automatisée via un plugin comme `jaxws-maven-plugin`. Le `pom.xml` du client contient probablement une configuration pour ce plugin, qui s'exécute pendant la phase `generate-sources` du build Maven.

    ```xml
    <!-- Extrait simplifié de pom.xml (client-soap-java/pom.xml) -->
    <build>
        <plugins>
            <plugin>
                <groupId>com.sun.xml.ws</groupId>
                <artifactId>jaxws-maven-plugin</artifactId>
                <version>...</version> <!-- Mettre la version appropriée -->
                <executions>
                    <execution>
                        <goals>
                            <goal>wsimport</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <wsdlUrls>
                        <wsdlUrl>http://localhost:9191/BanqueWS?wsdl</wsdlUrl>
                    </wsdlUrls>
                    <keep>true</keep>
                    <packageName>proxy</packageName> <!-- Package pour les classes générées -->
                    <sourceDestDir>src/main/java</sourceDestDir>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ```
    En exécutant `mvn generate-sources` (ou une phase ultérieure comme `mvn compile` ou `mvn package`), Maven invoquera `wsimport` pour générer les classes dans le répertoire et le package spécifiés (`src/main/java/proxy` dans cet exemple).

    **(Insérer ici une capture d'écran montrant les classes générées dans le package `proxy` après l'exécution de `wsimport` ou de Maven)**

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
package org.example;

import proxy.BanqueService;
import proxy.BanqueWS;
import proxy.Compte;

import java.util.List;

public class Main {
    public static void main(String[] args) {
        // Création d'une instance du service (stub)
        BanqueWS stub = new BanqueService().getBanqueWSPort();

        // Appel de l'opération de conversion
        double montant = 100.0;
        double resultatConversion = stub.conversionEuroToDH(montant);
        System.out.println("Conversion de " + montant + " EUR en DH : " + resultatConversion);

        // Appel de l'opération getCompte
        Long codeCompte = 1L;
        Compte compte = stub.getCompte(codeCompte);
        if (compte != null) {
            System.out.println("Détails du compte " + codeCompte + " : Solde = " + compte.getSolde());
        } else {
            System.out.println("Compte " + codeCompte + " non trouvé.");
        }

        // Appel de l'opération listComptes
        List<Compte> comptes = stub.listComptes();
        System.out.println("Liste des comptes :");
        for (Compte c : comptes) {
            System.out.println("  - Code: " + c.getCode() + ", Solde: " + c.getSolde());
        }
    }
}
```

Pour exécuter ce client, assurez-vous que le serveur JAX-WS (`ServeurJWS`) est en cours d'exécution. Compilez et exécutez ensuite la classe `Main`. La sortie dans la console montrera les résultats des appels aux différentes opérations du web service.

**(Insérer ici une capture d'écran de la console montrant la sortie de l'exécution du client Java `Main.java`)**


