
# ELK Stack Installation Guide on 3 Nodes

This setup installs Elasticsearch on each node (192.168.56.101, 192.168.56.102, and 192.168.56.103), with Kibana and Logstash configured only on the primary node (192.168.56.101). This design enables distributed Elasticsearch services while centralizing data visualization and ingestion.

## 1. Pre-requisites

- Ensure all nodes have Java installed.
- Set up `/etc/hosts` on each node with mappings for all IP addresses to enable smooth inter-node communication:

    ```plaintext
    192.168.56.101  elkbim-1
    192.168.56.102  elkbim-2
    192.168.56.103  elkbim-3
    ```

- Configure firewalls to allow traffic on necessary ports:
  - Elasticsearch HTTP API: 9200
  - Elasticsearch transport: 9300
  - Kibana: 5601
  - Logstash (if using Beats): 5044

- Synchronize the system clock on all nodes using `ntpd` or `chronyd` to prevent cluster timing issues.

---

## 2. LVM Setup (Storage Configuration)

If additional storage is needed for Elasticsearch, use LVM for managing space efficiently:

1. **Create and Configure Logical Volume**:

   ```bash
   df
   
   unmount /data
   
   lsblk
   
   pvcreate /dev/sdb1
   
   vgcreate mnggroup01 /dev/sdb1
   
   lvcreate -l +100%free -n mng mnggroup01
   
   mkfs.xfs /dev/mapper/mnggroup01-mng
   ```

2. **Mount New Storage**:

   ```bash
   mount /dev/mapper/mnggroup01-mng /data
   ```

3. **Persist Mount in `/etc/fstab`**:

   ```plaintext
   /dev/mapper/mnggroup01-mng  /data  xfs  defaults  0  0
   ```

   

------



## 3. Elasticsearch Installation

### Node 1 (192.168.56.101)

1. **Install Elasticsearch**:
    ```bash
    rpm -ivh elasticsearch-8.10.2-x86_64.rpm
    ```

2. **Initial Configuration**:
    Configure security settings, including enabling authentication and setting TLS on transport and HTTP layers.

3. **Edit Configuration** (`/etc/elasticsearch/elasticsearch.yml`):

    ```yaml
    cluster.name: elkbim
    
    node.name: elkbim-1
    
    path.data: /data/elastic
    
    network.host: 192.168.56.101
    
    http.port: 9200
    
    discovery.seed_hosts: ["192.168.56.101:9300"]
    ```

4. **Create Data Directory**:
    ```bash
    mkdir -p /data/elastic
    chown elasticsearch:elasticsearch /data/elastic
    ```

5. **Start and Verify Elasticsearch**:
    ```bash
    systemctl start elasticsearch.service
    
    curl -k -u elastic 'elastic_password' -X GET http://192.168.56.101:9200/_cat/nodes
    ```

### Node 2 (192.168.56.102)

1. **Install Elasticsearch** and **Edit Configuration**:
    Similar to Node 1, configure the following in `/etc/elasticsearch/elasticsearch.yml`:
    ```yaml
    cluster.name: elkbim
    
    node.name: elkbim-2
    
    path.data: /data/elastic
    
    network.host: 192.168.56.102
    
    http.port: 9200
    
    discovery.seed_hosts: ["192.168.56.101:9300", "192.168.56.102:9300"]
    ```

2. **Data Directory**:
    ```bash
    mkdir -p /data/elastic
    chown elasticsearch:elasticsearch /data/elastic
    ```

3. **Start Elasticsearch**:
    ```bash
    systemctl start elasticsearch.service
    ```

4. **Verification**:
    ```bash
    curl -k -u elastic 'elastic_password' -X GET http://192.168.56.102:9200/_cat/nodes
    ```

### Node 3 (192.168.56.103)

1. **Install Elasticsearch** and **Configure**:
    ```yaml
    cluster.name: elkbim
    
    node.name: elkbim-3
    
    path.data: /data/elastic
    
    network.host: 192.168.56.103
    
    http.port: 9200
    
    discovery.seed_hosts: ["192.168.56.101:9300", "192.168.56.102:9300", "192.168.56.103:9300"]
    ```

2. **Data Directory**:

    ```bash
    mkdir -p /data/elastic
    chown elasticsearch:elasticsearch /data/elastic
    ```

