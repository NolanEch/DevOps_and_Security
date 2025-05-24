Descriptif:
in-toto est un framework de sécurité qui permet d’assurer l’intégrité de la chaîne de développement logiciel, du code source jusqu’au déploiement. Son objectif principal est de garantir que chaque étape d’un processus (comme le développement, les tests, la compilation, ou le déploiement) a bien été réalisée comme prévu, par les bonnes personnes et sans modification malveillante.
Concrètement, in-toto fonctionne en définissant une supply chain (chaîne d'approvisionnement logicielle) sous forme de métadonnées cryptographiquement signées. Chaque acteur ou étape du pipeline (comme un développeur, un outil CI, ou un script de build) génère des preuves signées attestant de ce qui a été fait. À la fin, ces métadonnées sont vérifiées pour confirmer que le logiciel livré est bien celui qui a été construit en suivant toutes les règles définies au départ.
Ce système est particulièrement utile pour se protéger contre les attaques sur la chaîne d'approvisionnement logicielle, comme celles qui injectent du code malveillant lors de la compilation ou dans une dépendance. in-toto est souvent utilisé en complément d'autres outils comme TUF ou Sigstore, pour renforcer la confiance dans les processus de build et de livraison continue.
En résumé, in-toto permet de tracer et vérifier tout ce qui se passe dans une chaîne de production logicielle, garantissant que rien n’a été compromis entre le développement initial et le déploiement final.


Point de blocage:

Concepts peu documentés:

Les notions de layout, keys, étapes de la supply chain et attestation sont peu connues et assez abstraites.

Difficile de faire le lien concret entre in-toto et un projet réel sans exemple simple.

Utilisation en ligne de commande:

La création de clés (in-toto-keygen), la définition des étapes (in-toto-record start/end) et la finalisation (in-toto-verify) nécessitent une rigueur stricte dans les noms, chemins et signatures.

Nombreux échecs à la vérification (in-toto-verify) dus à des erreurs subtiles (chemin relatif, étape manquante, etc).