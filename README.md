# Nginx mirror module proof of concept

This repo is doing a proof of concept of the mirror module of nginx.
To do this POC, I'm using a docker-compose to orchestrate all the services and run in an easy way.

## How to run

Every test case has their own branch, do a checkout of the branch related with it test case.
Start docker in your computer and then``` docker-compose up``` to start the case.

In the expectations section of every test case, you can find the curl requests you can test.
To verify how this test case works, we can see the response of the proxy, and the logs of the mock services.

## How to tune up

Each docker container has their own main configuration externalized in a folder of this repo, those config files are mounted in the containers, so you only need to modify this configuration before run the docker-compose.

## Test cases
### Scenario 1: One proxy and one mirror
- git branch: master
- Mock services: Two mock services are simulating same api (api-a) with some different behaviours.
  - mock-app-a-1
  - mock-app-a-2
- Nginx: one nginx is exposed through http 80 port. With the following configuration
    - Proxy: mock-app-a-1
    - Mirror: mock-app-a-2
  
### Expectations

#### Test case: valid request in proxy and mirror
- request: `curl "localhost:80/api-a/valid"`
- response: code=200, body='app-a-1 valid response'
- nginx log: 200 http code response
- mock-app-a-1 log: request received, matched and 200 http code response
- mock-app-a-2 log: request received, matched and 200 http code response

#### Test case: valid request in proxy and error in mirror
- request: `curl "localhost:80/api-a/error-only-in-mirror"`
- response: code=200, body='app-a-1 valid response, and invalid in app-a-2'
- nginx log: 200 http code response
- mock-app-a-1 log: request received, matched and 200 http code response
- mock-app-a-2 log: request received, matched and 500 http code response

#### Test case: error request in proxy and valid in mirror
- request: `curl "localhost:80/api-a/error-only-in-proxy"`
- response: code=500, body='app-a-1 invalid response, and valid in app-a-2'
- nginx log: 500 http code response
- mock-app-a-1 log: request received, matched and 500 http code response
- mock-app-a-2 log: request received, matched and 200 http code response

#### Test case: error request in proxy and mirror
- request: `curl "localhost:80/api-a/error"`
- response: code=500, body='app-a-1 error response'
- nginx log: 500 http code response
- mock-app-a-1 log: request received, matched and 500 http code response
- mock-app-a-2 log: request received, matched and 500 http code response

#### Test case: valid POST request in proxy and mirror
- request: `curl -X POST "localhost:80/api-a/valid" -d '{"name": "Alberto"}'`
- response: code=201, body='app-a-1 valid response'
- nginx log: 201 http code response
- mock-app-a-1 log: request received, matched and 201 http code response
- mock-app-a-2 log: request received, matched and 201 http code response

#### Test case: valid POST request in proxy and error in mirror
- request: `curl -X POST "localhost:80/api-a/error-only-in-mirror" -d '{"name": "Alberto"}'`
- response: code=201, body='app-a-1 valid post response, and invalid in app-a-2'
- nginx log: 201 http code response
- mock-app-a-1 log: request received, matched and 201 http code response
- mock-app-a-2 log: request received, matched and 500 http code response

#### Test case: error POST request in proxy and valid in mirror
- request: `curl -X POST "localhost:80/api-a/error-only-in-proxy" -d '{"name": "Alberto"}'`
- response: code=500, body='app-a-1 invalid post response, and valid in app-a-2'
- nginx log: 500 http code response
- mock-app-a-1 log: request received, matched and 500 http code response
- mock-app-a-2 log: request received, matched and 201 http code response

#### Test case: error POST request in proxy and mirror
- request: `curl -X POST "localhost:80/api-a/error" -d '{"name": "Alberto"}'`
- response: code=500, body='app-a-1 error post response'
- nginx log: 500 http code response
- mock-app-a-1 log: request received, matched and 500 http code response
- mock-app-a-2 log: request received, matched and 500 http code response


## Official documentation
### Nginx mirror module
http://nginx.org/en/docs/http/ngx_http_mirror_module.html

Nginx project tests that are supporting this mirror module:
https://github.com/nginx/nginx-tests/blob/master/mirror.t
https://github.com/nginx/nginx-tests/blob/master/mirror_proxy.t

### MockServer
https://www.mock-server.com/