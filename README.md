# Vagrant_DockerSwarm

## *Part 3* | DockerSwarm Framework

###  Here in this project i'll be dockerizing a Node.js application and then deploy 10 containers in a swarm cluster, load balance them behind a single IP with Nginx and then finally conclude with scaling them up and down also visualize the containers in a better way.  

### So the main purpose of this project is not the NodeJs app itself, but rather to  deployi an Node.js app to 10 containers in a swarm then load balance them.

## Content

### - <a title="Part 1" href="https://github.com/cstoicescu/WhoAmI_NodeJs_Docker" target="_blank">Part 1</a> 
### - <a title="Part 2" href="https://github.com/cstoicescu/NGINX_Docker" target="_blank">Part 2</a> 
### - <a title="Part 3" href="https://github.com/cstoicescu/Vagrant_DockerSwarm" target="_blank">Part 3</a> 
 
 ## Final Result after Deploy:  
 https://user-images.githubusercontent.com/53979557/120241164-d6bd8a80-c26a-11eb-81dd-e749cd34b591.gif


 ### Third Step:  Building an image from github repo and pushing it to Docker Hub

Vagrant Setup for 2 VM's ( manager and worker ) using DockerSwarm Orchestration Framework

### Tutorial  

https://user-images.githubusercontent.com/53979557/120244912-39675400-c274-11eb-81b2-80069416fbc2.mp4

## Final Result: 

![1](https://user-images.githubusercontent.com/53979557/120245103-d4602e00-c274-11eb-8b2a-3c7779557edb.png)  

Accessing Different Container Instances 

![2](https://user-images.githubusercontent.com/53979557/120245127-e215b380-c274-11eb-8101-f3677ee51a0c.png)

Docker Visualizer 
![3](https://user-images.githubusercontent.com/53979557/120245130-e641d100-c274-11eb-9a82-e4ed7dae92d7.png)

On your laptop you should have installed next software:
```
Vagrant 2.0.2 or higher: https://www.vagrantup.com/docs/installation/
VirtualBox 5.2.22 or higher: https://www.virtualbox.org/wiki/Downloads 
```

#### How to run it:  
To create initial machines ( manager and worker ) described in Vagrantfile navigate to root folder of the project and execute:  
```
vagrant up
```

Then check if the machines are up and running 
```
vagrant global-status

# output
id       name          provider   state   directory
--------------------------------------------------------------------------------------------------
6ecbdec  swarm_manager virtualbox running C:/Users/Catalin/Desktop/NodeJsApp/project/docker-swarm
37f08c6  swarm_worker  virtualbox running C:/Users/Catalin/Desktop/NodeJsApp/project/docker-swarm

```
Let's initialize our Docker Swarm cluster, we need to ssh in VirtualBox machine which will have a manager role
```
vagrant ssh swarm_manager
```
To initialize Docker Swarm you need to execute next command:
```
docker swarm init --advertise-addr 192.168.56.101

#output:
Swarm initialized: current node (p0abb7u72opnf80awrbmz6zl6) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-302q4wfo1s4j097iu94ypbvnsji8dfsk2imcx7j75nsojfj7eq-83spgwzi3uts27kdrripbr2m7 192.168.56.101:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
The output will contain a command which we need to execute on any other machine which we want to join to our Docker Swarm Cluster.
To check the list of node in swarm you should execute next command:
```
docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
f7ysphew4er8dkmvefkbot4vc *   swarm-manager       Ready               Active              Leader              18.06.1-ce
```
Now join the swarm_worker to cluster. We need to login to our worker VirtualBox machine by ssh
```
vagrant ssh swarm_worker
```
Then execute command which returned our manager during initialization:
```
docker swarm join --token SWMTKN-1-302q4wfo1s4j097iu94ypbvnsji8dfsk2imcx7j75nsojfj7eq-83spgwzi3uts27kdrripbr2m7 192.168.56.101:2377
```

Let's check the list of nodes now. You need to execute this command in swarm manager terminal:
```
In swarm_manager terminal:
docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
f7ysphew4er8dkmvefkbot4vc *   swarm-manager       Ready               Active              Leader              18.06.1-ce
0anl5bgtyxjtui3xwszab89j9     swarm-worker        Ready               Active                                  18.06.1-ce
```
You can see that now our cluster contains 2 machines(worker and manager).
To deploy our simple application described in docker-stack.yml file we need to execute command below in docker swarm terminal:
```
In swarm_manager terminal:
docker stack deploy -c=/vagrant_data/docker-compose.yml swarm-test --with-registry-auth
```
Let's check the list of stacks in our cluster.
```
docker stack ls
NAME                SERVICES            ORCHESTRATOR
swarm-test          2                   Swarm
```

You can see that nginx has 0 replicas(your number maybe different). This mean that deploy is still in progress.
If we will execute this command one more time, then we will see that amount of containers 
increased to 12 for nginx service.
```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                                PORTS
llzhdszeee9q        swarm-app_nodeapp   replicated          12/12               cstoicescu/whoami-nodejs-app:1.4     *:8081->8081/tcp
1943shaomd0x        swarm-app_proxy     replicated          1/1                 cstoicescu/nginx-balancer-conf:1.0   *:80->80/tcp
```

Let's initiate this desired state with scale command to change the amount of replicas for nginx service:
```
docker service scale swarm-test_nginx=15

``` 
To remove our stack you can use command below:
```
docker stack rm swarm-test
```
If we want worker machine to leave the swarm, then in worker terminal we need to execute:
```
docker swarm leave

# output
Node left the swarm.
```
If you want manager to leave the cluster then you should pass --force param
```
docker swarm leave
Error response from daemon: You are attempting to leave the swarm on a node that is participating as a manager. Removing the last manager erases all current state of the swarm. Use `--force` to ignore this message.

docker swarm leave --force
Node left the swarm
```
To stop 2 VirtualBox machine you should use command below:
```
vagrant halt
```
To completely remove 2 VirtualBox machines you should execute:
```
vagrant destroy
    swarm_worker: Are you sure you want to destroy the 'swarm_worker' VM? [y/N] y
==> swarm_worker: Destroying VM and associated drives...
    swarm_manager: Are you sure you want to destroy the 'swarm_manager' VM? [y/N] y
==> swarm_manager: Destroying VM and associated drives...
```  
In order to have a better visualization of running contianers we can use the docker visualizer. Simply run docker container run -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -d dockersamples/visualizer  
