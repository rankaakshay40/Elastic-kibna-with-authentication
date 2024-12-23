# Elastic-kibna-with-authentication
Installing the Elastic and Kibana with authentication as docker containers


docker network create elastic-1 

6399c2ba4c152866ae41ac161acd23ed79874e93a80565faf19555de97424bf3

docker pull docker.elastic.co/elasticsearch/elasticsearch:8.17.0

wget https://artifacts.elastic.co/cosign.pub
cosign verify --key cosign.pub docker.elastic.co/elasticsearch/elasticsearch:8.17.0



docker run --name es01-8 --net elastic-1 -p 9200:9200 -d --restart unless-stopped -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.17.0


docker exec -it es01-8 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

Password for the [elastic] user successfully reset.
New value: 61GFTVqG3YVZc=ad93Zi


docker exec -it es01-8 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana


eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTcyLjIxLjAuMjo5MjAwIl0sImZnciI6IjZkNTdkZDUxZmQ1ZmM1ZTYzOWM3YTVlNTZiYzRiMW
FmYzg3NzhmOTgwYzRiNzk1MWFkZGRiMTJhYWU0ZjQ5ZTgiLCJrZXkiOiIxN0RZMUpNQmVUcGxiX3hDNWlsTDo4ZGV0bVp4Q1FMMkNOaGJGUDRxWlBRIn0=


mkdir elasticsearch

cd elasticsearch

 
docker cp es01:/usr/share/elasticsearch/config .
docker cp es01:/usr/share/elasticsearch/data .


export ELASTIC_PASSWORD="BvpoIGkN50M4ZiCE5Apm"


docker cp es01-8:/usr/share/elasticsearch/config/certs/http_ca.crt .


curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200


output:

root@ip-10-132-143-56:/home/ubuntu/elasticsearch# curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
{
  "name" : "f7f904d26c1d",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "7FkBjhdlRZGFddcMkLOHbA",
  "version" : {
    "number" : "8.17.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2b6a7fed44faa321997703718f07ee0420804b41",
    "build_date" : "2024-12-11T12:08:05.663969764Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}




------------------------------------------------------------------------

docker pull docker.elastic.co/kibana/kibana:8.17.0

docker run --name kibana-8 --net elastic-1 -p 5601:5601 -d docker.elastic.co/kibana/kibana:8.17.0

output:

7d63c44901318946b3ab57068637cd18853d5a076c48e966ddeefb2c508a14bb

docker exec -it es01-8 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana


eyJ2ZXIiOiI4LjE0LjAiLCJhZHIiOlsiMTcyLjIxLjAuMjo5MjAwIl0sImZnciI6IjZkNTdkZDUxZmQ1ZmM1ZTYzOWM3YTVlNTZiYzRiMWF
mYzg3NzhmOTgwYzRiNzk1MWFkZGRiMTJhYWU0ZjQ5ZTgiLCJrZXkiOiI1Skt1M1pNQmxyQURuRl9kRThSejo2ZTNobzV4RFM0U0sza0ZrMVFYUVpRIn0=


docker cp kibana-8:/usr/share/kibana/config .

docker cp kibana-8:/usr/share/kibana/data .



docker stop es01-8 kibana-8

docker rm es01-8 kibana-8

--------------------------------------------------------------------------------------

docker run --name es01-8 --net elastic-1 -p 9200:9200 -d --restart unless-stopped -e "discovery.type=single-node" -v /home/ubuntu/varada/elasticsearch8
/elasticsearch/config:/usr/share/elasticsearch/config -v /home/ubuntu/varada/elasticsearch8/elasticsearch/data:/usr/share/elasticsearch/data 
docker.elastic.co/elasticsearch/elasticsearch:8.17.0

This will start a new container with the name es01-8  with the docker network elastic-1 and maps port 9200 of the host machine to port 9200 in the contianer. It runs the container in a detached mode (container continues to run in the background even after the terminal session ends) configures the container to restart automatically unless it is explicitly stopped by the user and run on the single node. 
It mounts the host directory (/home/ubuntu/varada/elasticsearch8/elasticsearch/config) to the container directory (/usr/share/elasticsearch/config). The config directory in Elasticsearch contains configuration files. By mounting it, you can customize the Elasticsearch configuration from the host machine.
The data directory in Elasticsearch stores indices and other persistant data. This mount ensures data persists even if the container is removed.



curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200



docker run --name kibana-8 --net elastic-1 -p 5601:5601 -d -v /home/ubuntu/varada/elasticsearch8/kibana/config:/usr/share/kibana/config -v /home/ubuntu
/varada/elasticsearch8/kibana/data:/usr/share/kibana/data docker.elastic.co/kibana/kibana:8.17.0

-----------------------------------------------------------------------------------------------------------------------------------------------------------

When you mount a directory from your local machine to a Docker container using the -v option, you create a bind mount. This means that the specified directory on the host is directly linked to the specified directory inside the container. Any changes made in the mounted directory on the host are immediately reflected inside the container, and vice versa.


How It Works:

1. Bind Mount in Action:
   
In your command:This tells Docker:

The local host directory /home/ubuntu/varada/elasticsearch8/elasticsearch/config is mapped to /usr/share/elasticsearch/config inside the container.
Any files or subdirectories in the host directory /home/ubuntu/varada/elasticsearch8/elasticsearch/config will appear inside the container at /usr/share/elasticsearch/config.


2. Two-Way Sync:
   
Changes on the Host:

If you edit or add files in /home/ubuntu/varada/elasticsearch8/elasticsearch/config on the host, those changes are immediately visible in the container's /usr/share/elasticsearch/config.


Changes in the Container:

If a process in the container modifies /usr/share/elasticsearch/config, those changes are immediately reflected in the host directory /home/ubuntu/varada/elasticsearch8/elasticsearch/config.

Why This Happens:

Docker mounts the host directory into the container's filesystem at runtime.

This mapping is direct and live, meaning the container doesn't store its own separate copy of the directory—it directly accesses the host's directory.
   -v /home/ubuntu/varada/elasticsearch8/elasticsearch/config:/usr/share/elasticsearch/config










