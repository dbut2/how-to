### 1. Set env vars

```sh
export GCLOUD_PROJECT = [gcloud project-id] # project-1
export DB_HOST = [db host] # root
export DB_PORT = [db port] # password
export HOST = [url of monica service] # https://monica.example.com
```


### 1. Setup gcloud

Authenticate gcloud and set project-id

```sh
gcloud auth login --update-adc --use-oauth2client
gcloud config set project $GCLOUD_PROJECT
```

### 2. Create database

Create user and respective database through phpmyadmin:
1. navigate to home, user accounts, add user account
2. enter user name, example used `monica`
3. click generate password, take note of this
4. click `Create database with same name and grant all privileges.`
5. click `Go`

This will generate and run the following sql script, alternatively you can just run the sql script according to your needs

```sql
CREATE USER 'monica'@'%'
IDENTIFIED WITH mysql_native_password AS '***';

GRANT USAGE ON *.* TO 'monica'@'%'
REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;

CREATE DATABASE IF NOT EXISTS `monica`;

GRANT ALL PRIVILEGES ON `monica`.* TO 'monica'@'%';
```

### 3. Create gcp secrets for db credentials

Create secrets for monica to read as environments variables. Ensure env vars for `$DB_USERNAME` & `$DB_PASSWORD` are correctly set

```sh
echo $DB_USERNAME | gcloud secrets create monica-db-user --data-file=-
echo $DB_PASSWORD | gcloud secrets create monica-db-pass --data-file=-
```

### 4. Pull, tag and push docker image

Let cloud run see the latest image for monica

```sh
docker pull monica
docker tag monica gcr.io/$GCLOUD_PROJECT/monica
docker push gcr.io/$GCLOUD_PROJECT/monica
```

### 5. Deploy cloud run

Let the magic happen

```sh
gcloud run deploy monica \
--platform=managed \
--region=us-west1 \
--image=gcr.io/$GCLOUD_PROJECT/monica \
--port=80 \
--set-env-vars=APP_ENV=production,APP_URL=$HOST,DB_CONNECTION=mysql,DB_DATABASE=$DB_DATABASE,DB_HOST=$DB_HOST,DB_PORT=$DB_PORT \
--set-secrets=DB_USERNAME=monica-db-user:1,DB_PASSWORD=monica-db-pass:1 \
--allow-unauthenticated
```
