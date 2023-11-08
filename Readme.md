# Using Nginx as a TCP ingress for openshift

nginx has an option to allow TCP connnections and proxy them to the backend systems using Streams. The configmap "nginx" created from the file default.conf defines all the TCP connections.
nginx is run on specific nodes and the container ports use hostports to bind to the node on which nginx containers are running.

A load balancer is created with these nginx specic nodes as backend servers and the listeners will be pointing to the ports of nginx based on the proxy backends.


```
oc new-project nginx
oc create sa nginx
oc adm policy add-scc-to-user privileged -z nginx
oc create configmap nginx --from-file=default.conf=./default.conf
oc apply -f nginx-deploy.yaml
```

```
stream {
  server {
      listen       80;  # nginx listens on port 80
      listen  [::]:80;
      proxy_pass skupper-redis-server-1:6379;  # traffic on port 80 is redirected to redis server 1 on port 6379
  }
    server {
      listen       8080;  # nginx listens on port 80
      listen  [::]:8080;
      proxy_pass skupper-redis-server-2:6379; # traffic on port 8080 is redirected to redis server 2 on port 6379
  }
}
```

```
        ports:
        - containerPort: 80   # port 80 on container based on nginx conf above
          hostPort: 80   # container port 80 is mapped to 80 on host  
          protocol: TCP
        - containerPort: 8080  # port 8080 on container based on nginx conf above
          hostPort: 8111 # container port 8080 is mapped to 8111 on host
          protocol: TCP
```