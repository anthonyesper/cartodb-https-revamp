docker-cartodb-https
==============

[![](https://images.microbadger.com/badges/image/sverhoeven/cartodb.svg)](https://microbadger.com/#/images/sverhoeven/cartodb "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/sverhoeven/cartodb.svg)](https://hub.docker.com/r/sverhoeven/cartodb/)

This is an attempt to have a fully working https Cartodb Docker version of the https://hub.docker.com/r/sverhoeven/cartodb/ Docker solution as of May 2019 with the carto_db_production database and the Production environment. 

The default login is prod/pass1234. You may want to change it when you run it for the outside.

It also creates an 'example' organization with owner login admin4example/pass1234.
Organization members can be created on https://sub.domain.com/user/admin4example/organization

How to run the container:
-------------------------

```
sudo docker run -d -p 3000:3000 -p 8080:8080 -p 8181:8181 -h sub.domain.com --restart unless-stopped cartodb-https
```

You need to find and replace sub.domain.com with your domain in all configuration files. 

Nginx needs to run on the host and redirect all the 443 calls to the proper local ports using proxies. SSL certificates then need to be configured on the host. Note that if assets are served locally, the host needs to be able to access the assets of the container (/cartodb/assets/public). 


Persistent data
---------------

To persist the PostgreSQL data, the PostGreSQL data dir (/var/lib/postgresql) must be persisted outside the Cartodb Docker container.

One solution for this is to run the container, copy the /var/lib/postgresql/ data to the host in a folder called ~/cartodb_pgdata delete the container and then re-run it with 

```
docker run -d -p 3000:3000 -p 8080:8080 -p 8181:8181 -h sub.domain.com -v $PWD/cartodb_pgdata:/var/lib/postgresql --restart unless-stopped cartodb-https
```


After this the CartoDB container will have a database that stays filled after restarts.


Geocoder
--------

The external geocoders like heremaps, mapbox, mapzen or tomtom have dummy api keys and do not work.
No attempts have been made or will be made in this Docker image to get the external geocoders to work.

The internal geocoder is configured, but contains no data inside the image.

To fill the internal geocoder run
```
docker exec -ti <carto docker container id> bash -c /cartodb/script/fill_geocoder.sh
```

This will run the scripts described at https://github.com/CartoDB/data-services/tree/master/geocoder
It will use at least 5.7+7.8Gb of diskspace to download the dumps and import them.

How to build the image:
-----------------------

The image can be build with
```
git clone https://github.com/glaroc/docker-cartodb-https.git
docker build -t=cartodb-https docker-cartodb-https/
```

The build uses the master branches of the [CartoDB GitHub repositories](https://github.com/CartoDB). A fresh build may fail when code requires newer dependencies then the Dockerfile provides or when code is not stable at the moment of building.
