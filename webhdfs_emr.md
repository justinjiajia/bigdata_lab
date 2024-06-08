
### Settings

- 1 primary instance; type: `m4.large`

- 3 core instances; type: `m4.large`
  
    <img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1644cc8c-d79b-4c48-a194-f5c49478d126">

- EMR release: 7.1.0

- Software configurations
  ```json
  [
      {
          "classification":"core-site",
          "properties": {
              "hadoop.http.staticuser.user": "hadoop"
          }
      },
      {
          "classification": "hdfs-site",
          "properties": {
              "dfs.webhdfs.enabled": "true",
              "dfs.replication": "3"
          }
      }
  ]
  ```
- Make sure the EC2 security groups of both master and slaves have a rule allowing for "ALL TCP" from "My IP"




Official documentation:
https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/WebHDFS.html

<br>

### Example 1



<img width="1283" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/99efc7ff-cd97-4418-9fcd-41b5eb64514b">


 <br> 

Mapping between private DNS and public DNS:

|Role| Private DNS | Public DNS |
|---|---|---|
|master|ip-172-31-53-255.ec2.internal|ec2-54-160-96-204.compute-1.amazonaws.com|
|master|ip-172-31-63-217.ec2.internal|ec2-52-3-255-53.compute-1.amazonaws.com|
|master|ip-172-31-63-54.ec2.internal|ec2-52-91-136-254.compute-1.amazonaws.com|
|master|ip-172-31-49-88.ec2.internal|ec2-100-26-255-155.compute-1.amazonaws.com|


So the HTTP address of the NameNode is `http://ec2-54-160-96-204.compute-1.amazonaws.com:9870`. Run either

```shell
curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/?user.name=hadoop&op=LISTSTATUS"
```
or 
```shell
curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/?user.name=hadoop&op=LISTSTATUS" | jq .
```
to query the status of HDFS root directory:

The quoted string above is constructed by following this format: `"http://<HOST>:<PORT>/webhdfs/v1/<PATH>?[user.name=<USER>&]op=..." `

```shell
(base) jiajia@Jias-MacBook-Pro ~ % curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/?user.name=hadoop&op=LISTSTATUS"    
HTTP/1.1 200 OK
Date: Sat, 08 Jun 2024 17:59:04 GMT
Cache-Control: no-cache
Expires: Sat, 08 Jun 2024 17:59:04 GMT
Date: Sat, 08 Jun 2024 17:59:04 GMT
Pragma: no-cache
X-Content-Type-Options: nosniff
X-FRAME-OPTIONS: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Set-Cookie: hadoop.auth="u=hadoop&p=hadoop&t=simple&e=1717905544178&s=9kFpBzu344D0B0hVfipFrnGx43v2yh3EeHLe/pWXPJo="; Path=/; HttpOnly
Content-Type: application/json
Transfer-Encoding: chunked

{"FileStatuses":{"FileStatus":[
{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16403,"group":"hdfsadmingroup","length":0,"modificationTime":1717869268561,"owner":"hadoop","pathSuffix":"input","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":2,"fileId":16386,"group":"hdfsadmingroup","length":0,"modificationTime":1717868336253,"owner":"hdfs","pathSuffix":"tmp","permission":"1777","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":3,"fileId":16393,"group":"hdfsadmingroup","length":0,"modificationTime":1717868336346,"owner":"hdfs","pathSuffix":"user","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16397,"group":"hdfsadmingroup","length":0,"modificationTime":1717868336357,"owner":"hdfs","pathSuffix":"var","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}
]}}
(base) jiajia@Jias-MacBook-Pro ~ % curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/?user.name=hadoop&op=LISTSTATUS" | jq .       
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   751    0   751    0     0    249      0 --:--:--  0:00:03 --:--:--   249
{
  "FileStatuses": {
    "FileStatus": [
      {
        "accessTime": 0,
        "blockSize": 0,
        "childrenNum": 2,
        "fileId": 16386,
        "group": "hdfsadmingroup",
        "length": 0,
        "modificationTime": 1717868336253,
        "owner": "hdfs",
        "pathSuffix": "tmp",
        "permission": "1777",
        "replication": 0,
        "storagePolicy": 0,
        "type": "DIRECTORY"
      },
      {
        "accessTime": 0,
        "blockSize": 0,
        "childrenNum": 3,
        "fileId": 16393,
        "group": "hdfsadmingroup",
        "length": 0,
        "modificationTime": 1717868336346,
        "owner": "hdfs",
        "pathSuffix": "user",
        "permission": "755",
        "replication": 0,
        "storagePolicy": 0,
        "type": "DIRECTORY"
      },
      {
        "accessTime": 0,
        "blockSize": 0,
        "childrenNum": 1,
        "fileId": 16397,
        "group": "hdfsadmingroup",
        "length": 0,
        "modificationTime": 1717868336357,
        "owner": "hdfs",
        "pathSuffix": "var",
        "permission": "755",
        "replication": 0,
        "storagePolicy": 0,
        "type": "DIRECTORY"
      }
    ]
  }
}
```

