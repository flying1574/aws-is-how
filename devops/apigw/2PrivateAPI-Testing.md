# 2 Private API invocation testing

## Architecture
1. In account 1: Fargate cluster expose web-app service by target group web-app-fargate-nlb-tg. The web-app-fargate-nlb-tg attached to NLB web-app-fargate-nlb-internal
2. In account 1: VPC link web-app-fargate-vpclink (t0f7wl) point to NLB web-app-fargate-nlb-internal
3. In account 1: Private API Fargate-webpage-private (3i95y1yx06) /webpage-vpc (8hwj4v) path GET method point to VPC link integration web-app-fargate-vpclink (t0f7wl) 
4. In account 1: VPC endpoint vpce-0d4d61b31cecd49fc used for private API Fargate-webpage-private (3i95y1yx06) 
5. In account 1: NLB web-app-fargate-nlb-internal2 route traffic to private API Fargate-webpage-private (3i95y1yx06) 
6. In account 1: VPC link web-app-fargate-vpclink-2ndnlb (lsm4cw) point to NLB web-app-fargate-nlb-internal2
7. In account 1: Private API Fargate-private-to-first-api (v944po9kjb) /webpage-vpc path GET method point to VPC link integration web-app-fargate-vpclink-2ndnlb (lsm4cw) 
8. In account 1: VPC endpoint vpce-080de204ca78d2883 in account 1 consumer VPC for private API Fargate-private-to-first-api (v944po9kjb)
9. In account 2: VPC endpoint vpce-08cbeb493b8abab4f in account 2 consumer VPC for private API Fargate-private-to-first-api (v944po9kjb) 

![2PrivateAPI](media/2PrivateAPI.png)

![Fargate-private-to-first-api-vpce](media/Fargate-private-to-first-api-vpce.png)

## On Provider VPC of account1 or Consumer VPC of account1 NLB web-app-fargate-nlb-internal2 with -H 'x-apigw-api-id:3i95y1yx06' - Result: Success

Note: if you invoke without -k option of curl, it will report:

```bash
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  subjectAltName does not match web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn
* SSL: no alternative certificate subject name matches target host name 'web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn'
```

```bash
curl -v -k https://web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc-non-proxy -H 'x-apigw-api-id:3i95y1yx06'
   Trying 10.0.1.95...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn (10.0.1.95) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc-non-proxy HTTP/1.1
> Host: web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> x-apigw-api-id:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 01:45:59 GMT
< Content-Type: application/json
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 11efdbb6-4c3c-4dfb-96a8-f1ce7943f847
< x-amz-apigw-id: OjvBvF2VZPgFQFw=
< X-Amzn-Trace-Id: Root=1-5ef15ed7-426a634a4a810394fc9b15c2
<
* Connection #0 to host web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>
```

## On Provider VPC of account1 or Consumer VPC of account1 NLB web-app-fargate-nlb-internal2 with -H 'Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com' - Result: 403 Forbidden

The 403 caused by missing x-apigw-api-id when NLB invoke the VPCE of API GW.

```bash
curl -k -v https://web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc-non-proxy -H 'Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com'
   Trying 10.0.3.22...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn (10.0.3.22) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc-non-proxy HTTP/1.1
> Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 403 Forbidden
< Server: Server
< Date: Tue, 23 Jun 2020 01:45:30 GMT
< Content-Type: application/json
< Content-Length: 23
< Connection: keep-alive
< x-amzn-RequestId: 5bb402aa-193f-4fd5-a6f7-1d9c07e5bf02
< x-amzn-ErrorType: ForbiddenException
< x-amz-apigw-id: Oju9IGv65PgFlBg=
<
* Connection #0 to host web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn left intact
{"message":"Forbidden"}
```

## On Provider VPC of account1 or Consumer VPC of account1 NLB web-app-fargate-nlb-internal2 with -H 'x-apigw-api-id:3i95y1yx06' and -H 'Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com' - Result: Success

```bash
curl -v https://web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc-non-proxy -H 'x-apigw-api-id:3i95y1yx06' -H 'Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com'
*   Trying 10.0.9.229...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn (10.0.9.229) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc-non-proxy HTTP/1.1
> Host: 3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com
> User-Agent: curl/7.61.1
> Accept: */*
> x-apigw-api-id:3i95y1yx06
> 
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 02:14:10 GMT
< Content-Type: application/json
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: e1f335f4-e11c-4b72-acca-e1262e84dd37
< x-amz-apigw-id: OjzJ7FytZPgF4Jg=
< X-Amzn-Trace-Id: Root=1-5ef16572-8980fbea95d6443c3c112669
< 
* Connection #0 to host web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>
```


