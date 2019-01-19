# undefined-API-gateway

![feature](https://user-images.githubusercontent.com/3271895/51427833-6d132780-1c3f-11e9-8f73-7112a7f0da0c.png)


## Introduction
Asynchronous API Gateway with Spring boot 2.1, Servlet 4, jetty 9 client <br />
The Gateway is a network gateway created to provide a single access point for real-time web based protocol elevation that supports load balancing, clustering, and lots of validations. It is designed to make the best performance to deliver open API.
* Lightweight API Gateway
* High Performance/Scalability

## New Feature(1-15-2019)
* Management API
* Supported Server Cluster

## Features
Undefined API Gateway offers powerful, yet lightweight feature.

* **Request Validation** - 요청에 다양한 검증기능을 사용할 수 있습니다. 또한 새로운 feature 를 추가하거나 제거하기 쉽습니다.
    - Header, Query, Path Param
* **Rate Limiting** - API Users 별 강력한 Rate limiting 을 제공합니다. Redis 베이스로 Cluster 서버들은 Ratelimiting 정보를 공유합니다. on a per-key basis
    - App Daily Rate Limiting
    - App Minutely  Rate Limiting
* **Service Capacity** - 연동된 서비스의 Capacity 를 관리하여 안정적인 운영이 가능하도록 합니다.
    - Service Daily Capacity
    - Service Minutely Capacity
* **Service Contract(agreement)** - (Optional) API, APP Users 들은 계약관계 혹은 약관에 동의한 API 만 호출 가능합니다.
* **Request Transform** - (Optional) URI 변경 뿐만 아니라 Header, Query, Path Param 의 변경을 지원합니다. 이는 User 의 요청을 undefiend api Gateway 와 연동된 서비스의 요청으로 적절하게 변경해줍니다.
* **IP Whitelisting** - Block access to non-trusted IP addresses for more secure interactions on a per-key basis
* **Management API** - API Gateway 를 관리하는 강력한 Rest API 를 제공합니다.
    - API 추가/삭제/변경
    - APP 추가/삭제/변경 
    - IP Whitelist 추가/삭제
    - Key Expiry/Regenerate
* **Supported Server Cluster** - API Gateway Cluster 를 구성할 수 있습니다. Management API 를 사용하면 모든 서버에서 변경사항이 적용됩니다. Ratelimiting, Service Capacity 정보도 모두 공유됩니다. 

## Dependency
* Spring Boot 2.1
* Servlet 4
* Ehcache 3
* Jetty 9 client
* Jedis 3.0

## Configuration
There are required settings to run undefined-api-gateway.
You do not need to use initialization with the management API.

A. First
 <br />
    - Please set the initial application registration in application-apps.yaml <br />
    - (These settings are optional)

    init-apps:
      init-enable: true
      apps:
        -
          app-id: 0
          app-name: TestApp
          app-key: 1000-1000-1000-1000
          app-minutely-ratelimit: 2000
          app-daily-ratelimit: 10000
          app-service-contract: ['01', '02', '03']
          app-ip-acl: ['192.168.0.1', '127.0.0.1']
        -
          app-id: 1
          app-name: BestApp
          app-key: e3938427-1e27-3a37-a854-0ac5a40d84a8
          app-minutely-ratelimit: 1000
          app-daily-ratelimit: 50000
          app-service-contract: ['01', '02']
          app-ip-acl: ['127.0.0.1']

- init-enable: The initial registration setting is not used.
- app-service-contract: A list of API services that app has permission to use.
- app-ip-acl: The whitelist of ip that can use this App key(=API Key).
- app minutely/daily ratelimit: The amount of APIs available to the app. 

B. Second
 <br />
    - Set up service and API specification configurations in application-apis.yml <br />
    - The API Gateway obtains Service and API information through the APIExposeSpecLoader.

    api-spec:
      init-enable: true
      services:
        -
          service-id: '01'
          service-name: stackoverflow
          service-minutely-capacity: 10000
          service-daily-capacity: 240000
          service-path: /stackoverflow
          outbound-service-host: api.stackexchange.com
          apis:
            -
              api-id: '0101'
              api-name: getInfo
              protocol: http, https
              method: get
              inbound-url: /2.2/question/:first
              outbound-url: /2.2/questions
              header: page, votes
              header-required: ""
              url-param: version, site
              url-param-required: site
            -
              api-id: '0202'
              api-name: getQuestions
              protocol: https
              method: put
              inbound-url: /2.2/question/:first
              outbound-url: /2.2/questions
              header: page, votes
              header-required: ""
              url-param: version, site
              url-param-required: site
        -
          service-id: '02'
          service-name: stackoverflow2
          service-minutely-capacity: 5000
          service-daily-capacity: 100000
          service-path: /another
          outbound-service-host: api.stackexchange.com
          apis:
            -
              api-id: '0201'
              api-name: transformTest
              protocol: http, https
              method: get
              inbound-url: /2.2/haha/question/:site
              outbound-url: /:page/:site
              header: page, votes
              header-required: ""
              url-param: version, site
              url-param-required: site
              transform:
                page: [header, param_path]
                site: [param_path, header]
        -
          service-id: '03'
          service-name: service3
          service-minutely-capacity: 5000
          service-daily-capacity: 100000
          service-path: /service3
          outbound-service-host: api.stackexchange.com
          only-pass-request-without-transform: true

- init-enable: The initial registration setting is not used.
- service-path: URL The first parameter in the Path. The API is routed to the service registered in that parameter.
- service minutely/daily capacity: The total amount of APIs that the service can route.
- outbound-service-host: The APIs of the service are routed to that domain.
- apis/inbound-url: Externally exposed API URL Path specification. ':?' Is a variable.
- apis/outbound-url: The actual url path of the service connected to the api-gateway.
- apis/header: This is the header that can be received when requesting API.
- apis/header-required: This header is mandatory for API requests.
- apis/url-param: This is the url path parameter that can be received when requesting API.
- apis/url-param-required: This url path parameter is mandatory for API requests.
- transform: The param that is received at the time of request is transformed into another variable area at the time of routing.
    - Possible options: **header**, **param_path**, **param_query**
    - usage: [source, destination] like [header, param_path]
- only-pass-request-without-transform: All APIs are routed to services connected to the gateway without any analysis or transformation. 


Undefined-api-gateway supports the following protocol and method.

* Protocol
    - http, https
* Method
    - Get, Post, Put, Delete
    
## API Gateway Cluster
Undefined API Gateway supports clusters. Each node synchronizes API, APP, IP Whitelist and App Key (= API Key) information in near real time.
Cluster nodes also work together to calculate the correct API Ratelimiting, Service Capacity.

- **Service, API, APP, IP Whitelist Interval Sync**
![feature](https://user-images.githubusercontent.com/3271895/51427837-7b614380-1c3f-11e9-82f3-5a668ba63f00.png)


## Management REST API
The Management API helps manage a single gateway or cluster group.

**APP Management**
  - **APP Register** - [POST] /internal/apps
  - **APP UnRegister** - [DELETE] /internal/apps/{appId}
  - **APP Update** - [Future Feature]
  - **Appkey(=API Key) Expiry** - [DELETE] /internal/apps/{appId}/appkey
  - **Appkey(=API Key) Regenerate** - [PUT] /internal/apps/{appId}
  - **Add NEW IP Whitelist** - [POST] /internal/apps/{appId}/whitelist
  - **Remove IP Whitelist** - [DELETE] /internal/apps/{appId}/whitelist
  
**API Management**
  - **New API Register** - [POST] /internal/apis
  - **API UnRegister** - [DELETE] /internal/apis/{apiId}
  - **API Update** - [PUT] /internal/apis/{apiId}

**Service Group API Will be updated**

    

## Usage/Test

##### Test Case - stackoverflow API.

Run undefined-api-gateway

    ./gradlew test

Use rest-client like Postman. To set method and scheme.

    GET, http 

Input URL.

    http://localhost:8080/stackoverflow/2.2/question/test

Input URL parameter. ( site is mandatory query parameter )

    site = stackoverflow

OR you can input URL like below.

    http://localhost:8080/stackoverflow/2.2/question/test?site=stackoverflow

and then input header fields. ( appkey is mandatory header.(or Query Parameter) )

    appkey, 1000-1000-1000-1000
    page, 5
    votes, 1

Execute request and check response code and content.

## Future update
* Authentication for Private API
* Docker-Compose
    - Easy To Run

## Contact
For any inquiries, you can reach me at longcoding@gmail.com 

## License
undefined-api-gateway is released under the MIT license. See LICENSE for details.