To create and write a file:

- Step 1: Submit a HTTP PUT request without sending the file data.
  ```
  curl -i -X PUT "http://<HOST>:<PORT>/webhdfs/v1/<PATH>?op=CREATE
                    [&overwrite=<true |false>][&blocksize=<LONG>][&replication=<SHORT>]
                    [&permission=<OCTAL>][&buffersize=<INT>][&noredirect=<true|false>]"
  ```
  E.g.:
  ```shell
  (base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input/a.txt?user.name=hadoop&op=CREATE"
  HTTP/1.1 307 Temporary Redirect
  Date: Sat, 08 Jun 2024 17:53:07 GMT
  Cache-Control: no-cache
  Expires: Sat, 08 Jun 2024 17:53:07 GMT
  Date: Sat, 08 Jun 2024 17:53:07 GMT
  Pragma: no-cache
  X-Content-Type-Options: nosniff
  X-FRAME-OPTIONS: SAMEORIGIN
  X-XSS-Protection: 1; mode=block
  Set-Cookie: hadoop.auth="u=hadoop&p=hadoop&t=simple&e=1717905187034&s=5Q6uaLeYYHLLxNpKa3aHKmR1SIopzZbY/oe1B4JuD9E="; Path=/; HttpOnly
  Location: http://ip-172-31-49-88.ec2.internal:9864/webhdfs/v1/input/a.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false
  Content-Type: application/octet-stream
  Content-Length: 0
  ```
  Usually the request is redirected to a DataNode where the file data is to be written.
  Replace the private DNS in the returned location URL with its public DNS.

  - Step 2: Submit another HTTP PUT request using the URL in the Location header with the file data to be written.
    ```shell
    curl -i -X PUT -T <LOCAL_FILE> "http://<DATANODE>:<PORT>/webhdfs/v1/<PATH>?op=CREATE..."
    ```
    E.g.:
    ```shell
    (base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT -T Downloads/zh_vocab.txt "http://ec2-100-26-255-155.compute-1.amazonaws.com:9864/webhdfs/v1/input/a.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false"
    HTTP/1.1 100 Continue
    
    HTTP/1.1 201 Created
    Location: hdfs://ip-172-31-53-255.ec2.internal:8020/input/a.txt
    Content-Length: 0
    Access-Control-Allow-Origin: *
    Connection: close
    ```
    
<br>

### Example 2

This time a different DataNode is picked. Remember to replace the private DNS in the returned location URL with its public DNS.

