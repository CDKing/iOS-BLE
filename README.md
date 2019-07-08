# ios蓝牙交互部分

**懒人逻辑理解：蓝牙发射分外围设备(发射)和中心设备(接收)** 
基本的层次关系：***设备-管理器-服务-特征-值。***（管理器开始一对多） 
***

## < 一 > 蓝牙发射端(**外围设备**)

#import <CoreBluetooth/CoreBluetooth.h>  
协议：CBPeripheralManagerDelegate 

1）一个外围设备管理器

```
CBPeripheralManager *peripheralMgr; 
peripheralMgr = [[CBPeripheralManager alloc]initWithDelegate:self queue:nil];  
``` 
// 当外围设备管理器被实例化后，协议函数peripheralManagerDidUpdateState会被调用 

2）管理器状态管理
```
-(void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral
{
  // 判断蓝牙设备状态
  // iOS >= 10 的状态判断
  if (peripheral.state == CBManagerStatePoweredOn) {
    // 蓝牙可以正常使用且已打开
    // 这是一个设置下一步的方法，详情请看3）
    [self setUpBlueTooth];
  } else {
    // 蓝牙设备无法正常使用或未打开
  }
  //  iOS 10 以前的状态判断 就是设备蓝牙状态的判断名称变化了而已
  if (peripheral.state == CBPeripheralManagerStatePoweredOn) {
    // 蓝牙可以正常使用且已打开
  } else {
    // // 蓝牙设备无法正常使用或未打开
  }
}
```

3）搭建服务

```
-(void)setUpBlueTooth
{
  // 搭建假数据 
  NSData *dummyData = [@"HelloWorld" dataUsingEncoding:NSUTF8StringEncoding]; 
  // 特征的搭建 
  // 有读，写，订阅 三种，如果value不为空则不能只能读与订阅（订阅和读类似） 
  // UUID 我是完全瞎写的= = 
  CBMutableCharacteristic *character = [[CBMutableCharacteristic alloc]initWithType:[CBUUID UUIDWithString:@"AAAAAAAA-E798-4D5C-8DCF-49908332DF9F"] properties:CBCharacteristicPropertyRead value:dummyData permissions:CBAttributePermissionsReadable]; 
  // 写 分WithoutResponse和WithResponse，蓝牙的中心与外围必须一致 
  CBMutableCharacteristic *character2 = [[CBMutableCharacteristic alloc]initWithType:[CBUUID UUIDWithString:@"AAAAAAAA-E798-4D5C-8DCF-49908332DF8F"] properties:CBCharacteristicPropertyWriteWithoutResponse value:nil permissions:CBAttributePermissionsWriteable]; 
  // 特征加入服务中 
  service = [[CBMutableService alloc]initWithType:[CBUUID UUIDWithString:@"BBBBBBBB-6525-4489-801C-1C060CAC9767"] primary:YES]; 
   [service setCharacteristics:@[character,character2]]; 
  // 服务加入管理中 回调didAddService协议方法 
  [peripheralMgr addService:service]; 
}
``` 

4）发射广播

```
-(void)peripheralManager:(CBPeripheralManager *)peripheral didAddService:(CBService *)service error:(NSError *)error
{
  if (error) {
    // 发生错误未能成功添加服务
  } else {
    [peripheralMgr startAdvertising:@{
                      // 这是广播名称,可以作为中心设备接收判断的Flag
                      CBAdvertisementDataServiceUUIDsKey : @[[CBUUID UUIDWithString:@"CCCCCCCC-E798-4D5C-8DCF-49908332DF9F"]]
                     ,CBAdvertisementDataLocalNameKey : @"GoodGoodStudyDaydayUp"}];
  }
}
// 广播发射之后会走楼下的协议回调方法
-(void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral error:(NSError *)error
{
  if (error) {
    // 发送服务错误
  }else{
    // 发送服务成功
  }
}

```

5)读写请求的接收 

**读请求 didReceiveReadRequest 回调方法** 

**写请求 didReceiveWriteRequests 回调方法**

```
// 得到写请求的详细数据看楼下
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveWriteRequests:(NSArray<CBATTRequest *> *)requests
{
  NSLog(@"收到写数据请求: %@",peripheral);
  NSLog(@"数据请求： %@ , %lu",requests[0],(unsigned long)requests.count);
  CBATTRequest *request = requests[0];
  CBCharacteristic *cbs = request.characteristic;
  NSLog(@"得到特征的UUID ： %@",cbs.UUID.description);
  NSString*hahah = [[NSString alloc]initWithData:request.value encoding:NSUTF8StringEncoding];
  NSLog(@"获取  ：%@",hahah);
  NSLog(@"得到特征的UUID ： %@",cbs.UUID.description);
  NSString*hahah = [[NSString alloc]initWithData:request.value encoding:NSUTF8StringEncoding];
  NSLog(@"得到数据  ：%@",hahah);
}
```

6)变更（被订阅）内容

```
[Manager updateValue:Data forCharacteristic:kkk onSubscribedCentrals:nil];
// 变更后会走楼下的回调函数

- (void)peripheralManagerIsReadyToUpdateSubscribers:(CBPeripheralManager *)peripheral{
  // 准备更新订购者
}
```

***

## < 一 > 蓝牙接收端(**中心设备**)

发射时一层一层包装好了发出去，接收当然就是一层一层拿下来的收回来啦！

#import <CoreBluetooth/CoreBluetooth.h>

