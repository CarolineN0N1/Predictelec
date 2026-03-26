### structure sur la VM
/opt/predictelec/
│
├── releases/
│   ├── 2026-02-17/
│   └── 2026-02-25/
│
├── shared/
│   ├── .env
│   └── logs/
│
└── current/predictelec -> releases/2026-02-17

Note: namespace "predictelec" (current/predictelec) ajouté pour que Airflow trouve le projet et le lie à son plugins

###déploiement d'une nouvelle version
##Etape 1 : créer une release
# se positionner sur le répertoire releases
cd /opt/predictelec/releases

# cloner le projet depuis git hub et la branche souhaitée vers répertoire nom=date du jour (=version)
git clone --branch docker-vm-test --depth 1 https://github.com/JulienNeville/predictelec_etl.git 2026-02-18

##Etape 2 : lier le .env partagé (dans shared)
# créer ou mettre à jour le .env du répertoire /opt/predictelec/shared (voir le cas échéant les paramètres proposés dans .env.prod)

# créer un lien symlink avec la release de prod
ln -s /opt/predictelec/shared/.env /opt/predictelec/releases/2026-02-18/.env

##Etape 3 : pointer le predictelec current avec la release de prod
# le dossier namespace predictelec ne doit pas exister dans /opt/predictelec/current, faire un rm si besoin
S'il existe le lien créera un dossier du nom de la release, s'il n'existe pas la cmd créera le dossier predictelec et le liera avec le dossier release demandé

# créer un lien symlink entre la release de prod et le dossier current
ln -sfn /opt/predictelec/releases/2026-02-18 /opt/predictelec/current/predictelec

# vérifier
ls -l /opt/predictelec/current/

# on doit voir à quelle release le namespace predictelec est lié
predictelec -> /opt/predictelec/releases/2026-02-18

###lancement du docker du projet predictelec
#depuis le dossier /opt/predictelec/current/predictelec
docker compose up -d --build

###initialiser la base de données
## A ne lancer que la 1ère fois
#création de la bdd, des tables et des vues
docker compose run app INIT

## A lancer au besoin
#si update des vues ou création de nouvelles vues
docker compose run app INIT_VIEWS

#pour rafraîchir les données des vues matérialisées
docker compose run app REFRESH_VIEWS

## A lancer manuellement pour recette avant orchestration dans Airflow
#LANCEMENT MAJ_STRUCTURES : centrales électriques + stations météos + liens centrales-stations
docker compose run app MAJ_STRUCTURES

#LANCEMENT MAJ_PROD + refresh_views : données de production électrique
docker compose run app MAJ_PROD

#LANCEMENT MAJ_METEO + refresh_views : données météo récentes
docker compose run app MAJ_METEO

#LANCEMENT MAJ_METEO_PREC + refresh_views : données météo historiques
docker compose run app MAJ_METEO_PREC