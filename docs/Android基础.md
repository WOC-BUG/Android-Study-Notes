## Android基础

### Activity携带数据跳转

从`A`跳转到`B`

**Activity A**

```kotlin
// 从A中打开B
var intent = Intent() 
intent.putExtra("data","hello")
intent.setClass(this, B::class.java)  //(当前Activity，目标Activity)  
startActivityForResult(intent, 1)  // 1是请求的标识requestCode
```

```kotlin
/**
* Activity A中收取从B返回得到的数据
* requestCode A请求的标识
* resultCode 从B返回的标识
* data 从B返回的数据
*/
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
	super.onActivityResult(requestCode, resultCode, data)
    // ...
}
```

**Activity B**

```kotlin
// 返回数据
var data = Intent()
data.putExtra("data", "回传数据")
setResult(2, data);  // 2是返回的标识resultCode
finish()
```



### 观察者模式

**使用LiveData/MutableLiveData和Observer进行数据观测：**

1. 在LiveData中放入需要被观测的数据

   ```kotlin
   val data = LiveData<Int>()
   ```

2. 创建Observer，表明该数据变化后要进行的操作

   ```kotlin
   val myObserver = Observer<Int>{
       if(it == 1){
           // 一些被触发的操作
       }
   }
   ```

3. 观察

   ```kotlin
   data.observe(this,loginObserver)
   ```

4. 数据改变

   ```
   data.postValue(1)
   ```

   这样，当`data`发生改变时就会触发`myObserver`进行一些操作。

