Create a filter that discards received data if they contain ```"localhost"``` in ```"source"``` field.

1. Pull image:
```
docker pull fluent/fluentd:v1.4-2
```

2. Create a Dockerfile with the following content to be able to run processes inside a container as a root:
```
FROM fluent/fluentd:v1.4-2
USER root:root
```

3. Create an image and tag it as ```"myfluentd"```:
```
sudo docker build -t myfluentd .
```

4. Create ```fluentd.conf``` file with the following content:
```
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>

<match test1>
  @type file
  path /var/log/fluent/access
</match>

<filter test1>
  @type grep
  <exclude>
    key source
    pattern localhost
  </exclude>
</filter>
```

5. Create and run a container:
```
sudo docker run -p 8888:8888 -v $(pwd):/fluentd/etc myfluentd -c /fluentd/etc/fluentd.conf
```

6. Access an endpoint being logged:
```
curl -X POST -d 'json={"message":"It works!"}' http://localhost:8888/test1
curl -X POST -d 'json={"source": "localhost", "message": "test"}' http://localhost:8888/test1
curl -X POST -d 'json={"source": "localhost2", "message": "test"}' http://localhost:8888/test1
curl -X POST -d 'json={"source": "localhost3", "message": "test"}' http://localhost:8888/test1
```

7. Log in to a container as a root:
```
sudo docker exec -u root -i -t container_id sh
```

8. Make sure a new log file was created:
```
ls /var/log/fluent/access
```
