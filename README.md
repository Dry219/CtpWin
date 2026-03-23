# CtpWin 插件使用说明

综合交易平台（Comprehensive Transaction Platform）简称 CTP，是服务于期货市场的业务管理系统。CtpWin 插件可以订阅期货市场的实时行情数据、查询商品信息，它解决了 DolphinDB 官方 ctp 插件不支持 Windows 系统的问题。

## 安装插件

### 版本要求

DolphinDB Server: 3.00.5.x 系列版本，目前支持 x86-64 的 Windows 版本。

CTP 对应看穿式监管生产版本 (版本号: v6.7.2_20230913)。

**警告**：如果您的 Server 版本是其他版本，强行加载可能导致引擎报错或服务端崩溃！

### 安装步骤

1. 在 DolphinDB 客户端中使用 `listRemotePlugins` 命令查看插件仓库中的插件信息。

```
login("admin", "123456");
listRemotePlugins();
```

2. 使用 `installPlugin` 命令完成插件安装。

```
installPlugin("CtpWin");
```

3. 使用 `loadPlugin` 命令加载插件。

```
loadPlugin("CtpWin");
```

## 接口说明

### connect

**语法**

```
ctp::connect(host, port, [option], [username], [password])
```

**详情**

创建一个和 CTP 行情服务器之间的连接，返回一个句柄。如果该 ip，port 对应的账户已经建立了连接，则会直接返回已经建立了连接的 handle。支持原生关键字传参及缺省参数。

**参数**

**host** 为 STRING 类型标量或向量，表示 CTP 行情服务器的 IP 地址。如果传入向量，将启用多前置机灾备连接。

**port** 为 INT 类型标量或向量，表示 CTP 行情服务器的端口。如果为向量，其长度应与向量 *host* 的长度相等。

**option** 为一个字典，其 key 是一个字符串标量，可以为 "ConcatTime", "ReceivedTime", "OutputElapsed"；其 value 为 BOOL 标量。

-   如果 ConcatTime 为 true，则会将 UpdateTime 和 UpdateMillisec 字段合并为一个 tradeTime 字段置于表末尾，类型为 TIME。
- 如果 ReceivedTime 为 true，则会将接收到数据的时间作为一个字段置于表末尾，类型为 NANOTIMESTAMP。默认为 true。
- 如果 OutputElapsed 为 true，则会将数据进入插件，到插入流表前的时间间隔作为一个字段置于表末尾，类型为 LONG。

**username** 为 STRING 类型标量，代表 CTP 账号名。

**password** 为 STRING 类型标量，代表 CTP 密码。注意：username 和 password 必须同时提供或同时不提供。

### subscribe

**语法**

```
ctp::subscribe(handle, dataType, outputTable, codeList)
```

**详情**

订阅指定合约 ID 的数据到由 connect 指定的流表中。

**参数**

**handle** `connect` 接口返回的句柄。

**dataType** STRING 类型标量，表示需要订阅的数据类型，目前只支持 "marketData"。

**outputTable** 输出表，为共享流表或 IPC 表。

**codeList** STRING 类型向量，表示由合约 ID 组成的数组。

### unsubscribe

**语法**

```
ctp::unsubscribe(handle, dataType, codeList)
```

**详情**

取消订阅指定合约 ID 的数据。

**参数**

**handle** `connect` 接口返回的句柄。

**dataType** STRING 类型标量，表示需要取消订阅的数据类型，目前只支持 "marketData"。

**codeList** STRING 类型向量，表示由合约 ID 组成的数组。

### close

**语法**

```
ctp::close(handle)
```

**详情**

关闭当前连接。

**参数**

**handle** `connect` 接口返回的句柄。

### getSchema

**语法**

```
ctp::getSchema(handle, dataType)
```

**详情**

返回一个表，包含两列：name 和 typeInt，分别表示该类型表结构的名字和类型。通过该表来创建具有相同结构的共享流表。

**参数**

**handle** `connect` 接口返回的句柄。

**dataType** STRING 类型标量，表示行情（marketData）类型。

### getStatus

**语法**

```
ctp::getStatus(handle)
```

**详情**

获取订阅状态信息。

**参数**

**handle** `connect` 接口返回的句柄。

**返回值**

返回一个 dict，包含两个键值对。

- subStatus 键对应的是一个 8 列的表格：

    | 列名                      | 含义            | 类型            |
    | ----------------------- | ------------- | ------------- |
    | **topicName**           | 订阅的名称         | STRING        |
    | **startTime**           | 订阅开始的时间       | NANOTIMESTAMP |
    | **endTime**             | 订阅结束的时间       | NANOTIMESTAMP |
    | **firstMsgTime**        | 第一条消息收到的时间    | NANOTIMESTAMP |
    | **lastMsgTime**         | 最后一条消息收到的时间   | NANOTIMESTAMP |
    | **processedMsgCount**   | 已经处理的消息数      | LONG          |
    | **lastErrMsg**          | 最后一条错误信息      | STRING        |
    | **failedMsgCount**      | 处理失败的消息数      | LONG          |
    | **lastFailedTimestamp** | 最后一条错误消息发生的时间 | NANOTIMESTAMP |

- codeList 键对应的是一个 STRING VECTOR，里面为目前订阅的所有合约。

### queryInstrument

**语法**

```
ctp::queryInstrument(host, port, userID, password, brokerID, appID, authCode, [exchangeID], [timeout]);
```

**详情**

通过登录 CTP 交易端账号，查询并返回包含所有合约信息的表。具体字段含义请参考 CTP 官方文档。

**注意**：本函数需要有效的 CTP 交易端（穿透式监管）账号，如果只使用行情端的账号将无法获取结果并会提示超时或认证失败。

**参数**

**host** 为 STRING 类型标量或向量，表示 CTP 交易服务器的 IP 地址。

**port** 为 INT 类型标量或向量，表示 CTP 交易服务器的端口。如果为向量，其长度应与向量 host 的长度相等。

**userID** STRING 类型标量，指用户代码。

**password** STRING 类型标量，指密码。

**brokerID** STRING 类型标量，指经纪公司代码。

**appID** STRING 类型标量，指客户端认证的 App 代码。

**authCode** STRING 类型标量，指客户端认证请求的认证码。

**exchangeID** 可选参数，STRING 类型标量，指交易所代码，如果指定了则只会查询对应交易所的合约；若不填或传 NULL，则默认请求所有交易所的全量合约。

**timeout** 可选参数，为整型标量，必需为正整数。指查询最大等待时间，单位为秒，默认为 10。

## 使用示例

1. 使用 loadPlugin 加载插件

    ```
    loadPlugin("Your_plugin_path/build/CtpWin.txt")
    ```

2. 连接 ctp 服务器

    ```
    handle = ctp::connect(IP, PORT, dict(["ReceivedTime", "ConcatTime", "OutputElapsed"], [true, true, true]))
    ```

3. 获取对应的表结构

    ```
    schema = ctp::getSchema(handle, `marketData)
    ```

4. 创建共享流表

    ```
    share streamTable(100:0, schema.name, schema.typeInt) as marketDataTable;
    ```

5. 订阅

    ```
    ctp::subscribe(handle, `marketData, marketDataTable, codeList);
    ```

6. 取消订阅

    ```
    ctp::unsubscribe(handle, `marketData, codeList)
    ```

7. 开启订阅后后，查看订阅情况

    ```
    ctp::getStatus(handle)
    ```

8. 使用完成后，手动调用接口释放资源

    ```
    ctp::close(handle)
    ```