```shell
(base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input/b.txt?user.name=hadoop&op=CREATE&noredirect=true"
HTTP/1.1 200 OK
Date: Sat, 08 Jun 2024 18:15:36 GMT
Cache-Control: no-cache
Expires: Sat, 08 Jun 2024 18:15:36 GMT
Date: Sat, 08 Jun 2024 18:15:36 GMT
Pragma: no-cache
X-Content-Type-Options: nosniff
X-FRAME-OPTIONS: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Set-Cookie: hadoop.auth="u=hadoop&p=hadoop&t=simple&e=1717906536927&s=jf/DAiQRdEPzDeM0ibPTTDTTKfX6qWoIVtQj8AK67RQ="; Path=/; HttpOnly
Content-Type: application/json
Transfer-Encoding: chunked

{"Location":"http://ip-172-31-63-217.ec2.internal:9864/webhdfs/v1/input/b.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false"}%
                                                                       
(base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT -T Downloads/zh_vocab.txt "http://ec2-52-3-255-53.compute-1.amazonaws.com:9864/webhdfs/v1/input/b.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false"
HTTP/1.1 100 Continue

HTTP/1.1 201 Created
Location: hdfs://ip-172-31-53-255.ec2.internal:8020/input/b.txt
Content-Length: 0
Access-Control-Allow-Origin: *
Connection: close
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f31b0550-956e-4da8-8a04-53c645d855bb">


```shell
(base) jiajia@Jias-MacBook-Pro ~ % curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input?user.name=hadoop&op=LISTSTATUS"      
HTTP/1.1 200 OK
Date: Sat, 08 Jun 2024 18:30:37 GMT
Cache-Control: no-cache
Expires: Sat, 08 Jun 2024 18:30:37 GMT
Date: Sat, 08 Jun 2024 18:30:37 GMT
Pragma: no-cache
X-Content-Type-Options: nosniff
X-FRAME-OPTIONS: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Set-Cookie: hadoop.auth="u=hadoop&p=hadoop&t=simple&e=1717907437168&s=8DWfdKm5TbpkZ8wZB/ty/yoXbuTdu10X7q00lztHPoQ="; Path=/; HttpOnly
Content-Type: application/json
Transfer-Encoding: chunked

