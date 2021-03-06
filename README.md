# docker-mariadb-10.0-galera
Multi Master Replication using MariaDB 10.0 and Galera inside Docker.

Requires at least 3 nodes.

Let's say server1, server2, server3. You will have 2 data containers (data+config) and 1 container running MariaDB and Galera on each hosts.

# 1 - Config and data containers
### All Servers

##### Build Image
```
docker build https://github.com/MineYourMind/docker-mariadb-10.0-galera.git
```


##### Make a config container that will be accessible from the host and the container.
```
docker run --name mariadb-config -v /var/configs/mariadb/conf.d:/etc/mysql/conf.d busybox true
                                            ^^^^^^                      ^^^^^^
                                            Host directory               Container directory
```

##### Make a data container that will be accessible from the host and the container.
```
docker run --name mariadb-data -v /var/data/mariadb:/data busybox true
                                            ^^^^^^^                      ^^^^^^
                                            Host directory               Container directory
```

##### 

##### Make a ssh container that will be accessible from the host and the container.
```
docker run --name mariadb-ssh -v /var/configs/mariadb/.ssh:/root/.ssh busybox true
                                            ^^^^^^^                      ^^^^^^
                                            Host directory               Container directory
```

#####


# 2 - Config
```
sudo docker tag <imageid> factual/mariadb-galera
cd /var/configs/mariadb/conf.d
sudo nano cluster.cnf
```
Change <IP> to IP you want
Change <NODE>'s to other nodes
Do this on all servers

# 3 - Initial startup

Start the first server with (sometimes requires "sudo docker restart mariadb-srv"):
```
docker run -t -i -d --net=host --privileged=true --volumes-from mariadb-config --volumes-from mariadb-data --volumes-from mariadb-ssh -v /etc/timezone:/etc/timezone:ro -e "TZ=Europe/Berlin" --name mariadb-srv factual/mariadb-galera /bin/start new
```

Start other servers with :
```
# other servers
docker run -t -i -d --net=host --privileged=true --volumes-from mariadb-config --volumes-from mariadb-data --volumes-from mariadb-ssh -v /etc/timezone:/etc/timezone:ro -e "TZ=Europe/Berlin" --name mariadb-srv factual/mariadb-galera /bin/start node
```

# 4 - Restart server1 in "node mode"
It is very important to restart the first node just like the other. Otherwise if you stop and start your container, you will create a new cluster each time.

```
docker stop mariadb-srv
docker rm mariadb-srv
docker run -t -i -d --net=host --privileged=true --volumes-from mariadb-config --volumes-from mariadb-data --volumes-from mariadb-ssh -v /etc/timezone:/etc/timezone:ro -e "TZ=Europe/Berlin" --name mariadb-srv factual/mariadb-galera /bin/start node
```

# 5 - Debug
If anything goes wrong, you can always debug via the error.log
```
tail -f /var/data/mariadb/error.log
```