## On Provider VPC of account1 or Consumer VPC of account1 invoke the Private API Fargate-private-to-first-api

curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc
*   Trying 172.16.33.221...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.16.33.221) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host: v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
>
< HTTP/1.1 500 Internal Server Error
< Server: Server
< Date: Tue, 23 Jun 2020 02:01:02 GMT
< Content-Type: application/json
< Content-Length: 36
< Connection: keep-alive
< x-amzn-RequestId: 5656bea2-a2eb-4943-91f9-0073f9a37143
< x-amzn-ErrorType: InternalServerErrorException
< x-amz-apigw-id: OjxO1FJGZPgFlBg=
<
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
{"message": "Internal server error"}

Why 500 Internal server error? Let's use the API GW console Test client to find more reason

## Directly invoke from API GW console Test client
```bash
Execution log for request 2cd9f026-8ece-42b5-a448-fcbf917ba63e
Tue Jun 23 02:21:45 UTC 2020 : Starting execution for request: 2cd9f026-8ece-42b5-a448-fcbf917ba63e
Tue Jun 23 02:21:45 UTC 2020 : HTTP Method: GET, Resource Path: /webpage-vpc
Tue Jun 23 02:21:45 UTC 2020 : Method request path: {}
Tue Jun 23 02:21:45 UTC 2020 : Method request query string: {}
Tue Jun 23 02:21:45 UTC 2020 : Method request headers: {}
Tue Jun 23 02:21:45 UTC 2020 : Method request body before transformations: 
Tue Jun 23 02:21:45 UTC 2020 : Endpoint request URI: https://web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc
Tue Jun 23 02:21:45 UTC 2020 : Endpoint request headers: {x-amzn-apigateway-api-id=v944po9kjb, User-Agent=AmazonAPIGateway_v944po9kjb, Host=web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn}
Tue Jun 23 02:21:45 UTC 2020 : Endpoint request body after transformations: 
Tue Jun 23 02:21:45 UTC 2020 : Sending request to https://web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc
Tue Jun 23 02:21:45 UTC 2020 : Execution failed due to configuration error: Host name 'web-app-fargate-nlb-internal2-fdeb5f1534e02ddb.elb.cn-northwest-1.amazonaws.com.cn' does not match the certificate subject provided by the peer (CN=*.execute-api.cn-northwest-1.amazonaws.com.cn)
Tue Jun 23 02:21:45 UTC 2020 : Method completed with status: 500
```

The `500 Internal server error` is caused by SSL check the request Host does not match the certificate subject
Could we pass through the Host of final private API Fargate-webpage-private?

Let's double check the API Fargate-private-to-first-api private integration setting, seems I do not set correct Endpoint URL

The uri parameter is not used for routing requests to your endpoint, but is used for setting the Host header and for certificate validation.

So the correct value should be the first private API Fargate-webpage-private (3i95y1yx06): https://3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn

![Fargate-private-to-first-api1](media/Fargate-private-to-first-api1.png)

![Fargate-private-to-first-api2](media/Fargate-private-to-first-api2.png)

## Define propagate customer header

Customer header mapping: method.request.header.key1

![Fargate-private-to-first-api-customerheader1](media/Fargate-private-to-first-api-customerheader1.png)

![Fargate-private-to-first-api-customerheader2](media/Fargate-private-to-first-api-customerheader2.png)

1. Directly invoke from API GW console Test client with 
key1:3i95y1yx06

