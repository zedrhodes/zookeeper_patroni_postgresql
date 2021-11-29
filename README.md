# zookeeper_patroni_postgresql

```
Deep Dive Details: 

Research ~

https://habr.com/en/post/527370/
https://dev.to/jv/zookeeper-cluster-with-docker-compose-jml


Execution ~
docker login

docker swarm init

docker node ls

ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
soq59vaq7gtkc3kn5mnd5drbx *   docker-desktop   Ready     Active         Leader           20.10.10

docker pull zookeeper:latest

docker stack deploy --compose-file docker-compose.yml patroni

docker service ls

ID             NAME                   MODE         REPLICAS   IMAGE                          PORTS
6y65p4ar67fr   patroni_zk1            replicated   1/1        bitnami/zookeeper:3.6.2        *:21811->2181/tcp
zjhgfeig2zpl   patroni_zk2            replicated   1/1        bitnami/zookeeper:3.6.2        *:21812->2181/tcp
ywctsy48n2m2   patroni_zk3            replicated   1/1        bitnami/zookeeper:3.6.2        *:21813->2181/tcp
hxkn3ov5aikk   patroni_zoonavigator   replicated   1/1        elkozmon/zoonavigator:latest   *:9000->9000/tcp


Docker GUI should look like 
 

To get the node IP address you can use below command:

docker node inspect self --format '{{ .Status.Addr  }}'

To get the service IP address:

docker service inspect psw7tjy2fq7o | Grep Addr

To get the container IP address, use:

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' soq59vaq7gtkc3kn5mnd5drbx

To review the logs 
docker service logs 6y65p4ar67fr
docker service logs zjhgfeig2zpl
docker service logs ywctsy48n2m2
docker service logs hxkn3ov5aikk 

echo mntr | nc localhost 21811
echo mntr | nc localhost 21812
echo mntr | nc localhost 21813

(copy in the files from repo) 
mkdir patroni-testcd patroni-test
docker build --build-arg HTTP_PROXY="http://proxyout.mycompany.gov:8080" --build-arg HTTPS_PROXY="http://proxyout.mycompany.gov:8080" --build-arg FTP_PROXY="ftp://proxyout.mycompany.gov:8080" -t patroni-test .

 


mkdir patroni1
mkdir patroni2
mkdir patroni3
mkdir ../patroni1
mkdir ../patroni2
mkdir ../patroni3

change permissions to 750
cd ../
docker stack deploy --compose-file docker-compose-patroni.yml patroni


patroni-test % docker service ls
ID             NAME                   MODE         REPLICAS   IMAGE                          PORTS
31j12bsjfe9a   patroni_patroni1       replicated   1/1        patroni-test:latest            *:5441->5432/tcp, *:8091->8091/tcp
8cyytsljb6ef   patroni_patroni2       replicated   1/1        patroni-test:latest            *:5442->5432/tcp, *:8092->8091/tcp
082azq3oa46b   patroni_patroni3       replicated   1/1        patroni-test:latest            *:5443->5432/tcp, *:8093->8091/tcp
6y65p4ar67fr   patroni_zk1            replicated   1/1        bitnami/zookeeper:3.6.2        *:21811->2181/tcp
zjhgfeig2zpl   patroni_zk2            replicated   1/1        bitnami/zookeeper:3.6.2        *:21812->2181/tcp
ywctsy48n2m2   patroni_zk3            replicated   1/1        bitnami/zookeeper:3.6.2        *:21813->2181/tcp
hxkn3ov5aikk   patroni_zoonavigator   replicated   1/1        elkozmon/zoonavigator:latest   *:9000->9000/tcp



docker service ps --no-trunc 31j12bsjfe9a
docker service logs 31j12bsjfe9a
docker service logs 8cyytsljb6ef
docker service logs 082azq3oa46b 

docker container ls
CONTAINER ID   IMAGE                          COMMAND                  CREATED          STATUS                    PORTS                                    NAMES
47df482ddcb5   patroni-test:latest            "bin/sh /entrypoint.…"   32 seconds ago   Up 31 seconds             5432/tcp                                 patroni_patroni3.1.bh24m8sr5tklaf9wpfv9tbhm3
ab8a232bd94b   patroni-test:latest            "bin/sh /entrypoint.…"   47 seconds ago   Up 45 seconds             5432/tcp                                 patroni_patroni2.1.we8vb14yz2n398xug4xvrlcsp
9453720d626b   patroni-test:latest            "bin/sh /entrypoint.…"   49 seconds ago   Up 48 seconds             5432/tcp                                 patroni_patroni1.1.bd43hlak2puposvfrw3u612ht
[...]
docker exec -ti 47df482ddcb5 /bin/bash

docker exec -ti ab8a232bd94b /bin/bash

docker exec -ti 9453720d626b /bin/bash

pn2034313 PostgreSQL_Patroni_Zookeper % docker exec -ti 47df482ddcb5 /bin/bash
postgres@patroni3:/$ patronictl -c /etc/patroni.yml list patroni
+ Cluster: patroni (7035796090343002144) --+----+-----------+
| Member   | Host      | Role    | State   | TL | Lag in MB |
+----------+-----------+---------+---------+----+-----------+
| patroni1 | 10.0.2.17 | Leader  | running |  1 |           |
| patroni2 | 10.0.2.19 | Replica | running |  1 |         0 |
| patroni3 | 10.0.2.21 | Replica | running |  1 |         0 |
+----------+-----------+---------+---------+----+-----------+

exit out 

docker build --build-arg HTTP_PROXY="http://proxyout.mycompany.gov:8080" --build-arg HTTPS_PROXY="http://proxyout.mycompany.gov:8080" --build-arg FTP_PROXY="ftp://proxyout.mycompany.gov:8080" -t haproxy-patroni .

docker stack deploy --compose-file docker-compose-haproxy.yml patroni

docker ps | grep haproxy
ab8f6c1e1c82   haproxy-patroni:latest         "docker-entrypoint.s…"   12 seconds ago      Up 10 seconds                                                      patroni_haproxy.1.wnemmv5iypnyn4ugwgrmx8vdo

docker exec -ti ab8f6c1e1c82 /bin/bash

 
exit
docker ps | grep zoonavigator
50300c4a52b0   elkozmon/zoonavigator:latest   "./run.sh"               2 hours ago      Up 2 hours (healthy)   9000/tcp                                 patroni_zoonavigator.1.xqs2mpu5ydlc4r08fri5q1ui4
a88455f41030   elkozmon/zoonavigator:latest   "./run.sh"               3 hours ago      Up 2 hours (healthy)   9000/tcp                                 patroni_zoonavigator.1.xz0bts33ug0xxiyaslvhy16oi

docker exec -u 0 -ti 50300c4a52b0  /bin/bash

Verify the Patroni REST API is accessible with CURL https://patroni.readthedocs.io/en/latest/rest_api.html#restart-endpoint
curl -s patroni1:8091/patroni
{"patroni": {"scope": "patroni", "version": "2.1.1"}, "timeline": 1, "replication": [{"application_name": "patroni2", "sync_priority": 0, "usename": "replicator", "client_addr": "10.0.2.19", "sync_state": "async", "state": "streaming"}, {"application_name": "patroni3", "sync_priority": 0, "usename": "replicator", "client_addr": "10.0.2.21", "sync_state": "async", "state": "streaming"}], "cluster_unlocked": false, "xlog": {"location": 100665496}, "database_system_identifier": "7035796090343002144", "role": "master", "server_version": 110014, "postmaster_start_time": "2021-11-29 01:22:21.207681+00:00", "state": "running"}


apt-get install postgresql-client

psql --host patroni1 --port 5432 -U approle -d postgre

The password is appass

CREATE TABLE accounts (
	user_id serial PRIMARY KEY,
	username VARCHAR ( 50 ) UNIQUE NOT NULL,
	password VARCHAR ( 50 ) NOT NULL,
	email VARCHAR ( 255 ) UNIQUE NOT NULL,
	created_on TIMESTAMP NOT NULL,
        last_login TIMESTAMP 
);
CREATE TABLE roles(
   role_id serial PRIMARY KEY,
   role_name VARCHAR (255) UNIQUE NOT NULL
);
CREATE TABLE account_roles (
  user_id INT NOT NULL,
  role_id INT NOT NULL,
  grant_date TIMESTAMP,
  PRIMARY KEY (user_id, role_id),
  FOREIGN KEY (role_id)
      REFERENCES roles (role_id),
  FOREIGN KEY (user_id)
      REFERENCES accounts (user_id)
);

 


Database now has data and structure. 
root@50300c4a52b0:/app# pg_dump --host patroni1 --port 5432 -U approle -d postgres > dump.sql
Password:



cat dump.sql | more


-
-- PostgreSQL database dump
--

-- Dumped from database version 11.14 (Debian 11.14-1.pgdg90+1)
-- Dumped by pg_dump version 13.5 (Debian 13.5-0+deb11u1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

SET default_tablespace = '';

--
-- Name: account_roles; Type: TABLE; Schema: public; Owner: approle
--

CREATE TABLE public.account_roles (
    user_id integer NOT NULL,
    role_id integer NOT NULL,
    grant_date timestamp without time zone
);


ALTER TABLE public.account_roles OWNER TO approle;

--
-- Name: accounts; Type: TABLE; Schema: public; Owner: approle
--

CREATE TABLE public.accounts (
    user_id integer NOT NULL,
    username character varying(50) NOT NULL,
    password character varying(50) NOT NULL,
    email character varying(255) NOT NULL,
    created_on timestamp without time zone NOT NULL,
    last_login timestamp without time zone
);
[...]
CREATE TABLE public.roles (
    role_id integer NOT NULL,
    role_name character varying(255) NOT NULL
);


ALTER TABLE public.roles OWNER TO approle;

--
-- Name: roles_role_id_seq; Type: SEQUENCE; Schema: public; Owner: approle
--

CREATE SEQUENCE public.roles_role_id_seq
    AS integer
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.roles_role_id_seq OWNER TO approle;

--
-- Name: roles_role_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: approle
--

ALTER SEQUENCE public.roles_role_id_seq OWNED BY public.roles.role_id;


--
-- Name: accounts user_id; Type: DEFAULT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.accounts ALTER COLUMN user_id SET DEFAULT nextval('public.accounts_user_id_seq'::regclass);
[...]
COPY public.account_roles (user_id, role_id, grant_date) FROM stdin;
\.


--
-- Data for Name: accounts; Type: TABLE DATA; Schema: public; Owner: approle
--

COPY public.accounts (user_id, username, password, email, created_on, last_login) FROM stdin;
1	ed	pass	rose.ej@gmail.com	2021-11-27 00:00:00	2021-11-27 00:00:00

[...]
--
-- Name: accounts accounts_username_key; Type: CONSTRAINT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.accounts
    ADD CONSTRAINT accounts_username_key UNIQUE (username);


--
-- Name: roles roles_pkey; Type: CONSTRAINT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.roles
    ADD CONSTRAINT roles_pkey PRIMARY KEY (role_id);


--
-- Name: roles roles_role_name_key; Type: CONSTRAINT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.roles
    ADD CONSTRAINT roles_role_name_key UNIQUE (role_name);


--
-- Name: account_roles account_roles_role_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.account_roles
    ADD CONSTRAINT account_roles_role_id_fkey FOREIGN KEY (role_id) REFERENCES public.roles(role_id);


--
-- Name: account_roles account_roles_user_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: approle
--

ALTER TABLE ONLY public.account_roles
    ADD CONSTRAINT account_roles_user_id_fkey FOREIGN KEY (user_id) REFERENCES public.accounts(user_id);


--
-- PostgreSQL database dump complete
--





TEST FAILOVER:

docker service scale patroni_patroni3=0

pn2034313 PostgreSQL_Patroni_Zookeper % docker exec -ti 47df482ddcb5 /bin/bash
postgres@patroni3:/$ patronictl -c /etc/patroni.yml list patroni

postgres@patroni2:/$ patronictl -c /etc/patroni.yml list patroni
+ Cluster: patroni (7035796090343002144) --+----+-----------+
| Member   | Host      | Role    | State   | TL | Lag in MB |
+----------+-----------+---------+---------+----+-----------+
| patroni1 | 10.0.2.17 | Leader  | running |  1 |           |
| patroni2 | 10.0.2.19 | Replica | running |  1 |         0 |
+----------+-----------+---------+---------+----+-----------+


Restore the cluster member
postgres@patroni2:/$ exit
exit
ejrose@pn2034313 PostgreSQL_Patroni_Zookeper % docker service scale patroni_patroni3=1
patroni_patroni3 scaled to 1
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
ejrose@pn2034313 PostgreSQL_Patroni_Zookeper % docker exec -ti ab8a232bd94b /bin/bash
postgres@patroni2:/$ patronictl -c /etc/patroni.yml list patroni
+ Cluster: patroni (7035796090343002144) --+----+-----------+
| Member   | Host      | Role    | State   | TL | Lag in MB |
+----------+-----------+---------+---------+----+-----------+
| patroni1 | 10.0.2.17 | Leader  | running |  1 |           |
| patroni2 | 10.0.2.19 | Replica | running |  1 |         0 |
| patroni3 | 10.0.2.6  | Replica | running |  1 |         0 |
+----------+-----------+---------+---------+----+-----------+
```

