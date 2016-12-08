#Special IDOUtils data_processing_options

##Model Durchlaufkuehler
```
#special data for our durchlaufkuehler
#define IDOMOD_PROCESS_PROCESS_DATA                        1
# !!!!! ^--- needed for detecting a core reload/restart and marking all objects active/inactive !!!!!
#define IDOMOD_PROCESS_PROGRAM_STATUS_DATA              2048
#define IDOMOD_PROCESS_HOST_STATUS_DATA                 4096
#define IDOMOD_PROCESS_SERVICE_STATUS_DATA              8192
#define IDOMOD_PROCESS_OBJECT_CONFIG_DATA             262144
#define IDOMOD_PROCESS_MAIN_CONFIG_DATA               524288
#define IDOMOD_PROCESS_AGGREGATED_STATUS_DATA        1048576
#define IDOMOD_PROCESS_STATECHANGE_DATA              8388608
#define IDOMOD_PROCESS_CONTACT_STATUS_DATA          16777216
data_processing_options=27015169
```

##Model NagVis
```
#define IDOMOD_PROCESS_PROCESS_DATA                        1
# !!!!! ^--- needed for detecting a core reload/restart and marking all objects active/inactive !!!!!
#define IDOMOD_PROCESS_COMMENT_DATA                      256
#define IDOMOD_PROCESS_DOWNTIME_DATA                     512
#define IDOMOD_PROCESS_PROGRAM_STATUS_DATA              2048
#define IDOMOD_PROCESS_HOST_STATUS_DATA                 4096
#define IDOMOD_PROCESS_SERVICE_STATUS_DATA              8192
#define IDOMOD_PROCESS_ADAPTIVE_PROGRAM_DATA           16384
#define IDOMOD_PROCESS_ADAPTIVE_HOST_DATA              32768
#define IDOMOD_PROCESS_ADAPTIVE_SERVICE_DATA           65536
#define IDOMOD_PROCESS_OBJECT_CONFIG_DATA             262144
#define IDOMOD_PROCESS_MAIN_CONFIG_DATA               524288
#define IDOMOD_PROCESS_AGGREGATED_STATUS_DATA        1048576
#define IDOMOD_PROCESS_RETENTION_DATA                2097152

data_processing_options=4061953
```

##Model NagVis + Icinga-Web

```
#define IDOMOD_PROCESS_PROCESS_DATA                        1
# !!!!! ^--- needed for detecting a core reload/restart and marking all objects active/inactive !!!!!
#define IDOMOD_PROCESS_COMMENT_DATA                      256
#define IDOMOD_PROCESS_DOWNTIME_DATA                     512
#define IDOMOD_PROCESS_PROGRAM_STATUS_DATA              2048
#define IDOMOD_PROCESS_HOST_STATUS_DATA                 4096
#define IDOMOD_PROCESS_SERVICE_STATUS_DATA              8192
#define IDOMOD_PROCESS_ADAPTIVE_PROGRAM_DATA           16384
#define IDOMOD_PROCESS_ADAPTIVE_HOST_DATA              32768
#define IDOMOD_PROCESS_ADAPTIVE_SERVICE_DATA           65536
#define IDOMOD_PROCESS_OBJECT_CONFIG_DATA             262144
#define IDOMOD_PROCESS_MAIN_CONFIG_DATA               524288
#define IDOMOD_PROCESS_AGGREGATED_STATUS_DATA        1048576
#define IDOMOD_PROCESS_RETENTION_DATA                2097152
#define IDOMOD_PROCESS_STATECHANGE_DATA              8388608
#define IDOMOD_PROCESS_CONTACT_STATUS_DATA          16777216
data_processing_options=29227777
```