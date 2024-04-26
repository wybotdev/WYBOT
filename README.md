# WYBOTFramework

WYBOTFramework仅供经过认证的第三方开发者使用。

## Build

本框架基于Swift5.8版本编写，使用系统的Async/await Api。

## Installation

### Swift Package Manager

在XCode中选择Swift package -> Add Package dependency,使用git@github.com:wybotdev/WYBOT.git 添加package到项目中。
然后在项目中```import WYBOTFramewokr```按下面的列子使用

## Fucntions

[x] 连接设备
[x] 搜索设备
[x] 注册设备
[x] 获取用户设备列表
[x] 配置设备网络
[x] 设置/查询设备工作模式
[x] 设置/查询设备工作路径
[x] 设置/查询循环预约
[x] 查询设备功能
[x] 查询/升级设备固件
[x] 设置设备工作泳池参数
[] 电量查询（暂不支持，等待固件更新后支持）

## Usage

### 搜索设备

在头部引用WYLib库
```
import WYBOTFramework

```
配置SDK API Key, 联系开发者获取api key
```
WYLib.config(apiKey:"your api key"）
```

配置开发选项
```
WYLib.config(debug: true)
```
debug为true时连接测试服务器，false时连接线上正式服务器

默认超时
设置超时为5秒可以通过
 ```config(timeout: UInt64)```
更改超时等待时间。

确认已经打开蓝牙权限

```
Task{
	let wylib = WYBOTLib()
	let bleState = try await wylib.startBle()
	if bleState == .poweredOn {
		// 蓝牙已经打开
	} else {
		// 蓝牙异常，需要确认权限等
	}
}
```

开始搜索设备

``` 
let wylib = WYBOTLib()
let devices = try await wylib.searchDevice()
for try await device in devices {
	// device 为ScanDevice结构体
}
```

### 连接设备

连接设备有两种方式：
#### 1，通过ScanDevice中的Peripheral连接

```
let connect = try await wylib.connectPeriperal(device.peripheral)
if connect {
	// 连接成功
}
```
#### 2，通过设备列表中的Device连接

```
let connect = try await wylib.connectDevice(device)
if connect {
// 连接成功
}
```

### 配置网络

望圆设备只支持2.4G频段的WiFi网络，通过设备内置的芯片搜索配置。因此移动设备连接5G还是只处于蜂窝网络下都不影响设备连接WiFi。过程如下：
#### 1，通知设备搜索设备附近的WiFi
```
let wifi = try await wylib.searchWiFi()
//wifi 为WYWiFiInfo结构体的数组 
//	public var ssid: String
//	public var rssi: Int8

```
其中```ssid```为网络名字，rssi是信号强度

#### 2，设置WiFi 

```
let result = try await wylib.setWifi(ssid, password: password)
// result Bool 类型返回成功或失败
```

### 用户登录

#### 第三方账户登录


```
		/// 三方认证接口
		/// - Parameters:
		///   - userId: 第三方用户id
		///   - credential: 第三方认证token
		///   - email: 用户邮箱
		///   - name: 用户暱称
		///   - authType: 登录平台，参见AuthType结构体
		/// - Returns: 用户结构体。
let user = wylib.auth2("userId",
							credential: "credential", 
							email: "email", 
							name: "name", 
							authType: .dreame)
```

### 注册机器
``` 
 		/// 注册设备
		/// - Parameters:
		///   - device: 设备
		///   - token: 用户等后系统返回的token
		/// - Returns: 设备
let result = 	wylib.bindDevice(device, token: user.token)
// result 设备，于发送相同
```
 
### 设备使用相关

#### 获取设备列表

```
let devices = wylib.deviceList(user.userId, token: user.token)
//devices:[Devices]
```

#### 查询设备当前工作模式

```
		/// 查询清洗模式
		/// - Returns: 返回清洗模式enum
		/// 模式说明
		/// .floor 池底模式
		/// .wall 池壁模式
		///	.waterline 水线模式
		///	.combo 先清洗池壁，在清洗池底。
		///	.full 标准全池模式，同时清洗池壁和池底
		///	.eco 节能模式，节能模式只清洗池底
		///	.turbo 强力清洗模式，强力模式只清洗池底
		///	combo和full的不同是combo 会先清洗一段时间池壁，剩下的时间都清洗池底。full是同时清洗，遇到墙壁就先上墙。
let mode = wylib.cleanMode()
		/// 查询清洗模式
		/// 通过蓝牙和物联网同时发送请求
		/// - Returns: 返回清洗模式enum
		/// 模式说明
		/// .floor 池底模式
		/// .wall 池壁模式
		///	.waterline 水线模式
		///	.combo 先清洗池壁，在清洗池底。
		///	.full 标准全池模式，同时清洗池壁和池底
		///	.eco 节能模式，节能模式只清洗池底
		///	.turbo 强力清洗模式，强力模式只清洗池底
		///	combo和full的不同是combo 会先清洗一段时间池壁，剩下的时间都清洗池底。full是同时清洗，遇到墙壁就先上墙。
	let mode = wylib.cleanMode(for: deviceId)
```
`cleanMode(for deviceId: String)`方法将自动通过互联网和蓝牙来获取数据。`cleanMode()`方法只使用蓝牙获取当前连接的设备

#### 设置设备工作模式

当设备开启cyle timer 后设置工作模式会失败，cyle timer只工作在池底模式下，不可修改其他模式
``` 
蓝牙通道
let result = wylib.setCleanMode(.floor)
//result 为true时设置成功
双通道
//token为用户登录后返回的token
let result = wylib.setCleanMode(.floor, to: deviceId, with: token)

```
#### 查询设备功能

本api返回设备含有的功能列表，可以依据返回的功能定制显示内容

```
//蓝牙通道
//返回值function 为WYFunction结构体
let function = wylib.deviceFuncs()
//双通道
let function = wylib.deviceFuncs(for: deviceId)
```

#### 查询循环预约

```
//蓝牙通道
		/// 查询循环预约
		/// - Returns: WYCycleTimer 枚举
		///	once 关闭
		///	twice 一次充电清洗2次
		///	three 一次充电清洗3次
		///	four 一次充电清洗4次
let times = wylib.cycleTimer()
//双通道
let times = wylib.cycleTimer(for: deviceId)
```

#### 设置循环预约

```
//蓝牙通道
let result = wylib.setCycleTimer(.once)
// result true 成功，fales 失败
//双通道
let result = wylib.setCycleTimer(.onec, to: deviceId, with: token)

```

#### 通过wifi查询
        /// 查询清洗模式
        /// 通过蓝牙和物联网同时发送请求
        /// - Returns: 返回清洗模式enum
        /// 模式说明
        /// .floor 池底模式
        /// .wall 池壁模式
        ///    .waterline 水线模式
        ///    .combo 先清洗池壁，在清洗池底。
        ///    .full 标准全池模式，同时清洗池壁和池底
        ///    .eco 节能模式，节能模式只清洗池底
        ///    .turbo 强力清洗模式，强力模式只清洗池底
        ///    combo和full的不同是combo 会先清洗一段时间池壁，剩下的时间都清洗池底。full是同时清洗，遇到墙壁就先上墙。
    public func cleanMode(deviceId: String) async throws -> AsyncThrowingStream<WYCleanMode, Error> 
    
#### 通过wifi设置
        /// 设置循环预约
        /// 通过蓝牙和物联网设置
        /// - Parameter timer: WYCycleTimer 结构体
        /// - Returns: 成功后返回true，返回false代表设置未生效
    public func setCycleTimer(_ timer: WYCycleTimer, to deviceId: String, with token: String) async throws -> WYCycleTimer 
