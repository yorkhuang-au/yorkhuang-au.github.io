## Steps

1. Create a working folder
```
mkdir airflow-demo
cd airflow-demo
mkdir airflow
```

2. create docker-compose.yaml in airflow-demo folder
```
version: '3.8'
services:
  metadb:
    container_name: airflow_metadb
    image: postgres
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    networks:
      - airflow
    restart: unless-stopped
    tty: true
    command: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_database:/var/lib/postgresql/data:Z
  scheduler:
    container_name: airflow_scheduler
    image: apache/airflow
    command: scheduler
    depends_on:
      - metadb
    networks:
      - airflow
    restart: unless-stopped
    volumes:
      - ./airflow:/opt/airflow
  webserver:
    container_name: airflow_webserver
    image: apache/airflow
    command: webserver
    depends_on:
      - metadb
    networks:
      - airflow
    ports:
      - 8080:8080
    restart: unless-stopped
    volumes:
      - ./airflow:/opt/airflow
    environment:
      _AIRFLOW_WWW_USER_USERNAME: admin
      _AIRFLOW_WWW_USER_PASSWORD: admin
     
networks:
  airflow:
    name: airflow_airflow

volumes:
  postgres_database:
      external: true    
```
3. create airflow.cfg in airflow-demo/airflow folder
```
[core]
sql_alchemy_conn = postgresql+psycopg2://airflow:airflow@airflow_metadb:5432/airflow
executor = LocalExecutor
```

4. create postgres_database volume
```
docker volume create --name=postgres_database
```

5. In airflow-demo folder
```
docker-compose up
docker exec -it airflow_webserver bash
# In airflow_webserver container, do the following:
airflow db init
FLASK_APP=airflow.www.app flask fab create-admin
airflow scheduler
```
