FROM postgres:14.1-alpine

#ENV POSTGRES_DB=db \
#   POSTGRES_USER=usr \
#   POSTGRES_PASSWORD=pwd

# Copie des scripts SQL dans le répertoire spécifié
COPY init-scripts-db/*.sql /docker-entrypoint-initdb.d/