协议 ： CBCentralManagerDelegate,CBPeripheralDelegate

1)一个中心管理器

```
CBCentralManager *centralMgr;
// 楼下的也一般声明一下意思意思
// 用于存储设备信息(苹果要求)
NSMutableArray* peripheralMutableArray;
NSMutableArray* RSSRMutableArray;
// 目标设备
CBPeripheral *targetPeripheral;
// 初始化后centralManagerDidUpdateState执行
centralMgr = [[CBCentralManager alloc]initWithDelegate:self queue:nil];
```

2）管理器状态管理

```
-(void)centralManagerDidUpdateState:(CBCentralManager *)central
{
  // 判断设备蓝牙状态 iOS 10 前
  if (central.state == CBCentralManagerStatePoweredOn) {
    // 可用
  } else {
    // 不可用
  }
  // 判断设备蓝牙状态 iOS 10 前
  if (central.state == CBManagerStatePoweredOn) {
    // 设备蓝牙状态可用
    // 设备可用，开始搜索周围设备,之后发现设备后会执行didDiscoverPeripheral回调方法
    [centralMgr scanForPeripheralsWithServices:nil options:nil]; 
  } else {
     // 设备蓝牙状态异常，请开启蓝牙或检查设备蓝牙是否可用
  }
}
```

3)寻找目标

**找设备**

```
-(void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI
{
  // 苹果要求你做的。。。
  [peripheralMutableArray addObject:peripheral];
  [RSSRMutableArray addObject:RSSI];
  // 获取广播名称
  NSString* adName = [advertisementData objectForKey:@"kCBAdvDataLocalName"];
  NSLog(@"广播名称：%@",adName);
  // 获取外围设备信息
  NSLog(@"外围设备信息:%@",peripheral);
  if ([adName isEqualToString:@"GoodGoodStudyDaydayUp"]) {
    // 发现目标设备,开始连接。结果会通过回调函数返回
    // 开始连接目标设备，结果看楼下的函数
    [centralMgr connectPeripheral:peripheral options:nil];
  }
}

-(void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
  // 连接外围设备失败
}

-(void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
  // 断开外围设备连接
}

-(void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral
{
  NSLog(@"设备连接成功：%@",peripheral.description);
  targetPeripheral = peripheral;
  // 搜索设备内服务,发现后didDiscoverServices调用
  [peripheral setDelegate:self];
  [peripheral discoverServices:nil];
}
```

**找服务**

```
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error
{
  if (error) {
    NSLog(@"扫描服务失败：%@",error);
  } else {
     NSLog(@"扫描到了服务个数：%lu",(unsigned long)peripheral.services.count);
     for (CBService* service in peripheral.services)
     {
        NSLog(@"被扫描到的服务：%@",service.UUID);
        // 获取对应UUID的服务
        if ([service.UUID.UUIDString isEqualToString:@"BBBBBBBB-6525-4489-801C-1C060CAC9767"])
        {
          // 发现目标服务
          // 搜索服务，结果在楼下的回调函数调用中体现
          [peripheral discoverCharacteristics:nil forService:service];
        }
     }
  }
}
```

**找特征**

```
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error
{
  if (error) {
    NSLog(@"搜索设备特征失败：%@",error);
  } else {
    NSLog(@"%@的特征个数：%ld",service.UUID,(unsigned long)service.characteristics.count);
    for (CBCharacteristic* character in service.characteristics)
    {
       NSLog(@"%@的character：%@",service.UUID,character.UUID);
       if ([character.UUID.description isEqualToString:@"AAAAAAAA-E798-4D5C-8DCF-49908332DF9F"])
       {
          NSLog(@"发现目标特征");
          // 读特征
          [peripheral readValueForCharacteristic:character];
       }
       // 订阅的处理
       if ([character.UUID.description isEqualToString:@"？？？？？？"]) {
          [peripheral setNotifyValue:YES forCharacteristic:character];
          [peripheral discoverDescriptorsForCharacteristic:character];
       }
    }
  }
}
```

**监听值的变化与修改**

```
// 读与订阅都会走楼下的方法
-(void)peripheral:(CBCentralManager *)peripheral didUpdateValueForCharacteristic:(nonnull CBCharacteristic *)characteristic error:(nullable NSError *)error
{
  if (error) {×} else {
    // 发现目标特征值更新
    NSLog(@"发现目标特征值更新:%@",characteristic.UUID.description);
    // 获取值
    NSString *str = [[NSString alloc]initWithData:characteristic.value encoding:NSUTF8StringEncoding];
    NSLog(@"获取目标值:%@",str);
    // 假如我需要写数据了的话
    if ([characteristic.UUID.description isEqualToString:@"AAAAAAAA-E798-4D5C-8DCF-49908332DF8F"]) 
    {
       // 写数据
       NSLog(@"写数据中。。。");
       NSData *writeData  = [@"世界你好" dataUsingEncoding:NSUTF8StringEncoding];
       // 写完数据后会走到外围设备的对应回调函数中(看第一部分的5）)
        [targetPeripheral writeValue:writeData forCharacteristic:characteristic type:CBCharacteristicWriteWithoutResponse];
    }
  }
}
```

***

## BLE4.0 发射，两台iPhone设备交互的情况下 可以让自己的值0.01秒这样的疯狂变化，只要是withoutresponse，接收端就不会出现数据丢失的情况
（信号干扰等除外）

