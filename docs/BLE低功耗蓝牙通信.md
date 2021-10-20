## BLE低功耗蓝牙

首先要分清楚`BLE`分为主设备和从设备，从设备作为`Server`将自己广播出去，主设备作为`Client`去连接从设备

### 参数

1. `BluetoothDevice`是蓝牙连接的设备。

2. `BluetoothGatt`是一个蓝牙连接的协议，它定义两个`BLE`设备通过叫做 `Service` 和 `Characteristic` 的东西进行通信，使用它可以进行对蓝牙的一系列操作，可以通过`bluetoothDevice.connectGatt(this,false,callback)`返回的`gatt`来获得。一个`gatt`中有多个`service`，一个`service`中有多个`characteristic`，如下图：

   ![](img/BluetootGatt.png)

   注意，**`GATT` 连接是独占的**。也就是一个 `BLE `外设同时只能被一个中心设备连接。一旦外设被连接，它就会马上停止广播，这样它就对其他设备不可见了。当设备断开，它又开始广播。

3. `BluetoothGattService`是一个蓝牙服务，一个`gatt`中有多个`service`，可以从`bluetoothGatt.getService(Service_UUID)`来获得。

4. `BluetoothGattCharacteristic`是一个蓝牙特征值，一个`service`中有多个`characteristic`，可以从`service.getCharacteristic(Characteristic_UUID)`来获得。通过对`characteristic.setValue(value)`的操作可以写入数据。

5. `UUID`是对设备的一个标识，`service`、`characteristic`都有其自己的`UUID`，主从设备间的的`UUID`应经过协商后保持一致。



### Client端

1. 先拿到`bluetoothManager`

   ```kotlin
   val bluetoothManager = application.getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager	// application是自带的系统组件，Activity、Service等中都有
   ```

2. 再拿到`BluetoothAdapter`

   ```kotlin
   btAdapter = bluetoothManager.getAdapter(); 
   // 也可以通过BluetoothAdapter.getDefaultAdapter()获得
   ```

3. 开始扫描蓝牙设备

   ```kotlin
   btAdapter.startLeScan(BluetoothAdapter.LeScanCallback); 
   ```

4. 从`LeScanCallback`中得到`BluetoothDevice`

   ```kotlin
   private class LeScanCallback:LeScanCallback(){
       override fun onLeScan(device : BluetoothDevice, rssi : Int, scanRecord : byte[]) {
           // 保存当前设备device
       } 
   }
   ```

5. 用`BluetoothDevice`得到`BluetoothGatt`

   ```kotlin
   gatt = device.connectGatt(this, true, gattCallback)
   ```

6. 获取对应的`service`

   ```kotlin
   service = gatt.getService(Service_UUID)
   ```

7. 获取对应的`Characteristic`

   ```kotlin
   characteristic = service.getCharacteristic(Characteristic_UUID)
   ```

8. 此时就可以对`characteristic`写入数据了

   ```kotlin
   characteristic.setValue(value)
   ```

9. 给`server`发送数据

   ```kotlin
   gatt.writeCharacteristic(characteristic)
   ```

   

### Server端
1. 同样的，先拿到`bluetoothManager`

   ```kotlin
   val bluetoothManager = application.getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager	// application是自带的系统组件，Activity、Service等中都有
   ```

2. 再拿到`BluetoothAdapter`

   ```kotlin
   btAdapter = bluetoothManager.getAdapter(); 
   // 也可以通过BluetoothAdapter.getDefaultAdapter()获得
   ```

3. 新建一个`characteristic`

   ```kotlin
   characteristic = BluetoothGattCharacteristic(Characteristic_UUID,BluetoothGattCharacteristic.PROPERTY_WRITE,
   BluetoothGattCharacteristic.PERMISSION_WRITE)	// 后面的属性是自带的，有NOTIFY、READ、WRITE，选择多个可以用or连接
   characteristic.setValue(value)	// 同理，可以在这里设置要传递的参数
   ```

4. 新建一个`service`

   ```kotlin
   service = BluetoothGattService(SERVICE_UUID, BluetoothGattService.SERVICE_TYPE_PRIMARY)
   ```

5. 为`service`添加上这个`characteristic`

   ```kotlin
   service.add(characteristic)
   ```

6. 获取/打开周边

   ```kotlin
   gattServerCallback = GattServerCallback()	// 回调
   gattServer = bluetoothManager.openGattServer(app,gattServerCallback)
   
   // GattServerCallback回调
   private class GattServerCallback : BluetoothGattServerCallback() {
       
       // 连接状态发生改变
       override fun onConnectionStateChange(device: BluetoothDevice, status: Int, newState: Int) {
           super.onConnectionStateChange(device, status, newState)
           val isSuccess = status == BluetoothGatt.GATT_SUCCESS
           val isConnected = newState == BluetoothProfile.STATE_CONNECTED
           if (isSuccess && isConnected) {
               // ... 记录当前设备等
           }
       }
   
       // 接收Client写过来的数据
       override fun onCharacteristicWriteRequest(
           device: BluetoothDevice,
           requestId: Int,
           characteristic: BluetoothGattCharacteristic,
           preparedWrite: Boolean,
           responseNeeded: Boolean,
           offset: Int,
           value: ByteArray?
       ) {
           super.onCharacteristicWriteRequest(device, requestId, characteristic, preparedWrite, responseNeeded, offset, value)
           
           if (characteristic.uuid == MESSAGE_UUID) {
               gattServer?.sendResponse(device, requestId, BluetoothGatt.GATT_SUCCESS, 0, null)
               val message = value?.toString(Charsets.UTF_8)
               Log.d(TAG, "onCharacteristicWriteRequest: Have message: \"$message\"")
           }
       }
   }
   ```

7. 把`service`添加给周边

   ```kotlin
   gattServer.addService(service)
   
   // 可以通过notifyCharacteristicChanged向Client传递消息
   gattServer.notifyCharacteristicChanged(device, characteristic, false);
   ```

8. 获取广播

   ```kotlin
   advertiser = adapter.bluetoothLeAdvertiser
   ```

9. 开始广播

   ```kotlin
   advertiseCallback = DeviceAdvertiseCallback()	// 广播回调
   advertiser.startAdvertising(settings, data, advertiseCallback);
   
   // 广播回调实现
   private class DeviceAdvertiseCallback : AdvertiseCallback() {
       
       //失败
       override fun onStartFailure(errorCode: Int) {
           super.onStartFailure(errorCode)
           // Send error state to display
           Log.d(TAG, "Advertising failed with error: $errorCode")
       }
   
       // 成功
       override fun onStartSuccess(settingsInEffect: AdvertiseSettings) {
           super.onStartSuccess(settingsInEffect)
           Log.d(TAG, "Advertising successfully started")
       }
   }
   ```
   
   
