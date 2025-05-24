InSpec
Descriptif:
InSpec est un framework open source développé par Chef qui permet d’écrire des tests d’infrastructure sous forme de code. Son but est de vérifier que les systèmes, les serveurs ou les conteneurs sont bien configurés selon les règles de sécurité, de conformité ou d’exploitation attendues. Ces tests peuvent porter sur la présence de paquets, les permissions de fichiers, la configuration d’un service, ou même les règles réseau.
L’intérêt principal d’InSpec, c’est qu’il permet d’automatiser les audits de sécurité. Au lieu de faire les vérifications manuellement, on écrit des fichiers de tests déclaratifs, en Ruby, qui décrivent ce qui doit être présent ou interdit sur un système. C’est donc une approche “compliance-as-code”, où la conformité devient vérifiable de façon reproductible.
Dans le cadre de notre projet, InSpec nous permet de nous assurer que nos services déployés dans Docker respectent certaines bonnes pratiques de sécurité, comme ne pas exposer de ports inutiles, avoir des permissions limitées, ou interdire l’accès en root à certains fichiers. On peut intégrer ces tests directement dans un pipeline CI/CD pour que chaque déploiement soit automatiquement validé.
Code:
On commence par créer une image personnalisé de InSpec avec un Dockerfile:
FROM ruby:3.2
RUN gem install inspec
WORKDIR /share
RUN inspec --version
ENTRYPOINT ["inspec"]

Dans le dossier où se trouve le Dockerfile:
docker build -t custom-inspec .

Créer un dossier pour profil InSpec pour la suite:
mkdir inspec-profile
cd inspec-profile

Puis on initialise le profil ce qui va créer la structure du profil avec un dossier controls/
docker run --rm -it -v "${PWD}:/share" custom-inspec init profile /share


1. Écrire les contrôles (tests)
Après avoir créé le profil InSpec avec :

docker run --rm -it -v "${PWD}:/share" custom-inspec init profile /share


On aura un dossier controls/ où l’on peut écrire les tests en Ruby. Par exemple, crée un fichier controls/example.rb avec un test simple :
control 'ssh-1' do
  impact 1.0
  title 'Le service SSH doit être installé'
  describe package('openssh-server') do
    it { should be_installed }
  end
end



2. Exécuter les tests localement
Pour lancer les tests depuis le dossier de ton profil, utilise :

docker run --rm -v "${PWD}:/share" custom-inspec exec /share

Cela exécutera les tests dans le profil et donnera un rapport de conformité.


Point de blocage:

Création d’un profil:

La commande inspec init profile fonctionne bien, mais la structure générée n’est pas évidente à adapter sans expérience en Ruby.

Les fichiers controls/example.rb sont verbeux et la syntaxe des tests n’est pas intuitive sans exemples précis.

Exécution des tests:

Exécution avec Docker : il a fallu créer une image custom de InSpec (via Dockerfile) car l’image officielle ne couvrait pas tous.

Montage de volume (-v "${PWD}:/share") sensible aux erreurs.