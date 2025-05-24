Descriptif:
Vault est un outil développé par HashiCorp qui sert à sécuriser, stocker et contrôler l’accès aux secrets utilisés dans une infrastructure informatique, comme les mots de passe, les tokens, les clés API ou les certificats. Plutôt que de laisser ces informations sensibles traîner dans des fichiers de configuration ou dans le code, Vault les centralise dans un coffre-fort chiffré, accessible uniquement via des politiques d’accès bien définies.
Ce qui le rend particulièrement puissant, c’est sa capacité à générer des secrets dynamiques, c’est-à-dire temporaires, comme des identifiants pour une base de données qui expirent automatiquement. Cela réduit fortement les risques de fuite ou d’utilisation abusive. Vault permet aussi la rotation automatique des secrets, ainsi qu’un suivi précis de qui accède à quoi, grâce à son système d’audit.
Dans notre projet, Vault est utilisé pour sécuriser la gestion des secrets entre les différents services Docker. Plutôt que de définir les mots de passe dans les fichiers docker-compose, chaque service va interagir avec Vault pour récupérer ce dont il a besoin, de manière sécurisée et centralisée. C’est une solution fiable pour renforcer la sécurité d’une architecture distribuée.
Code:
On commence par télécharger l’image de vault pour l’utiliser avec dockers.
On passe ensuite à la configuration du fichier vault config que l’on placera dans le dossier config, pour la configuration du vault qui ressemblera à ça :

storage "file" {
  path = "/vault/data"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = true
}

api_addr = "http://localhost:8200"
ui = true


Ensuite on passe à l’ajout de Vault au docker.compose.yml:

 vault:
    image: hashicorp/vault:latest
    container_name: vault-new
    ports:
      - "8200:8200"
      - "8201:8201"
    environment:
      VAULT_ADDR: "https://vault1.poc.thecloudgarage.com:8200"
      VAULT_API_ADDR: "https://vault1.poc.thecloudgarage.com:8200"
      VAULT_ADDRESS: "https://vault1.poc.thecloudgarage.com:8200"
    volumes:
      - ./logs:/vault/logs/:rw
      - ./data:/vault/data/:rw
      - ./config:/vault/config/:rw
      - ./certs:/certs/:rw
      - ./file:/vault/file/:rw
    cap_add:
      - IPC_LOCK
    entrypoint: vault server -config=/vault/config/vault-config.hcl

Puis on peut lancer le vault dans le terminal avec la commande suivante en se plaçant ici dans le dossier api:

docker compose up 


On peut ensuite stocker les secrets dans le vault avec la commande suivante ici par exemple on veut stocker le mot de passe pour le sécuriser:

docker exec -it vault vault kv put secret/mysql password="tlc"


Ensuite si l’on souhaite le lire par la suite la commande sera la suivante:

docker exec -it vault vault kv get secret/mysql



Point de blocage:

Installation et lancement:

Vault nécessite une phase d’initialisation et de déverrouillage (unseal) assez déroutante pour un premier usage.

Comprendre le fonctionnement des “keys” d’unseal et du root token a demandé plusieurs essais.

Utilisation avec Docker:

Complexité pour exposer Vault en mode dev dans un container Docker et y accéder depuis l’extérieur.

Problèmes de ports ou de configuration réseau liés à l’accessibilité de l’API Vault depuis d'autres services.