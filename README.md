# Shell Scripts as Serverless functions 

Goal : Run old shell scripts as serverless functions. 

OpenFaaS is very straight forward to setup.  This setup uses a local Docker Swarm. Instructions can be found [here](https://docs.openfaas.com/deployment/docker-swarm/).

Follow the instructions [here](https://blog.alexellis.io/cli-functions-with-openfaas). When you reach the "2.1 nmap" section then you can try the following.

## Setup
 
```
faas new --lang dockerfile sh
```

Copy [test.sh](./sh/test.sh) into the "sh" directory.

Edit sh.yml to look like : 
```
provider:
  name: faas
  gateway: http://127.0.0.1:8080
functions:
  sh:
    lang: dockerfile
    handler: ./sh
    image: sh:latest
```
Update Dockerfile to include :
```
RUN apk add --no-cache whois iputils 
 
ADD test.sh /tmp/test.sh

ENV fprocess="xargs sh /tmp/test.sh"

```

## Build and Deploy

Example is for .sh scripts. Works for bash too.
```
faas build -f sh.yml  && faas deploy -f sh.yml
```

## Test it works

echo -n "google.com" | faas invoke sh

echo -n "bbc.com" | faas invoke sh

echo -n "amazon.com" | faas invoke sh

Example Output :
```
=========================================
Running WHOIS for : google.com
=========================================
Name Server: ns4.google.com
Name Server: ns2.google.com
Name Server: ns3.google.com
Name Server: ns1.google.com
=========================================
Ping Check
=========================================
PING google.com (74.125.193.100) 56(84) bytes of data.
64 bytes from ig-in-f100.1e100.net (74.125.193.100): icmp_seq=1 ttl=46 time=14.0 ms

--- google.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 14.004/14.004/14.004/0.000 ms
=========================================

```

## Add more replicas to handle incoming requests

```
docker service scale sh=3

sh scaled to 3
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 

Verify the hostname changes when running this a number of times :

echo -n "amazon.com" | faas invoke sh

```
