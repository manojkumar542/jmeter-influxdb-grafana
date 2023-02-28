## Monitoring using jmeter,influxDB & Grafana
* It automates jmeter executions and shows live results in grafana dashboard using docker compose easily
* Create a folder under which docker-compose file resides and add grafana,influxdb and jmeter folder to store the dockerfile and concerned config and data files
* Tweak the jmeter script and data files according to your need and i'm using sample jpetstore demo jmeter script file for execution.
* Update the jmeter dockerfile accordingly based on your requirement 
* Update the docker compose file according to your need.
* Use docker container name to establish connection between containers.

Setup as below

* run **docker-compose build**
* **docker-compose up -d**
* **docker-compose down**
* https://wp.me/p7eHc4-w3
<!-- wp:paragraph -->
<p>With a single <strong>docker-compose up -d </strong>command we can setup our jmeter execution and monitor the live results in grafana and saves the raw and html report of the jmeter execution in your host machine</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Create a folder under which docker-compose file resides and add grafana,influxdb and jmeter folder to store the dockerfile and concerned config and data files accordingly on the <a rel="noreferrer noopener" href="https://github.com/manojkumar542/Monitoringtoolkit" target="_blank">github repo</a> <strong> </strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>For setting up only jmeter docker image please refer the below links</strong> <a href="https://perfdevops.wordpress.com/2020/02/01/setup-jmeter-in-linux-docker-container-with-all-plugins/">https://perfdevops.wordpress.com/2020/02/01/setup-jmeter-in-linux-docker-container-with-all-plugins/</a> </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><a href="https://github.com/manojkumar542/Jmeter-docker">https://github.com/manojkumar542/Jmeter-docker</a> </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Basic of docker-compose file</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"yaml"} -->
<pre class="wp-block-syntaxhighlighter-code">&lt;strong>Docker compose file&lt;/strong>
version: '3.8'
services:
  influxdb:
    image: 'influxdb:1.8.9-alpine'
    container_name: influxdb
    restart: always
    ports: 
      - '8086:8086'
    networks:
      - monitoring
    environment:
      - INFLUXDB_DB=jmeter
    volumes:
      - influxdb-storage:/var/lib/influxdb
    
  grafana:
    build:
      context: ./grafana
    image: 'grafana:8.1.0'
    container_name: grafana
    ports:
      - '3000:3000'
    restart: always
    networks: 
      - monitoring
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=grafana
      - GF_INSTALL_PLUGINS=grafana-clock-panel,briangann-gauge-panel,natel-plotly-panel,grafana-simple-json-datasource
    volumes:
      - grafana-volume:/etc/grafana/provisioning
      - grafana-data:/var/lib/grafana
    depends_on: 
      - influxdb

  jmeter:
    build:
      context: ./jmeter
    image: 'jmeter-docker:latest'
    container_name: jmeter
    restart: always
    networks:
      - monitoring
    volumes:
      - ./jmeter:/results
    depends_on: 
      - influxdb
      - grafana

networks:
  monitoring:
volumes:
  influxdb-storage:
  grafana-data:
  grafana-volume:
</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p><strong>Steps to be followed</strong> <strong>to create Docker Compose Yaml file</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>1. Create customized docker files for jmeter &amp; grafana and place in each respective folders and for influxdb can be directly pull it from dockerhub.   </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>2. Create a Network to connect multiple containers to establish connection between them and default network is bridge and just define it in the yaml no need to explicitly create it.  </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>3. Create volumes such as named volumes to persist the data whenever there are any unseen container exists and we can use volumes to get the data back just define it in Yaml docker will create for you</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>4. Create environment variables to pass it to the runnning containers such as <strong>ex:</strong> <strong>creating TSDB with name Jmeter in influxdb</strong> <strong>before running jmeter executions</strong>. I<strong>NFLUXDB_DB:jmeter</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>5. Use docker-build to build the docker image from the specific build context folder where dockerfile resides to get the latest changes into docker image.   </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>6. Finally just run <strong>docker-compose up -d</strong> to run all the services defined in docker-compose.yaml file in detached mode.</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"yaml"} -->
<pre class="wp-block-syntaxhighlighter-code">Yaml files grafana and automatic creation of Datasource and Dashboard

FROM grafana/grafana:8.1.0

