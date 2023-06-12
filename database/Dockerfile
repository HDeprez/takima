FROM postgres:14.1-alpine

ENV POSTGRES_DB=tp_db \
   POSTGRES_USER=exampleuser \
   POSTGRES_PASSWORD=examplepassword

COPY ./docker-entrypoint-initdb.d /docker-entrypoint-initdb.d