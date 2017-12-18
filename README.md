# RabbitMQ
## Installing RabbitMQ on RHEL 6 Linux server

### Message Brokers
Messaging is a way of exchanging certain data between processes, applications, and servers (virtual and physical). These messages exchanged, helping with certain engineering needs, can consist of anything from plain text messages to blobs of binary data serving to address different needs. For this to work, an interface managed by a third party program (a middleware) is needed… welcome Message Brokers.
Message Brokers are usually application stacks with dedicated pieces covering each stage of the exchange setup. From accepting a message to queuing it and delivering it to the requesting party, brokers handle the duty which would normally be much more cumbersome with non-dedicated solutions or simple hacks such as using a database, cron jobs, etc. They simply work by dealing with queues which technically constitute infinite buffers, to put messages and pop-and-deliver them later on to be processed either automatically or by polling.
#### Why use them?
These message broking solutions act like a middleman for various services (e.g. your web application). They can be used to greatly reduce loads and delivery times by web application servers since tasks, which would normally take quite bit of time to process, can be delegated for a third party whose sole job is to perform them (e.g. workers). They also come in handy when a more "guaranteed" persistence is needed to pass information along from one place to another.
#### When to use them?
All put together, the core functionality explained expands to cover a multitude of areas, including-but-not-limited-to:
•	Allowing web servers to respond to requests quickly instead of being forced to perform resource-heavy procedures on the spot
•	Distributing a message to multiple recipients for consumption (e.g. processing)
•	Letting offline parties (i.e. a disconnected user) fetch data at a later time instead of having it lost permanently
•	Introducing fully asynchronous functionality to the backend systems
•	Ordering and prioritizing tasks
•	Balancing loads between workers
•	Greatly increase reliability and uptime of your application
•	and much more

### RabbitMQ
RabbitMQ is one of the more popular message broker solutions in the market, offered with an open-source license (Mozilla Public License v1.1) as an implementation of Advanced Message Queuing Protocol. Developed using the Erlang language, it is actually relatively easy to use and get started. It was first published in early 2007 and has since seen an active development with its latest release being version 3.7.0 (November 2017).

### Installation
Linux and UNIX-like systems has environment variable called http_proxy. It allows you to connect text based session and/or applications via the proxy server. 
1. Login to the RHEL servers
2. Switch to root user
```
sudo bash 
Or
su -
```
3. Set the proxy server in case the server is behind a proxy.
```
export http_proxy=http://<IP_ADDRESS>:<PORT>
export https_proxy=http://<IP_ADDRESS>:<PORT>
```
If you are working in a corporate setup and have trouble with installing latest version, then we have to manually download the latest packages, keys and move the same to the servers and then install locally.

