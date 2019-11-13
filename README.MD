# websocket-mq-poc
## goals
- backend microservice in all major programming language can do live interact with front-end web apps asyncally
- maintenance-free ws server

## architecture
overview

```
                                                 <<ws srv>>
<<ws client>>       <<r-proxy>>
                                                +---------------+
+--------+           +--------+                ++ mq hosts|
| webapp +-----------+  nginx +-----------------| mqtt-enabled  |
+--------+           +--------+                |----------------+
                                               +---------------+
                                                     ^   |
                                                     |   |
                                                     |   |
                                                     |   |
                                                     |   v
                                                 +---+---+----+
                                                 |backend     |
                                                 |mu-services |
                                                 +------------+

```

## tech stacks
- mqttws31.js https://www.eclipse.org/paho/clients/js/
- emqx https://hub.docker.com/emqx/emqx
- all backend frameworks that have mqtt connection adaptor or lib available

## proposed communication model and spec
### topics
const APP_TOPICS_ROOT = 'evsuites/'
- topic WEBAPP_TOPIC_STATUS
  - path: ${APP_TOPICS_ROOT} + 'webapp/status/'
  - payload
    - type: json
    - value: {online: true|false, uid: '{user_defined_id}'}   # uid can be ommited since the topic path contains it
    - qos: 1
    - retain: true.    # this is cleaned by backend service
- topic WEBAPP_TOPIC_REQUEST
  - path: ${APP_TOPICS_ROOT} + 'webapp/req/'
  - payload
    - type & value: application defined
  - qos: 1
  - retian: false
- topic WEBAPP_TOPIC_RESPONSE
  - path: ${APP_TOPICS_ROOT} + 'webapp/res/'
  - payload
    - type: json
    - value: application defined
    - qos: 1
    - retian: false
- topic DEVICE_TOPIC_STATUS
  - path ${APP_TOPICS_ROOT} + 'device/status/'
  - payload
    - type: json
    - value: {online: true|false, recording:true|false, broadcasting: true|false, motionEnabled: true|false}
    - qos: 1
    - retian: true   #important!!!

### web client
- setup
  - generate uid: ${org-id} + ':' +${user-id} + ':' + randomStr(4)
    - randomStr is used for multiple connection with same user account
  - connect
  - in connect.success callback:
    - send message to WEBAPP_TOPIC_STATUS with online=true
    - subscribe to topics WEBAPP_TOPIC_RESPONSE + uid
  - call backend api to get user scope devices list: DEV_SN_LIST
  - [optional] subscribe to all device status on topic ${DEVICE_TOPIC_STATUS} or individual device with sn in ${DEV_SN_LIST}

### backend app
- setup
- 