```bash
Execution log for request 184996c0-dfb8-4095-9305-59b46a06f147
Tue Jun 23 05:56:11 UTC 2020 : Starting execution for request: 184996c0-dfb8-4095-9305-59b46a06f147
Tue Jun 23 05:56:11 UTC 2020 : HTTP Method: GET, Resource Path: /webpage-vpc
Tue Jun 23 05:56:11 UTC 2020 : Method request path: {}
Tue Jun 23 05:56:11 UTC 2020 : Method request query string: {}
Tue Jun 23 05:56:11 UTC 2020 : Method request headers: {key1=3i95y1yx06}
Tue Jun 23 05:56:11 UTC 2020 : Method request body before transformations: 
Tue Jun 23 05:56:11 UTC 2020 : Endpoint request URI: https://3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
Tue Jun 23 05:56:11 UTC 2020 : Endpoint request headers: {key1=3i95y1yx06, x-amzn-apigateway-api-id=v944po9kjb, User-Agent=AmazonAPIGateway_v944po9kjb, Host=3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn, x-apigw-api-id=3i95y1yx06}
Tue Jun 23 05:56:11 UTC 2020 : Endpoint request body after transformations: 
Tue Jun 23 05:56:11 UTC 2020 : Sending request to https://3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
Tue Jun 23 05:56:11 UTC 2020 : Received response. Status: 403, Integration latency: 19 ms
Tue Jun 23 05:56:11 UTC 2020 : Endpoint response headers: {Server=Server, Date=Tue, 23 Jun 2020 05:56:11 GMT, Content-Type=application/json, Content-Length=23, Connection=keep-alive, x-amzn-RequestId=da814d3f-8d91-4eae-94a9-f2b283ed0cf7, x-amzn-ErrorType=ForbiddenException, x-amz-apigw-id=OkTrUEOEZPgFQFw=}
Tue Jun 23 05:56:11 UTC 2020 : Endpoint response body before transformations: {"message":"Forbidden"}
Tue Jun 23 05:56:11 UTC 2020 : Method response body after transformations: {"message":"Forbidden"}
Tue Jun 23 05:56:11 UTC 2020 : Method response headers: {Server=Server, Date=Tue, 23 Jun 2020 05:56:11 GMT, Content-Type=application/json, Content-Length=23, Connection=keep-alive, x-amzn-RequestId=da814d3f-8d91-4eae-94a9-f2b283ed0cf7, x-amzn-ErrorType=ForbiddenException, x-amz-apigw-id=OkTrUEOEZPgFQFw=}
Tue Jun 23 05:56:11 UTC 2020 : Successfully completed execution
Tue Jun 23 05:56:11 UTC 2020 : Method completed with status: 403
```

2. We test on Provider VPC of account1 or Consumer VPC of account1 invoke the Private API Fargate-private-to-first-api

- Use Proxy Integration

```bash
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'key1:3i95y1yx06'

*   Trying 172.16.33.221...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.16.33.221) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host: v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 504 Gateway Timeout
< Server: Server
< Date: Tue, 23 Jun 2020 05:57:06 GMT
< Content-Type: application/json
< Content-Length: 56
< Connection: keep-alive
< x-amzn-RequestId: 32df3c57-da89-4d29-b584-627b85e273b9
< x-amzn-ErrorType: InternalServerErrorException
< x-amz-apigw-id: OkTz-Fm25PgFdlg=
<
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
{"message": "Network error communicating with endpoint"}
```

- Non Use Proxy Integration

```bash
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc-non-proxy -H 'key1:3i95y1yx06'
*   Trying 172.16.33.221...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.16.33.221) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc-non-proxy HTTP/1.1
> Host: v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 05:58:07 GMT
< Content-Type: application/json
< Content-Length: 24
< Connection: keep-alive
< x-amzn-RequestId: babdf8f5-5ba0-41c9-9973-07d39a8dd53f
< x-amz-apigw-id: OkT9cFi55PgFRpA=
< X-Amzn-Trace-Id: Root=1-5ef199ef-8d47ca618435e73b508686ed
<
{"message":"Forbidden"}
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
```

3. We test on Consumer VPC of account2 invoke the Private API Fargate-private-to-first-api

- Use Proxy Integration

```bash
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'key1:3i95y1yx06'

*   Trying 172.31.22.155...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.31.22.155) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host: v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 504 Gateway Timeout
< Server: Server
< Date: Tue, 23 Jun 2020 06:05:10 GMT
< Content-Type: application/json
< Content-Length: 56
< Connection: keep-alive
< x-amzn-RequestId: 870ff1d9-18a0-4212-9365-dcc9c90d77bf
< x-amzn-ErrorType: InternalServerErrorException
< x-amz-apigw-id: OkU_lFMuZPgF-Ag=
<
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
{"message": "Network error communicating with endpoint"}
```

- Non Use Proxy Integration

```bash
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc-non-proxy -H 'key1:3i95y1yx06'

*   Trying 172.31.22.155...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.31.22.155) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc-non-proxy HTTP/1.1
> Host: v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 06:05:55 GMT
< Content-Type: application/json
< Content-Length: 24
< Connection: keep-alive
< x-amzn-RequestId: a3553730-c26b-4630-ba17-2dd7d7859c4b
< x-amz-apigw-id: OkVGhEjAZPgFt_Q=
< X-Amzn-Trace-Id: Root=1-5ef19bc3-edf99c5445d1c54a43653021
<
{"message":"Forbidden"}
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
```

