Claire:
Descriptif:
Dans un environnement basé sur les conteneurs comme Docker, la sécurité est un enjeu majeur. Même si les conteneurs offrent un certain isolement, ils ne sont pas invulnérables : ils peuvent embarquer des vulnérabilités dans leurs images. C’est là qu’intervient Clair, un outil open source développé par CoreOS (maintenant intégré à Red Hat), conçu pour analyser la sécurité des images Docker.
Clair fonctionne comme un système de scan de vulnérabilités. Il analyse les différentes couches d’une image de conteneur, en extrayant les métadonnées des paquets installés, puis en les comparant à une base de données publique de vulnérabilités (comme le NVD, National Vulnerability Database). Le résultat est un rapport clair et détaillé qui liste toutes les failles de sécurité connues affectant les composants de l’image.
Ce type d’analyse est crucial pour intégrer la sécurité dans une approche DevSecOps. On peut automatiser l’analyse avec Clair dans des pipelines CI/CD pour détecter les vulnérabilités avant même que les images soient déployées en production. Cela permet non seulement de sécuriser les applications, mais aussi de maintenir la conformité avec certaines normes de sécurité.
En résumé, Clair est un outil indispensable pour analyser les vulnérabilités des images Docker, s’assurer que le code embarqué n’introduit pas de failles connues, et renforcer la sécurité globale des environnements conteneurisés.
Code:
Pour l’installation de claire il est préférable de le faire avec Linux ou Wsl que Windows.
On passe ensuite à l'installation dans un premier temps il faut prendre en compte le fait qu’il est utile de créer une base PostgreSQL puisqu’elle nous sera utile pour stocker et organiser une grande quantité de données nécessaires à l’analyse de vulnérabilités. On ajoute donc les éléments suivant à notre fichier docker-compose:
 clair-db:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: clair
      POSTGRES_USER: clair
      POSTGRES_PASSWORD: clair
    ports:
      - "5432:5432"
    volumes:
      - clair-db-data:/var/lib/postgresql/data
    networks:
      - clairnet
  clair:
    image: quay.io/projectquay/clair:4.7.2
    container_name: clair
    depends_on:
      - clair-db
    networks:
      - clairnet
    ports:
      - "6060:6060"
    command: ["clair", "-config", "/config/config.yaml"]
    volumes:
      - ./config.yaml:/config/config.yaml
volumes:
  clair-db-data:

networks:
  clairnet:


Si nécessaire il sera possible de pull les images des deux en utilisant les commandes suivantes:
docker pull quay.io/projectquay/clair:v4.7.2
docker pull postgres:13


On va également en plus d’ajouter à docker-compose les deux éléments, créer un fichier config.yaml que l’on va placer dans le même dossier et qui sera de la forme suivante:
http_listen_addr: ":6060"
log_level: "info"

indexer:
  connstring: host=clair-dblocalhost user=clair dbname=clair password=clair sslmode=disable
  scanlock_retry: 10
  layer_scan_concurrency: 5  
  migrations: true

matcher:
  connstring: host=clair-db user=clair dbname=clair password=clair sslmode=disable

notifier:
  connstring: host=clair-db user=clair dbname=clair password=clair sslmode=disable
  delivery_interval: 1m
  poll_interval: 5m

auth:
  psk:
    key: "bXlzZWNyZXRrZXk="  
    iss: ["clairctl"]

metrics:
  name: "prometheus"

On passe ensuite à l’ajout de clairctl qui permet d'analyser des images Docker pour trouver des failles de sécurité. Car par défaut, Clair ne comprend pas les images Docker directement : il attend des données bien préparées qu'on doit lui envoyer via des requêtes techniques.
wget https://github.com/jgsqware/clairctl/releases/download/v1.2.8/clairctl-linux-amd64 -O clairctl


