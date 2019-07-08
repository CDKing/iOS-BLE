# ios蓝牙交互部分

**懒人逻辑理解：蓝牙发射分外围设备(发射)和中心设备(接收)** 
基本的层次关系：***设备-管理器-服务-特征-值。***（管理器开始一对多） 
***

< 一 > 蓝牙发射端(**外围设备**)

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
  NSLog(@"哇哈哈哈哈哈  ：%@",hahah);
}
```



