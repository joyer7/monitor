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


## B. Grafana?

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
	
	
## C. Use It!
    - windows key + R
    - type "CMD"  and run CMD Window
    - type "git clone https://github.com/joyer7/mornitor.git" on command window
    - Enjoy It !!!
    

## D. Question & Answer
Please feel free to contact Austin Rho 
    - ssrho@kolon.com, injoyer@yonsei.ac.kr
	
	