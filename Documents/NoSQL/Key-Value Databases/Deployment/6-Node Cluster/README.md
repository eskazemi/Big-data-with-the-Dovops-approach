
## 6-Node Cluster
We will talk about creating a Redis cluster using docker.
We will create a Redis cluster with 6 nodes(3 master+3 replica), and Redis will decide which nodes will be master or replica.
#### Redis Cluster and Docker
- Docker uses a technique called port mapping: programs running inside Docker containers may be exposed to a different port compared to the one the program believes to be using.
- To make Docker compatible with Redis Cluster, you need to use Docker’s host networking mode. Please see the — net=host option in the Docker documentation for more information.
- The host networking driver only works on Linux hosts and is not supported on Docker Desktop for Mac, Docker Desktop for Windows, or Docker EE for Windows Server.
- we can set up our cluster using a TCP proxy in front of all nodes. All communications to nodes will be through TCP proxy. For this example I picked [HAProxy]('http://github.com/eskazemi').

We have 3 files;
* docker-compose.yaml
* redis.sh
* haproxy.cfg
With these files you can run all cluster, proxy and UI with command:

    docker compose up

#### docker-compose.yaml
important points
 - 6 server nodes
- 1 cluster creater node. Status of this container should be “stopped” after it created cluster successfully. (if you wish you can exec into any container and run cluster command instead of this container).
- TCP proxy
- UI (redis insight)
 - 10.11.12.13 is IP of local machine(actually IP of TCP Proxy). You should change it (may be to your PCName)

#### redis.sh
important points
- Instead of creating separate Redis.conf file for each node, I created a sh file that generates a Redis.conf file according to given parameters then started the server with this file. 

- Each Redis server runs on port 6379 but their cluster announces IP and ports are different.

- cluster-announce-port is used for client-to-server communications.
- cluster-announce-bus-port is used for server-to-server communications.

#### haproxy.cfg

HAProxy listens to 9001–9006 ports for client access (client-to-server), and 9101–9106 ports for server access(server-to-server). HAProxy stats can be accessible from http://localhost:8404/stats

Access order:

client => haproxy-container:700x => haproxy:900x => redis-node-x:6379
redis-node-* => haproxy-container:710x => haproxy:910x => redis-node-x:16379