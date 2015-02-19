
# Hammock Server.md
## General notes on how everything is glued together
- BeLL Apps is stored in CouchDB
- Hammock runs multiple CouchDBs, each on its own port (run `docker ps` to see which )
- Each Nation BeLL gets a "containerized" CouchDB (note the name in `docker ps`) based on the klaemo/couchdb image.
- All National BeLL's URLs resolve to the same server, http://hammock.media.mit.edu.  For example, if you are at http://somaliabell.ole.org:5987, that is the exact same as http://hammock.media.mit.edu:5987. BE WARNED, Earth BeLL is located at http://earthbell.ole.org:5989 but can also be accessed at http://somaliabell.ole.org:5989 .  http://somaliabell.ole.org:5989 is not somaliabell! It is earthbell! This goes the same for all nation bells on this server. You can get very mixed up if you are not making sure you are using the correct port.   
- For convenience to humans who need to access their nation bell app, `node /root/oleproxy/server.js` issues redirect from, for example, `http://somaliabell.ole.org`  to `http://somaliabell.ole.org:5987/apps/_design/bell/MyApp/index.html`. Note that this redirect changes the port to the correct port for that National BeLL's CouchDB and adds the URL fragment for the path to the app's index.
## OLE Hammock Recipe 
```
apt-get install curl;
curl -sSL https://get.docker.com/ | sh;
docker pull klaemo/couchdb:latest;
#old
docker run -d -p 5984:5984 --name frontdoor klaemo/couchdb;
docker run -d -p 5985:5984 --name earthbell-qa klaemo/couchdb;
docker run -d -p 5986:5984 --name nationalbell-qa klaemo/couchdb;
docker run -d -p 5987:5984 --name somaliabell klaemo/couchdb;
docker run -d -p 5988:5984 --name ubuntubell klaemo/couchdb;
docker run -d -p 5989:5984 --name earthbell klaemo/couchdb;
#new install command on dusttrack example
docker run -d -p 5999:5984 --name dusttrack -v /srv/data/dusttrack:/usr/local/var/lib/couchdb -v /srv/log/dusttrack:/usr/local/var/log/couchdb klaemo/couchdb
```
## Start OLE Hammock after a reboot
```
#old
sudo su;
screen -t oleproxy;
# list all the machines
docker ps -a;
# docker start all by Container ID, the IDs below may have changed
docker start c2d06b6c412a;
docker start 6f91d79d25a8;
docker start db8bb1d9f0e3;
docker start 12982ed0e22b;
docker start 215c1b764718;
#new gets handled by crontab and @reboot /root/hammock.sh
cat /root/hammock.sh  
#!/bin/sh
sleep 16
docker start dusttrack
sleep 3
docker start splash
sleep 3
docker start frontdoor
sleep 3
docker start earthbell
sleep 3
docker start ubuntubell
sleep 3
docker start somaliabell
sleep 3
docker start nationbell-qa
sleep 3
docker start earthbell-qa
sleep 3
docker start examplebell
node /root/oleproxy/server.js >> /var/log/oleproxy
...
```
## Software for managing Docker Containers
- http://www.projectatomic.io/
- http://decking.io/
- http://docs.ansible.com/docker_module.html
- https://www.getchef.com/solutions/docker/
- http://shipyard-project.com/
- https://coreos.com/docs/launching-containers/launching/launching-containers-fleet/