3. **Start Elasticsearch** and **Verify Cluster Health**:
    ```bash
    systemctl start elasticsearch.service
    
    curl -k -u elastic 'elastic_password' -X GET http://192.168.56.103:9200/_cluster/health?pretty
    ```

### Cluster Enrollment Tokens

- **Node 1**:
    ```bash
    /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
    ```

- **Node 2 and Node 3**:
    Use the generated token on Nodes 2 and 3:
    ```bash
    /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>
    ```

---

## 4. Kibana Installation (192.168.56.101 Only)

1. **Install Kibana**:
    ```bash
    rpm -ivh kibana-8.10.2-x86_64.rpm
    ```

2. **Configure Kibana** (`/etc/kibana/kibana.yml`):
    ```yaml
    server.port: 5601
    
    server.host: "192.168.56.101"
    
    elasticsearch.hosts: ["http://192.168.56.101:9200"]
    
    elasticsearch.username: "kibana_system"
    
    elasticsearch.password: "kibana_password"
    
    server.ssl.enabled: false
    
    elasticsearch.ssl.verificationMode: none
    ```

3. **Start Kibana**:
    ```bash
    systemctl start kibana.service
    ```

4. **Generate Kibana Enrollment Token**:
    ```bash
    /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
    ```

---

## 5. Logstash Installation (192.168.56.101 Only)

1. **Install Logstash**:
    ```bash
    rpm -ivh logstash-8.15.0-x86_64.rpm
    ```

2. **Configure Logstash**:
    - Set log level to minimize verbosity in `/etc/logstash/logstash.yml`:
    ```yaml
    log.level: error
    ```

3. **Manage Local YUM Repositories (if required)**:
    Transfer required RPM files with WinSCP, configure `local.repo` under `/etc/yum.repos.d/`, and ensure dependencies.

    ```bash
    # Transfer required RPM files with WinSCP to /ogg
    
    cd /ogg
    
    unzip mnt.zip
    
    cd /ogg/mnt/mnt/AppStream/
    
    ll
    
    cd /etc/yum.repo.d/
    
    mkdir oldrepos
    
    mv oracle-linux-ol8.repo oldrepos/
    
    mv uek-ol8.repo oldrepos/
    
    mv virt-ol8.repo oldrepos/
    
    vim local.repo
    ```

    Set config for local.repo

    ```yml
    [OracleLinux-base]
    
    name=OracleLinux-base
    
    baseurl=file:///ogg/mnt/mnt/BaseOS/
    
    #https:
    
    enabled=1
    
    gpgcheck=0
    
    #gpgkey=https://yum.oracle.com/RPM-GPG-KEY-oracle
    
    #repo_gpgkey=0
    
    #priority=1
    
    ssl_verify=0
    
    #skip_if_unavailable
    
    
    
    [OracleLinux-appstream]
    
    name=OracleLinux-appstream
    
    baseurl=file:///ogg/mnt/mnt/AppStream/
    
    #https:
    
    enabled=1
    
    gpgcheck=0
    
    #gpgkey=https://yum.oracle.com/RPM-GPG-KEY-oracle
    
    #repo_gpgkey=0
    
    #priority=1
    
    ssl_verify=0
    
    #skip_if_unavailable
    ```

    Install Oracle Instant Client with Local YUM Repositories for use JDBC Plugin in Logstash

    ```bash
    yum clean all
    
    yum update
    
    dnf install -y libnsl
    
    rpm -ivh oracle-instantclient19.21-basic-19.21.0.0.0-1.x86_64.rpm
    ```

    If you have Internet on Node 1 

    ```bash
    # You don't need to steps for Local YUM Repositories.Just run this code
    
    rpm -ivh oracle-instantclient19.21-basic-19.21.0.0.0-1.x86_64.rpm
    ```

    

4. **Start Logstash**:
    ```bash
    systemctl start logstash.service
    ```

---

## 6. Service Enablement

1. Enable services in Node 1:

   ```bash
   systemctl enable elasticsearch.service
   
   systemctl enable kibana.service
   
   systemctl enable logstash.service
   ```

   

2. Enable service in Node 2 and Node 3:

   ```bash
   systemctl enable elasticsearch.service
   ```

------

With this setup, you should have a distributed Elasticsearch cluster, with Kibana and Logstash centralized on Node 1, supporting a scalable and resilient ELK stack environment.