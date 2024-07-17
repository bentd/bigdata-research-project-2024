up:
	docker-compose up -d

down:
	docker-compose down

start-sc:
	docker exec -it spark-master bash /opt/bitnami/spark/sbin/start-connect-server.sh --packages=org.apache.spark:spark-connect_2.12:3.5.1

stop-sc:
	docker exec -it spark-master bash /opt/bitnami/spark/sbin/stop-connect-server.sh

install-pyspark-packages:
	docker exec -it spark-worker-1 pip install py4j pandas wheel
	docker exec -it spark-worker-2 pip install py4j pandas wheel
	docker exec -it spark-worker-3 pip install py4j pandas wheel	

start-jupyter:
	/.local/lib/python3.11/site-packages/jupyterlab$ __main__.py

clean-staging:
	docker exec -itu 0 namenode rm -rf /home/staging/

clean-hdfs:
	docker exec -it namenode hadoop fs -rm -r -skipTrash /data/ || true
	docker exec -it namenode hadoop fs -rm -r -skipTrash /results/ || true

load-data:
	docker exec -itu 0 namenode mkdir -p /home/staging/
	docker cp ../data/  namenode:/home/staging/	
	docker exec -it namenode hadoop fs -mkdir /data/ || true
	docker exec -it namenode hadoop fs -put /home/staging/data/ / || true

save-results:
	docker exec -itu 0 namenode mkdir -p /home/staging/results/
	docker exec -itu 0 namenode hadoop fs -get /results/ /home/staging/ || true
	docker cp namenode:/home/staging/results/ ../