{"FileStatuses":{"FileStatus":[
{"accessTime":1717869268561,"blockSize":134217728,"childrenNum":0,"fileId":16404,"group":"hdfsadmingroup","length":10047,"modificationTime":1717869268984,"owner":"hadoop","pathSuffix":"a.txt","permission":"644","replication":3,"storagePolicy":0,"type":"FILE"},
{"accessTime":1717870756716,"blockSize":134217728,"childrenNum":0,"fileId":16405,"group":"hdfsadmingroup","length":10047,"modificationTime":1717870756836,"owner":"hadoop","pathSuffix":"b.txt","permission":"644","replication":3,"storagePolicy":0,"type":"FILE"}
]}}
(base) jiajia@Jias-MacBook-Pro ~ % curl -i "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input?user.name=hadoop&op=LISTSTATUS" |jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   557    0   557    0     0    972      0 --:--:-- --:--:-- --:--:--   973
parse error: Invalid numeric literal at line 1, column 9
(base) jiajia@Jias-MacBook-Pro ~ % curl "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input?user.name=hadoop&op=LISTSTATUS" | jq . 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   557    0   557    0     0   1021      0 --:--:-- --:--:-- --:--:--  1023
{
  "FileStatuses": {
    "FileStatus": [
      {
        "accessTime": 1717869268561,
        "blockSize": 134217728,
        "childrenNum": 0,
        "fileId": 16404,
        "group": "hdfsadmingroup",
        "length": 10047,
        "modificationTime": 1717869268984,
        "owner": "hadoop",
        "pathSuffix": "a.txt",
        "permission": "644",
        "replication": 3,
        "storagePolicy": 0,
        "type": "FILE"
      },
      {
        "accessTime": 1717870756716,
        "blockSize": 134217728,
        "childrenNum": 0,
        "fileId": 16405,
        "group": "hdfsadmingroup",
        "length": 10047,
        "modificationTime": 1717870756836,
        "owner": "hadoop",
        "pathSuffix": "b.txt",
        "permission": "644",
        "replication": 3,
        "storagePolicy": 0,
        "type": "FILE"
      }
    ]
  }
}
```

<br>

### Example 3

Try to use a different replication factor for a new file:

```shell
(base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input/c.txt?user.name=hadoop&op=CREATE&replication=2"                                                                                                             
HTTP/1.1 307 Temporary Redirect
Date: Sat, 08 Jun 2024 18:33:18 GMT
Cache-Control: no-cache
Expires: Sat, 08 Jun 2024 18:33:18 GMT
Date: Sat, 08 Jun 2024 18:33:18 GMT
Pragma: no-cache
X-Content-Type-Options: nosniff
X-FRAME-OPTIONS: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Set-Cookie: hadoop.auth="u=hadoop&p=hadoop&t=simple&e=1717907598940&s=Ey2HG3Zx6JDTbMUVgqhhNo9H0S6K2zfJvVGecVclQcs="; Path=/; HttpOnly
Location: http://ip-172-31-63-54.ec2.internal:9864/webhdfs/v1/input/c.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false&replication=2
Content-Type: application/octet-stream
Content-Length: 0

(base) jiajia@Jias-MacBook-Pro ~ % curl -i -X PUT -T Downloads/zh_vocab.txt "ec2-52-91-136-254.compute-1.amazonaws.com:9864/webhdfs/v1/input/c.txt?op=CREATE&user.name=hadoop&namenoderpcaddress=ip-172-31-53-255.ec2.internal:8020&createflag=&createparent=true&overwrite=false&replication=2"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0HTTP/1.1 100 Continue

100 10047    0     0  100 10047      0  14760 --:--:-- --:--:-- --:--:-- 14753HTTP/1.1 201 Created
Location: hdfs://ip-172-31-53-255.ec2.internal:8020/input/c.txt
Content-Length: 0
Access-Control-Allow-Origin: *
Connection: close

100 10047    0     0  100 10047      0   7895  0:00:01  0:00:01 --:--:--  7898

(base) jiajia@Jias-MacBook-Pro ~ % curl "http://ec2-54-160-96-204.compute-1.amazonaws.com:9870/webhdfs/v1/input?user.name=hadoop&op=LISTSTATUS" | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   818    0   818    0     0   1081      0 --:--:-- --:--:-- --:--:--  1082
{
  "FileStatuses": {
    "FileStatus": [
      {
        "accessTime": 1717869268561,
        "blockSize": 134217728,
        "childrenNum": 0,
        "fileId": 16404,
        "group": "hdfsadmingroup",
        "length": 10047,
        "modificationTime": 1717869268984,
        "owner": "hadoop",
        "pathSuffix": "a.txt",
        "permission": "644",
        "replication": 3,
        "storagePolicy": 0,
        "type": "FILE"
      },
      {
        "accessTime": 1717870756716,
        "blockSize": 134217728,
        "childrenNum": 0,
        "fileId": 16405,
        "group": "hdfsadmingroup",
        "length": 10047,
        "modificationTime": 1717870756836,
        "owner": "hadoop",
        "pathSuffix": "b.txt",
        "permission": "644",
        "replication": 3,
        "storagePolicy": 0,
        "type": "FILE"
      },
      {
        "accessTime": 1717871682507,
        "blockSize": 134217728,
        "childrenNum": 0,
        "fileId": 16406,
        "group": "hdfsadmingroup",
        "length": 10047,
        "modificationTime": 1717871682628,
        "owner": "hadoop",
        "pathSuffix": "c.txt",
        "permission": "644",
        "replication": 2,
        "storagePolicy": 0,
        "type": "FILE"
      }
    ]
  }
}
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f1fc1b08-0d98-4bde-aeb2-c9cfd6fbd622">

