# Monitor
Prometheus & Grafana
- Kolon Benit, BigData Analytics Team, Austin Rho, (Shin Seok)
- last update: 2023.03.02

## 0. Software
- Prometheus : Etl
- Grafana : Visualization
- Ubuntu Linux : AWS EC2 t2.micro (free-tier)
- Docker : Execute Prometheus & Grafana


## A. Prometheus

**1. Install **

	[Docker]
    - step01 : sudo apt update
	- step02 : sudo apt install -y docker.io
	
	[Prometheus Install]
    - step03 : sudo mkdir -p /prometheus/config /prometheus/data
    - step04 : vim /prometheus/config/prometheus.yml
	    ==========================
		scrape_configs:
		  - job_name: 'prometheus'
		    scrape_interval: 5s
		    scrape_timeout: 1s
		    static_configs:
		    - targets:
		      - localhost:9090
	    ==========================
	
	[Prometheus start / Stop / Reuse]
	- step05 :
	    sudo docker run --rm prom/prometheus:v2.29.2  (kill prometheus when exits) 
		sudo docker ps -a
		sudo docker stop prometheus
		sudo docker rm -f prometheus
	
	[Prometheus running command]
	- step06 : 
		sudo docker run -d --net=host --name=prometheus 
		-v /prometheus/config:/etc/prometheus
		-v /prometheus/data:/data prom/prometheus:v2.29.2
		--config.file=/etc/prometheus/prometheus.yml
		--storage.tsdb.path=/data
		--web.enable-lifecycle
		--log.level=debug
		
		* run check
		buntu@ip-172-31-11-11:~$ ss -ntlp
		

	[Logs]
	- step07 : logs
		sudo docker logs -f prometheus
		sudo docker logs -f prometheus 2>&1 | grep permission
		sudo docker logs -f prometheus 2>&1 | grep retention
		sudo docker logs -f prometheus 2>&1 | grep debug
	
	[When Error]	
	- step08 : when permission error
		* check docker ps 
			sudo docker ps -a
			CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                      PORTS     NAMES
			62a543136034   prom/prometheus:v2.29.2   "/bin/prometheus --c…"   21 minutes ago   Exited (2) 20 minutes ago             prometheus
		* run into prometheus
			sudo docker exec -it prometheus sh
		* check id (in docker process)
			/prometheus $ id
			uid=65534(nobody) gid=65534(nobody)
	- step09 : 
		* sudo chown -R 65534:65534 /prometheus
		
	[Test Web]
	- step10 : Test Web
		* curl localhost:9090/-/healthy -D /dev/stdout
		* curl localhost:9090/-/ready -D /dev/stdout
		* ubuntu@ip-172-31-11-11:~$ curl localhost:9090/-/reload -XPOST -D /dev/stdout
			HTTP/1.1 200 OK
			Date: Fri, 03 Mar 2023 05:25:03 GMT
			Content-Length: 0	


**2. Node Exporter **

	[Node Exporter]
	- https://prometheus.io/docs/instrumenting/exporters/
	- https://prometheus.io/docs/guides/node-exporter/
	- https://github.com/prometheus/node_exporter
	

	[Install by docker]
	docker run -d \
	  --name=node_exporter
	  --net="host" \
	  --pid="host" \
	  -v "/:/host:ro,rslave" \      <= forbidden changing host root volume, check host root volumn changes
	  quay.io/prometheus/node-exporter:latest \
	  --path.rootfs=/host
	
	[Install by systemctl]
	- sudo docker rm node_exporter
	- https://prometheus.io/download/#node_exporter : Check url & Copy
	wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
	tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz
	cd node_exporter-1.5.0.linux-amd64.tar.gz
	./node_exporter
	
	*on /opt
	- tar -C /opt -xvzf node_exporter-1.5.0.linux-amd64.tar.gz
	- ln -s /opt/node_exporter-1.5.0.linux-amd64 /opt/node_exporter
	- echo "OPTION=" > /etc/default/node_exporter
	
	*node_exporter.service
	cat << EOF < /etc/systemd/system/node_exporter.service
	> [Service]
	> User=root
	> EnvironmentFile=/etc/default/node_exporter
	> ExecSTart=/opt/node_exporter/node_exporter
	> EOF
	
	*systemctl daemon-reload
	
	*systemctl start node_exporter.service
	
	
**3. Discovery **

	[Description]
	get target server info
	http://43.201.xx.xx:9090/service-discovery
	
	[Install]
	- cd /prometheus/config
	- mv prometheus.yml static_sd.yml
	- vim /prometheus/config/file_sd.yml
	    ==========================
		scrape_configs:
	    - job_name: 'prometheus'
	      follow_redirects: true
	      scrape_interval: 5s
	      scrape_timeout: 1s
		
	      file_sd_configs:
	      - files:
	        - sd/*.yml
	    ==========================
		
	- ln -sf file_sd.yml prometheus.yml
	- vim sd/localhost.yml
	    ==========================
	    -targets:
	       - localhost:9100
	       labels:
	          region: KR
	          tier: frontend
	          environment: development
	          disk: NVMe
	    ==========================
		
	- curl localhost:9090/-/reload -XPOST -D /dev/stdout
	- http://43.201.xx.xx:9090/service-discovery


	[Label Change]
	- vim /prometheus/config/file_sd.yml
	    ==========================
		scrape_configs:
	    - job_name: 'prometheus'
	      follow_redirects: true
	      scrape_interval: 5s
	      scrape_timeout: 1s
		
	      file_sd_configs:
	      - files:
	        - sd/*.yml
	      relabel_configs:
	      - cource_lavels: [__adress__']
	        regex: '(.*):(.*)'
	        replacement: '${1}'
	        target_label: 'instance'
	      - cource_lavels: [__adress__']
	        regex: '(.*):(.*)'
	        replacement: '${2}'
	        target_label: 'port'
	    ==========================
	- curl localhost:9090/-/reload -XPOST -D /dev/stdout
	- http://43.201.xx.xx:9090/service-discovery
	
**4. Metric **
	
	[Install]
	- sudo apt install -y python3-pip
	- python3 -m pip install prometheus_client
	- mkdir agent_python && cd agent_python
	- vim agent.py
	    ==========================
	    import time
		import http.server
		import prometheus_client import Histogram, start_http_server
		
		histogram = Histogram(
		'response_time_histogram',
		'Response time for a request',
		buckets=[0.0003, 0.00035, 0.0004, 0.0005])
	
		class Driver(http.server.BaseHTTPRequestHandler):
		   def do_GET(self):
		   start = time.time()
		   self.send_response(200)
		   self.wfile.write(b"Histogram Test")
		   histogram.observe(time.time() - start)
		
		if _name__ == "__main__":
		   start_http_server(8081)
		   server = http.server.HTTPServer(('localhost',8000), Driver)
		   print('Exporter running on 8081')
		   print('Server running on 8080')
		   server.servr_forever()
	- python3 agent.py
	

## B. Grafana?

**1. Install **




## C. Use It!
    - windows key + R
    - type "CMD"  and run CMD Window
    - type "git clone https://github.com/joyer7/mornitor.git" on command window
    - Enjoy It !!!
    

## D. Question & Answer
Please feel free to contact Austin Rho 
    - ssrho@kolon.com, injoyer@yonsei.ac.kr
	
	