[toc]

# 1 Introduce

Instructions on how to start docker and use the client.



# 2 Pull image

```
docker pull auperastor/aupera_crowd:2.0.0
```



# 3 Install docker and firmware

[Onedrive link](https://auperatechvancouver.sharepoint.com/:w:/s/customersupportdocument/EYBy1_LilLVEoF0EYEnov0kBLMVSpFn9wGX52eKs-v-7Wg?e=jTWc1j)



# 4 Start docker and AI service

Please make sure that the server running the U30 can access the Internet , because the U30 device needs to be connected to the Internet to update time information and activate DRM.

## 4.1 Start docker
1. Create docker container
   - Port `56109` is the API server port of crowd flow
   - Port  `56110` is the API server port of car reid

```
sudo docker create --name aupera_crowd_app  -it --privileged=true -v /opt/aupera/crowd/:/opt/aupera/crowd/ -e NFS_ABS_PATH=/opt/aupera/crowd/ -p 56109:56109 -p 56110:56110 aupera_crowd:2.0.0 bash
```
2. Start docker container
```
sudo docker container restart aupera_crowd_app
```
3. Active `DRM` and install AI application to U30
```
sudo docker exec it aupera_crowd_app bash start.sh
```



# 5 How to use client

## 5.1 Car_reid

- Client pkg [Onedrive link](https://auperatechvancouver.sharepoint.com/:u:/s/SW/ETv9xOfN__FFjR9MwIy882YBHctK512Mf90IKqcMxfZAkQ?e=QyZbNZ)

- User Guide [Onedrive link ](https://auperatechvancouver.sharepoint.com/:b:/g/EYG9VhhQtlpCqM1cz2J34U8B6K4LEdZRv8Nf3jcQ34GXQQ?e=GHUIyh)  refer to step 3、4、5

```
[server]
host = http://172.16.1.12:56110
port = 9003
language = en
first = 0
version = v2.0.4
notify_address = http://172.16.1.28:9003
notify_checkbox = 2
onvif_port = 57077
```

**Note**

- host=http://<IP>:<port>

  - IP is the device IP the application is running on. For U30, it is the x86 host IP.

  - Port is the API server port

     

##  5.2 Crowd_flow
- Client pkg  [Onedrive link](https://auperatechvancouver.sharepoint.com/:f:/s/SW/Eo6_xHn4gJxOgyD2IKvuqdoBbysOqTDXP7CU2uu8Sbp80g?e=eCWX5U) 

-  User Guide [Onedrive link ]([Crowd_flow user guide.pdf](https://auperatechvancouver.sharepoint.com/:b:/g/EYTN8DDxf3pDvvixAjBf1N8BY5fLN7sasUrV0nHb801qfQ?e=lHQGDU))  refer to step 3、4
  
  ```
  [server]
  host = http://172.16.1.16:56109 
  port = 9002
  language = en
  first = 0
  version = v2.0.4
  ```

**Note**

- host=http://<IP>:<port>

  - IP is the device IP the application is running on. For U30, it is the x86 host IP.

  - Port is the API server port

    

# 6 FAQ

## 6.1 Car_reid
### 6.1.1 Task crashes

- Login to manager device .

- Edie `D0/.aup_crowd_app/app.ini` and set `pipeline=1` to save pipeline log.

  ```
  # this is configs template for Aupera ai application
  # to use this configs, please copy this template to same folder and rename it to 'app.ini'
  # then reboot device
  
  # we don't support it persistence now
  [persistence]
  #enable = 0               ;task persistence. Default value is 1
  #recover_delay = 30       ;after the manager is started, it will be delayed for a period of time to read the persistent data and start the task. Deafult value is 30s
  
  [api]
  port =  56109              ; http server port. Default data is 44999
  
  [log]
  pipeline = 1               ;pipeline stderr/stdout will be save to /tmp/aup_*_*.log. Default value is 0
  reid_manager = 0           ;reid_manager stderr/stdout will be saved to /tmp/reid_manager_<api_port>.log
  notify = 0                 ;save reid_manager notification to /var/log/aup_crowd_app_manager/notify.log
  
  [task]
  recover = 0    ;if the task stops abnormally and has been running for more than 60s. if recover = 1, try to restart task
  
  [stream_ip_map]
  json_file = /D0/.aup_crowd_app/stream_ip_map.json
  ```

- Run`ps -x`command check manager port

- Run`screen -r aupm` `Ctrl + c` to stop manager

- Run`screen-S aupm -dm mc_manager --manager-port 44101 --worker-port 44102 --engine aup_crowd_app_manager`restart manager

- To start task again, and then `cd /tmp`check log messages to view pipeline log.



### 6.1.2 Client can't receive any result

- Login the manager device and then execute command: `cat /var/log/aup_crowd_app_manager/crowd_reid_upload.log | grep fail`

  1. If there are no logs, please check the input video and AI parameters are available.

  2. If the edge device fails to send notification to the client, please check whether the notification server IP is correct. You can use `ping` command to check.
  3. If notification server IP is correct, please turn off network firewall  or update the firewall of PC.



### 6.1.3 Client can't connect API server 

If the client can connect to the API server at the beginning, then it fails to connect to the server after running for a while.

1. Please login the U30 device `10.10.10.6` to check if the manager process is running. The device whose IP is 10.10.10.6 runs the API server by default

   ```
   root@v205-vitis:~# ps -x | grep mc_manager
    5966 ?        Ss     0:10 SCREEN -S aupm -dm mc_manager --manager-port 44101 --worker-port 44102 --engine aup_crowd_app_manager
    5967 pts/6    Ssl+ 190:23 mc_manager --manager-port 44101 --worker-port 44102 --engine aup_crowd_app_manager
   ```

2. If the process named `mc_manager` is not running now, you need to restart it manually by executing the command line. And you can use `ps -x | grep mc_manager`  to check again. If it doesn't work, you can execute command line  `cd /D0/;./start` to install application for this device again. 

   ```
   screen -S aupm -dm mc_manager --manager-port 44101 --worker-port 44102 --engine aup_crowd_app_manager
   ```



## 6.2 Crowd flow

### 6.2.1 Client can't receive any result

- Login the manager device and the execute command:  `cat /var/log/aup_ai_app_manager/upload.log |grep fail `

- Please refer to 5.1.2 steps

  

## 6.3 Docker issue
### 6.3.1 Driver can't be installed

- Need to `rmmod xocl`install xdma

### 6.3.2 Can't see xdma_user
- Maybe try to reboot the machine 



##  6.4 Adjust the APP installation list

There is a configure file can adjust the application installation list. You can modify this list. 
```
{NFS_ABS_PATH}/CROWD-2.0.0/package/D0-image/ai_app_install.list
```