# IPC009 nas profile -v0.8
 

### 1.1 nas模块 

#### 1.1.1 目录相关 

- 清空目录 

  请求：

```json
  {
    "method": "nas_clear_dir",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```

- 创建目录


  请求：

```json
  {
    "method": "nas_make_dir",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```

 - 扫描
 
  请求：

```json
  {
    "method": "nas_scan",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```
 
- 目录
 
  请求：

```json
  {
    "method": "nas_list_dir",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```

#### 1.1.2 设置相关

- 设置nas
 
  请求：

```json
  {
    "method": "nas_set_config",
	"params": {
                 "state":1,
                 "sync_interval":197000000000
                 "video_retention_time":86400
                 "share":{
                         "type", mType
                         "group", mGroup
                         "addr", mAddr
                         "name", mName
                         "dir", mDir         //目录
                         "user", mUserName  账号
                         "pass", mPwd    //密码
                      }
             },
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```
- 获取nas
 
  请求：

 ```json
  {
    "method": "nas_set_config",
	"params": {
                 "state":1,
                 "sync_interval":197000000000
                 "video_retention_time":86400
                 "share":{
                         "type", mType
                         "group", mGroup
                         "addr", mAddr
                         "name", mName
                         "dir", mDir         //目录
                         "user", mUserName  账号
                         "pass", mPwd    //密码
                      }
             },
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result": "state":1,
                 "sync_interval":197000000000
                 "video_retention_time":86400
                 "share":{
                         "type", mType
                         "group", mGroup
                         "addr", mAddr
                         "name", mName
                         "dir", mDir         //目录
                         "user", mUserName  账号
                         "pass", mPwd    //密码
                      }
  }
  ```


- 重置nas
 
  请求：

 ```json
  {
    "method": "nas_reset",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```


1.1.3 同步信息

- 开始同步nas
 
  请求：

 ```json
  {
    "method": "nas_sync_start",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```

- 停止同步nas
 
  请求：

 ```json
  {
    "method": "nas_sync_stop",
	"params": [],
  }
  ```
 
  > 说明：
  > 
  > - method:  方法名字
  > - params: 传入json 参数


  返回：

  ```json
  {
      "code": 0,
      "result":["OK"]
  }
  ```