ENV GF_SECURITY_ADMIN_USER=admin
ENV GF_SECURITY_ADMIN_PASSWORD=grafana
ENV GF_INSTALL_PLUGINS=grafana-clock-panel,briangann-gauge-panel,natel-plotly-panel,grafana-simple-json-datasource

# automatically provisions datasources and dashboards in grafana
ADD ./provisioning /etc/grafana/provisioning

# Add configuration file
#ADD ./grafana.ini /etc/grafana/grafana.ini

</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:syntaxhighlighter/code -->
<pre class="wp-block-syntaxhighlighter-code">Grafana Datasource Yaml format
apiVersion: 1
datasources:
  - name: 'Influx-Jmeter'
    type: influxdb
    access: proxy
    database: jmeter
    url: http://influxdb:8086
    isDefault: true
    editable: true
</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:group -->
<div class="wp-block-group"><!-- wp:paragraph -->
<p>Grafana Dashboard format download <a rel="noreferrer noopener" href="https://github.com/manojkumar542/Monitoringtoolkit/blob/main/Docker-compose/grafana/provisioning/dashboards/Apache%20Jmeter%20Dashboard.json" target="_blank">here</a></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now go to docker-compose file folder and  run <strong>docker-compose up -d</strong></p>
<!-- /wp:paragraph --></div>
<!-- /wp:group -->

<!-- wp:paragraph -->
<p>Use <strong>docker exec -it containerid /bin/sh</strong> to connect to containers and check the influxdb to check the database with name jmeter is created or not same way you can do it for grafana to check the dashboards and datasources are referring correctly from the copied files.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Since i have enabled port mapping to expose container port to localhost port i can able to access both grafana and influxdb using httP://localhost:portnumber</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But in containers they use internal ips to communicate and internally container name DNS is mapped to ip that's the reason i have used container names to configure datasource connection in influxdb url in grafana</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"yaml"} -->
<pre class="wp-block-syntaxhighlighter-code">Using docker-compose build  by providing build context we can build the image from the specific docker file folder using docker-compose yaml itself as below
  jmeter:
    build:
      context: ./jmeter
    image: 'jmeter-docker:latest'
    container_name: jmeter
    restart: always
    networks:
      - monitoring
    volumes:
      - ./jmeter:/results
    depends_on: 
      - influxdb
      - grafana

Context - where exactly the dockerfile resides and build and tag with the image name specified in our case it's &lt;strong>'jmeter-docker:latest'&lt;/strong>

depends_on - make sure services in the section specified will be starting and running which are dependent on the current service.</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:image {"id":2049,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfdevops.files.wordpress.com/2021/09/image-7.png?w=1024" alt="" class="wp-image-2049"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2037,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfdevops.files.wordpress.com/2021/09/image.png?w=1024" alt="" class="wp-image-2037"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2042,"width":654,"height":296,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large is-resized"><img src="https://perfdevops.files.wordpress.com/2021/09/image-2.png?w=1024" alt="" class="wp-image-2042" width="654" height="296"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2053,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfdevops.files.wordpress.com/2021/09/image-9.png?w=1024" alt="" class="wp-image-2053"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2044,"width":654,"height":443,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large is-resized"><img src="https://perfdevops.files.wordpress.com/2021/09/image-4.png?w=1024" alt="" class="wp-image-2044" width="654" height="443"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2055,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfevops.files.wordpress.com/2021/09/image-10.png?w=1024" alt="" class="wp-image-2055"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2056,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfdevops.files.wordpress.com/2021/09/jpetstore_grafana.jpg?w=1024" alt="" class="wp-image-2056"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2045,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://perfdevops.files.wordpress.com/2021/09/image-5.png?w=654" alt="" class="wp-image-2045"/></figure>
<!-- /wp:image -->

For more details follow my blogs
* https://perfdevops.wordpress.com/2021/01/22/easy-setup-to-dockerize-jmeter-monitoring-solution/
* https://perfdevops.wordpress.com/2020/09/11/run-jmeter-in-docker-and-save-results-in-csv/
* https://perfdevops.wordpress.com/2020/02/01/setup-jmeter-in-linux-docker-container-with-all-plugins/
* [monitoring setup](https://perfdevops.wordpress.com/2021/01/22/easy-setup-to-dockerize-jmeter-monitoring-solution/) 