## How to fix the 504 Gateway Timeout

Let's focus on 504 Gateway Timeout when Use Proxy Integration model in Private API Fargate-private-to-first-api (v944po9kjb)

`curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'key1:3i95y1yx06'` report 504 Gateway Timeout with `Network error communicating with endpoint` error. Due to NLB web-app-fargate-nlb-internal2 can route traffic to private API Fargate-webpage-private (3i95y1yx06), so the Network error happened on Private API Fargate-private-to-first-api (v944po9kjb) to NLB web-app-fargate-nlb-internal2.

So I enable execution log of private API Fargate-webpage-private (3i95y1yx06)

![API-Gateway-Execution-Logs_v944po9kjb](dmedia/API-Gateway-Execution-Logs_v944po9kjb.png)

**Now I add header 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' to my request**

1. Test on Provider VPC of account1 or Consumer VPC of account1 invoke the Private API Fargate-private-to-first-api

Result: Success get resposne from Fargate

```bash
# Provider VPC of account1
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06' 

*   Trying 10.0.0.75...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (10.0.0.75) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
> 
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 09:47:56 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: c8c53db5-6473-45b5-9684-3868d64b7f87
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok1n7GsO5PgFt_Q=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 09:47:56 GMT
< 
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>


# Consumer VPC of account1
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06'
*   Trying 172.16.33.221...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.16.33.221) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 09:57:50 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 12b592ad-e125-4398-b077-d45aa6b93e06
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok3EuFyOZPgFZDw=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 09:57:50 GMT
<
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>
```

2. Test on Consumer VPC of account2 invoke the Private API Fargate-private-to-first-api

Result: Success get resposne from Fargate

```bash
# Consumer VPC of account2
curl -k -v https://v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06'
*   Trying 172.31.22.155...
* TCP_NODELAY set
* Connected to v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn (172.31.22.155) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 09:38:03 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 7f1ac2df-57b3-4b1b-8657-fbf0d2f67aa8
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok0LUHydZPgFbXg=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 09:38:03 GMT
<
* Connection #0 to host v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>
```


## Invoke from Transit NLB from Consumer VPC of account1 and Consumer VPC of account2

1. In account 1: Fargate cluster expose web-app service by target group web-app-fargate-nlb-tg. The web-app-fargate-nlb-tg attached to NLB web-app-fargate-nlb-internal
2. In account 1: VPC link web-app-fargate-vpclink (t0f7wl) point to NLB web-app-fargate-nlb-internal
3. In account 1: Private API Fargate-webpage-private (3i95y1yx06) /webpage-vpc (8hwj4v) path GET method point to VPC link integration web-app-fargate-vpclink (t0f7wl) 
4. In account 1: VPC endpoint vpce-0d4d61b31cecd49fc used for private API Fargate-webpage-private (3i95y1yx06) 
5. In account 1: NLB web-app-fargate-nlb-internal2 route traffic to private API Fargate-webpage-private (3i95y1yx06) 
6. In account 1: VPC link web-app-fargate-vpclink-2ndnlb (lsm4cw) point to NLB web-app-fargate-nlb-internal2
7. In account 1: Private API Fargate-private-to-first-api /webpage-vpc (8hwj4v) path GET method point to VPC link integration web-app-fargate-vpclink-2ndnlb (lsm4cw) 
8. In account 1: VPC endpoint vpce-080de204ca78d2883 in account 1 consumer VPC for private API Fargate-private-to-first-api (v944po9kjb)
9. In account 2: VPC endpoint vpce-08cbeb493b8abab4f used for private API Fargate-private-to-first-api (v944po9kjb)
10. In account 1: Add Transit NLB web-app-fargate-nlb-transit and invoke request from Transit NLB to Private API Fargate-private-to-first-api 
11. In account 2: Add Transit NLB web-app-fargate-nlb-transit and invoke request from Transit NLB to Private API Fargate-private-to-first-api 


![TransitNLBRequest](media/TransitNLBRequest.png)

**add header 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' to my request**

1. Test on Provider VPC of account1 or Consumer VPC of account1 invoke the Private API Fargate-private-to-first-api

Result: Success get resposne from Fargate