To verify installation packages, you must import the GPG key. To do so, execute the following command at a shell prompt:
The --import option to the rpm command imports the public key.
```
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```
Before installing RabbitMQ, you must install a supported version of Erlang/OTP. 
4. Install erlang and rabbitmq-server
```
yum install -y --nogpgcheck erlang
yum install -y --nogpgcheck rabbitmq-server*
```
5. The server is not started as a daemon by default when the RabbitMQ server package is installed. To start the daemon by default when the system boots, as an administrator run
```
chkconfig rabbitmq-server on
```
6. As an administrator, start and stop the server as usual:
```
/sbin/service rabbitmq-server start
/sbin/service rabbitmq-server stop
```
The server is set up to run as system user rabbitmq. If you change the location of the node database or the logs, you must ensure the files are owned by this user (and also update the environment variables).
Port Access, SELinux, and similar mechanisms may prevent RabbitMQ from binding to a port. When that happens, RabbitMQ will fail to start. Firewalls can prevent nodes and CLI tools from communicating with each other. Make sure the following ports can be opened:
•	4369: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
•	5672, 5671: used by AMQP 0-9-1 and 1.0 clients without and with TLS
•	25672: used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000). See networking guide for details.
•	15672: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled)
•	61613, 61614: STOMP clients without and with TLS (only if the STOMP plugin is enabled)
•	1883, 8883: (MQTT clients without and with TLS, if the MQTT plugin is enabled
•	15674: STOMP-over-WebSockets clients (only if the Web STOMP plugin is enabled)
•	15675: MQTT-over-WebSockets clients (only if the Web MQTT plugin is enabled)
It is possible to configure RabbitMQ to use different ports and specific network interfaces.

#### Managing the Broker
To stop the server or check its status, etc., you can use package-specific scripts (e.g. the service tool) or invoke rabbitmqctl (as an administrator). It should be available on the path. All rabbitmqctl commands will report the node absence if no broker is running.
•	Invoke rabbitmqctl stop to stop the server.
•	Invoke rabbitmqctl status to check whether it is running.

#### Logging
Output from the server is sent to a RABBITMQ_NODENAME.log file in the RABBITMQ_LOG_BASE directory. Additional log data is written to RABBITMQ_NODENAME-sasl.log.
The broker always appends to the log files, so a complete log history is retained.
You can use the logrotate program to do all necessary rotation and compression, and you can change it. By default, this script runs weekly on files located in default /var/log/rabbitmq directory. See /etc/logrotate.d/rabbitmq-server to configure logrotate.

7. Check the status
```
rabbitmqctl status
``` 
8. Start the server
```
rabbitmq-server start &
``` 
 9. Check the status
```
rabbitmqctl status
``` 
10. Check the process
```
ps -ef | grep rabbit
```
11. Stop the server
```
rabbitmq-server stop
``` 
12. Start the rabbitmq_management plugin
```
cd /usr/lib/rabbitmq/bin/
umask 0022
./rabbitmq-plugins enable rabbitmq_management

ps -ef | grep rabbit
rabbitmqctl status
```
13. Check whether RabbitMQ server is listening to default ports
5672, 5671: used by AMQP 0-9-1 and 1.0 clients without and with TLS
15672: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled)
```
netstat -tunap | grep 5672
netstat -tunap | grep 15672
``` 

14. Add user and required permissions
```
rabbitmqctl add_user test_user password123
rabbitmqctl set_user_tags test_user administrator
rabbitmqctl set_permissions -p / test_user ".*" ".*" ".*"
```
15. Stop the server
```
rabbitmqctl stop
rabbitmq-server stop &
ps aux | grep rabbit
netstat -tunap | grep 5672
```

#### Clustering
RabbitMQ nodes and CLI tools (e.g. rabbitmqctl) use a cookie to determine whether they are allowed to communicate with each other. For two nodes to be able to communicate they must have the same shared secret called the Erlang cookie. The cookie is just a string of alphanumeric characters up to 255 characters in size. It is usually stored in a local file. The file must be only accessible to the owner (e.g. have UNIX permissions of 600 or similar). Every cluster node must have the same cookie.

16. make sure the file .erlang.cookie has same content in both servers, if not copy the content from one server and paste in the .erlang.cookie file in next server.
```
/var/lib/rabbitmq/.erlang.cookie
```
##### Starting independent nodes
Clusters are set up by re-configuring existing RabbitMQ nodes into a cluster configuration. Hence the first step is to start RabbitMQ on all nodes in the normal way:

17. Create independent RabbitMQ brokers.
```
rabbitmq-server -detached
```
This creates three independent RabbitMQ brokers, one on each node, as confirmed by the cluster_status command:

18. Check the cluster status
```
rabbitmqctl cluster_status
``` 

#### Creating the cluster
In order to link up our two nodes in a cluster, we tell one of the nodes, rabbit@node02, to join the cluster of the other, rabbit@node01.
We first join rabbit@node02 in a cluster with rabbit@node01. To do that, on rabbit@node02 we stop the RabbitMQ application and join the rabbit@node01 cluster, then restart the RabbitMQ application. Note that joining a cluster implicitly resets the node, thus removing all resources and data that were previously present on that node.

19. Join the clusters
```
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@sn3-haj-rv-rq02
rabbitmqctl start_app
``` 
We can see that the two nodes are joined in a cluster by running the cluster_status command on either of the nodes:

20. Verify the cluster status
```
rabbitmqctl cluster_status
``` 


