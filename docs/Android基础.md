## Android基础



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