- Get the VPCe IP which used for Transit NLB target group
```bash
# Consumer VPC of account1
nslookup v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
Server:         172.16.0.2
Address:        172.16.0.2#53

Non-authoritative answer:
v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn  canonical name = execute-api.cn-northwest-1.amazonaws.com.cn.
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.16.80.49
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.16.33.221
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.16.116.127

telnet 172.16.80.49 443
Trying 172.16.80.49...
Connected to 172.16.80.49.
Escape character is '^]'.
^]
telnet> quit
```

- Create Transit NLB web-app-fargate-nlb-transit in Consumer VPC of account1 and target group point to above IP with port 443

Make sure the backend targets are healthy

![TransitNLB-TG2](media/TransitNLB-TG2.png)

- Testing

```bash
# Consumer VPC of account1

curl -k -v https://web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06'
*   Trying 172.16.97.170...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn (172.16.97.170) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 10:18:34 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 45ad08fd-7747-4ed5-b1b5-b002e8782c4c
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok6HFEbg5PgFctQ=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 10:18:34 GMT
<
* Connection #0 to host web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>


# Provider VPC of account1

curl -k -v https://web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06'
*   Trying 172.16.97.170...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn (172.16.97.170) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
> 
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 10:18:17 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 23f4ca50-21da-45aa-baaf-aaf2067897a2
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok6EiGyqZPgFquA=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 10:18:17 GMT
< 
* Connection #0 to host web-app-fargate-nlb-transit-df5272d1cbace6be.elb.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>

```

2. Test on Consumer VPC of account2 invoke the Private API Fargate-private-to-first-api

Result: Success get resposne from Fargate

- Get the VPCe IP which used for Transit NLB target group

```bash
# Consumer VPC of account2
nslookup v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn
Server:         172.31.0.2
Address:        172.31.0.2#53

Non-authoritative answer:
v944po9kjb.execute-api.cn-northwest-1.amazonaws.com.cn  canonical name = execute-api.cn-northwest-1.amazonaws.com.cn.
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.31.38.48
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.31.7.77
Name:   execute-api.cn-northwest-1.amazonaws.com.cn
Address: 172.31.22.155

telnet 172.31.38.48 443
Trying 172.31.38.48...
Connected to 172.31.38.48.
Escape character is '^]'.
^]
telnet> quit
```
- Create Transit NLB web-app-fargate-nlb-transit in Consumer VPC of account1 and target group point to above IP with port 443

Make sure the backend targets are healthy

![TransitNLB-TG](media/TransitNLB-TG.png)

- Testing
```bash
curl -k -v https://web-app-fargate-nlb-transit-179f98e4771e25b2.elb.cn-northwest-1.amazonaws.com.cn/dev/webpage-vpc -H 'Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn' -H 'key1:3i95y1yx06'
*   Trying 172.31.18.243...
* TCP_NODELAY set
* Connected to web-app-fargate-nlb-transit-179f98e4771e25b2.elb.cn-northwest-1.amazonaws.com.cn (172.31.18.243) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
* TLSv1.2 (OUT), TLS header, Certificate Status (22):
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: CN=*.execute-api.cn-northwest-1.amazonaws.com.cn
*  start date: Apr 24 00:00:00 2020 GMT
*  expire date: Apr 14 12:00:00 2021 GMT
*  issuer: C=US; O=Amazon; OU=Server CA 1B; CN=Amazon
*  SSL certificate verify ok.
> GET /dev/webpage-vpc HTTP/1.1
> Host:3i95y1yx06.execute-api.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.61.1
> Accept: */*
> key1:3i95y1yx06
>
< HTTP/1.1 200 OK
< Server: Server
< Date: Tue, 23 Jun 2020 10:36:54 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 51
< Connection: keep-alive
< x-amzn-RequestId: 2f7dd330-ba0a-442b-b28c-a678e75c8abd
< x-amzn-Remapped-Content-Length: 51
< x-amz-apigw-id: Ok8zFHD2ZPgF4UA=
< x-amzn-Remapped-Server: Werkzeug/0.16.0 Python/2.7.5
< x-amzn-Remapped-Date: Tue, 23 Jun 2020 10:36:54 GMT
<
* Connection #0 to host web-app-fargate-nlb-transit-179f98e4771e25b2.elb.cn-northwest-1.amazonaws.com.cn left intact
<html><h1>Hello World From Ray Webpage!</h1></html>

```