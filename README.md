# MySql-Clustering-In-Docker
<p align="center">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=Mysql,linux,docker " />
  </a>
</p>



In this project sample, i need to dockerized MySql cluster 1 master read write and volumes 2TB and 5 cluster nodes x 2TB volumes per nodes and Readonly and activated synch between master and worker nodes.


To set up a Dockerized MySQL cluster with one master node for read-write operations and five worker nodes with 2TB volumes per node, along with synchronization between the master and worker nodes, you can follow these general steps:

1. **Create a Docker network**: Start by creating a Docker network that will allow communication between the master and worker nodes. Run the following command:
   ```
   docker network create mysql-cluster
   ```

2. **Create Docker volumes**: Create Docker volumes to be used by the MySQL containers. Run the following commands to create volumes for the master and worker nodes:
   ```
   docker volume create master-volume --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   docker volume create worker-volume1 --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   docker volume create worker-volume2 --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   docker volume create worker-volume3 --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   docker volume create worker-volume4 --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   docker volume create worker-volume5 --driver local --opt type=tmpfs --opt device=tmpfs --opt o=size=2T
   ```

3. **Set up the MySQL master node**:
   - Create a Docker container for the MySQL master node using the `mysql:latest` image. Mount the `master-volume` created in step 2 as the MySQL data directory.
   - Expose the necessary ports for MySQL access.
   - Configure MySQL replication settings to enable master-slave synchronization.
   - Start the container. Here's an example of the command:
     ```
     docker run -d --name mysql-master --net=mysql-cluster -v master-volume:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=your_root_password -e MYSQL_REPLICATION_MODE=master -e MYSQL_REPLICATION_USER=repl_user -e MYSQL_REPLICATION_PASSWORD=repl_password mysql:latest
     ```

4. **Set up the MySQL worker nodes**:
   - Create Docker containers for each MySQL worker node, similar to the master node setup.
   - Mount the respective `worker-volumeX` (X represents the node number) volumes as the MySQL data directories.
   - Expose MySQL ports and configure replication settings to enable slave synchronization with the master.
   - Start the containers. Here's an example command for a single worker node:
     ```
     docker run -d --name mysql-worker1 --net=mysql-cluster -v worker-volume1:/var/lib/mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=your_root_password -e MYSQL_REPLICATION_MODE=slave -e MYSQL_REPLICATION_MASTER_HOST=mysql-master -e MYSQL_REPLICATION_MASTER_PORT=3306 -e MYSQL_REPLICATION_USER=repl_user -e MYSQL_REPLICATION_PASSWORD=repl_password mysql:latest
     ```

5. **Configure read-only routing**: To direct read-only traffic to the worker nodes, you can set up a reverse proxy or load balancer. Options like HAProxy or Nginx can be used for this purpose. Configure them to distribute the read-only queries among the worker nodes.

That's the general outline of setting up a Dockerized MySQL cluster with one master node, five worker nodes, and synchronization between them. Remember to adjust the configurations and parameters based on your specific requirements.



**Configure read-only routing,** To direct read-only traffic to the worker nodes, you can set up a reverse proxy or load balancer. Options like HAProxy used for this purpose. Configure them to distribute the read-only queries among the worker nodes.

Here's an example of configuring HAProxy to distribute read-only traffic among the worker nodes in your Dockerized MySQL cluster:

1. **Install HAProxy**: Install HAProxy on a separate machine or server that will act as the reverse proxy. You can refer to the HAProxy documentation for installation instructions specific to your operating system.
* Sure! I can help you with the installation and configuration of HAProxy as a reverse proxy on a separate machine or server. Here's a step-by-step guide:

**Step 1: Install HAProxy**

1. Connect to your target machine or server using SSH or any other preferred method.

2. Update the system package repository by running the following command:
   ```
   sudo apt update
   ```

3. Install HAProxy by executing the following command:
   ```
   sudo apt install haproxy
   ```

**Step 2: Configure HAProxy**

1. Open the HAProxy configuration file using a text editor. For example:
   ```
   sudo nano /etc/haproxy/haproxy.cfg
   ```

2. Inside the configuration file, you'll find various sections for global settings, defaults, frontend, and backend configurations.

3. Modify the configuration based on your requirements.The explanation is provided in step 2 below. Here's a basic example configuration that you can use as a starting.

4. Save the configuration file and exit the text editor.

**Step 3: Restart HAProxy**

After configuring HAProxy, you need to restart the service for the changes to take effect. Execute the following command:
```
sudo service haproxy restart
```

That's it! HAProxy is now installed and configured as a reverse proxy on your separate machine or server. You can customize the configuration further based on your needs and add additional backend servers as required.





2. **Configure HAProxy**: Once HAProxy is installed, create a configuration file, typically named `haproxy.cfg`, and edit it with the following content:

   ```
   global
       log /dev/log local0
       log /dev/log local1 notice
       maxconn 4096
       user haproxy
       group haproxy
       daemon
   
   defaults
       log global
       mode http
       option httplog
       option dontlognull
       retries 3
       timeout connect 5000
       timeout client 50000
       timeout server 50000
   
   frontend mysql-cluster
       bind <HAProxy_IP>:3306  # Replace <HAProxy_IP> with the IP address of the HAProxy machine.
       default_backend mysql-servers
   
   backend mysql-servers
       mode tcp
       balance roundrobin
   
       server mysql-worker1 <Worker1_IP>:3306 check
       server mysql-worker2 <Worker2_IP>:3306 check
       server mysql-worker3 <Worker3_IP>:3306 check
       server mysql-worker4 <Worker4_IP>:3306 check
       server mysql-worker5 <Worker5_IP>:3306 check
   ```

   - Replace `<HAProxy_IP>` with the IP address of the machine where HAProxy is installed.
   - Replace `<WorkerX_IP>` with the IP addresses of the worker nodes in your MySQL cluster.

3. **Start HAProxy**: Start the HAProxy service using the command appropriate for your operating system.

4. **Update Application**: Update your application's MySQL connection settings to point to the HAProxy IP address and port (e.g., `<HAProxy_IP>:3306`) for read-only operations.

With this configuration, HAProxy will distribute incoming read-only queries among the worker nodes in a round-robin fashion, providing load balancing and high availability for your MySQL cluster.

Remember to adjust the configuration based on your specific environment and requirements.
