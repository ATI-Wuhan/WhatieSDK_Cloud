# Whatie SDK

<!-- TOC -->

- [Whatie SDK](#whatie-sdk)
    - [How to use SDK](#how-to-use-sdk)
        - [**Init SDK in Java**](#init-sdk-in-java)
        - [**Attention**](#attention)
    - [API Doucument](#api-doucument)
        - [User Login](#user-login)
        - [Device List](#device-list)
        - [Publish New Dps](#publish-new-dps)
        - [Publish Timer Task](#publish-timer-task)
        - [Remove Timer Task](#remove-timer-task)
        - [List Timer Task](#list-timer-task)

<!-- /TOC -->

## How to use SDK
### **Init SDK in Java**

```java

    String apiUri = "https://XXX.XXXXX.XXX/api/v1/XXXX";
    String accessId = "XXXXXXXXX";
    String accessKey = "XXXXXXXXXXXXXXXXXXXXXXXXXX";

    // 1.Create WhatieCloudClient
    WhatieCloudClient client = new WhatieCloudClient(accessId, accessKey, apiUri);

    // 2.Create new request object
    RequestMessage request = new RequestMessage();
    // (1) Set api and apiVersion
    request.setApi(apiUri);
    // The apiVersion:/api/v1 -> "1.0", /api/v2 -> "2.0"
    request.setApiVersion("1.0");
    // (2) The default language is "en-us"
    request.setLang(Const.MessageConst.LANG_EN);
    //request.setLang(Const.MessageConst.LANG_CN);

    // 3.Create parameters via HashMap
    Map<String, String> params = new HashMap<>();
    params.put("XXXX", "145");
    params.put("accessId", "XXXXXXXXX");
    params.put("accessKey", "XXXXXXXXXXXXXXXXXXXXXXXXXX");

    // 4.Set parameters
    request.setParams(params);

    // 5.Send request and receive response
    ResponseMessage response = client.sendRequest(request);
    if (response.isSuccess()) {
        // The request succeeds. Now check the result from the json object
        // Note: the JSONObject object method is imported from com.alibaba.fastjson.JSONObject
        String result = JSONObject.toJSONString(response, true);
        System.out.println(result);
    } else {
        // The request fails. Please check the errorCode and errorMessage
        int errorCode = response.getCode();
        String errorMsg = response.getMessage();
        System.out.println(errorMsg);
    }

```

### **Attention**

- Requirements: The accessId and accessKey should be applied from ATI.
- The ClientConfig could be customized:
    ```java
        ClientConfig clientConfig = new ClientConfig();
        clientConfig.setConnectionTimeout(10000);
        clientConfig.setSocketTimeout(10000);
        clientConfig.setMaxConnections(100);
        clientConfig.setMaxErrorRetry(5);
        WhatieCloudClient client = new WhatieCloudClient(accessId, accessKey, apiUri, clientConfig);
    ```
    The ClientConfig's default values:
    ```java
        SOCKET_TIMEOUT = 50000;
        CONNECTION_TIMEOUT = 50000;
        MAX_ERROR_RETRY = 3;
        MAX_CONNECTIONS = 50;
    ```
- Dependency:

    If you encounter conflict in your project because of .jar file while using WhatieSDK.jar, please use WhatieSDK-WD.jar instead, and then please add the following Maven dependencies or import the jar file in your project.
    
    ```xml
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.6</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.49</version>
        </dependency>
    ```

---
## API Doucument

### User Login
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://users.whatie.net/api/v1/login|POST|email||
    |||password|user password MD5 hashed|

- Response
    
    If success, the user id would be returned in the data.
    ```json
    {
        "code":0,
        "data":459,
        "success":true
    }
    ```

### Device List
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://devices.whatie.net/api/v1/deviceList|POST|customerId|user id, from user login request|

- Response

    **Attention**
    - The deviceId is different from devId: deviceId(Integer), devId(String)
    - The typeName represents Device Type which could be "Plug" or "RgbLight" so far (2018.8.27)
    , and typeId which you may not use right now.
    ```json
    {
        "code":0,
        "data":[
            {
                "devId":"0123456abc",
                "dps":{
                    "light":"false",
                    "power":"false"
                },
                "name":"269846",
                "typeName":"Plug",
                "online":true,
                "deviceId":269846,
                "typeId":3,
                "room":"defaultRoom"
            }
        ],
        "success":true
    }
    ```

### Publish New Dps
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://msg.whatie.net/api/v1/changeDps|POST|devId||
    |||dps||

    - dps example:
        - Plug:
            
            turn on(all true)/turn off(all false)

            ```json
            {
                "dps": {
                    "1": true,
                    "2": true
                }
            }
            ```

        - RgbLight:

            Attention: 
            
            The dps of light mode has a key "l" which is lowercase of "L", no number "1".

            The light's dps only lightMode is Integer value, the others are all String values.

            - lightMode1: adjust the brightness value of light

            ```json
            {
                "dps": {
                    "l": "25",
                    "lightMode": 1
                }
            }   
            ```

            - lightMode2: change the color of light via RGB value, l is the brightness value
            ```json
            {
                "dps": {
                    "l": "35",
                    "lightMode": 2,
                    "rgb": "255_237_228"
                }
            }
            ```

            - lightMode4: turn on/off light, on -> true/ off -> false
            ```json
            {
                "dps": {
                    "lightMode": 4,
                    "status": "false"
                }
            }
            ```

            - lightMode5(StreamLight): rgb1/2/3/4 are four kinds of color's RGB values that you want light to change, and l is the brightness value, t is interval(ms) of StreamLight which 1000 is recommended.
            ```json
            {
                "dps": {
                    "l": "35",
                    "lightMode": 5,
                    "rgb1": "255_221_123",
                    "rgb2": "22_145_56",
                    "rgb3": "146_237_97",
                    "rgb4": "46_189_228",
                    "t": "1000"
                }
            }
            ```

- Response
    ```json
    {
        "code": 0,
        "success": true
    }
    ```

### Publish Timer Task
   **Attention**
   
    The timerId would not return in this API. If you need the timerId, please get it by List Timer Task.
   
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://msg.whatie.net/api/v1/setTimer|POST|deviceId|device id|
    |||timerType|eg.0000001 only Monday every week, 1000000 only Sunday every week, 0000000 Only once|
    |||finishTime|Current time, eg. 10:15 -> 1015, 00:01 -> 0001|
    |||dps||
    |||timezone|Timezone, eg.+800, Beijing|
    |||customerId||

- Response
    ```json
    {
        "code":0,
        "success":true
    }
    ```

### Remove Timer Task
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://msg.whatie.net/api/v1/removeTimer|POST|timerId||
    |||customerId||

- Response
    ```json
    {
        "code":0,
        "success":true
    }
    ```

### List Timer Task
- Request

    |URL|TYPE|PARAMS|Description|
    |:---:|:---:|:---:|:---:|
    |https://msg.whatie.net/api/v1/listTimer|POST|devId||

- Response
    ```json
    {
        "code":0,
        "data":[
            {
                "finishTime":"225",
                "timerType":"0000000",
                "clockId":0,
                "dps":{
                    "1":true,
                    "2":true
                },
                "clockStatus":true,
                "timerId":930
            },
            {
                "finishTime":"1801",
                "timerType":"0000000",
                "clockId":1,
                "dps":{
                    "1":true,
                    "2":true
                },
                "clockStatus":true,
                "timerId":931
            }
        ],
        "success":true
    }
    ```