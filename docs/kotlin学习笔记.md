[TOC]

## Kotlin

### 1. 条件表达式

```kotlin
val answerString = when {
    count == 42 -> "I have the answer."
    count > 35 -> "The answer is close."
    else -> "The answer eludes me."
}

println(answerString)
```



### 2.匿名函数

```kotlin
// 声明
val Func: (String) -> Int = { input ->
    input.length
}

// 调用
val len: Int = Func("Android")
```



### 3. 高阶函数

```kotlin
// 函数作为参数传入
val Func1(str: String, Func2:(String)->Int): Int{
    return Func2(str)
}

// 调用
Func1("kotlin", { input -> input.length })

// 若匿名函数是最后一个参数，则可以简化为：
Func1("kotlin"){ input -> 
	input.length
}
```



### 4. 伴生对象

相当于`static`关键字

```kotlin
class LoginFragment: Fragment(){
    // ...
    companion object{
        private val num: Int = 10
        private const val TAG = "LoginFragment"
    }
}
```



### 5. 属性委托

```kotlin
// viewModels 可检索当前 Fragment 的 ViewModel
private val viewModel: LoginViewModel by viewModels()
```



### 6. `!`、`!!`和`?`

**`String!` 可以表示 `String` 或 `String?`**

`!!`表示左侧的变量为非空，如：

```kotlin
val account = Account("name", "type")
val accountName = account.name!!.trim()
```



### 7. lateinit

通过 `lateinit` 关键字，可以避免在构建对象时初始化属性

如将下面代码：

```kotlin
class LoginFragment : Fragment() {
    private var statusTextView: TextView? = null	// 最好使用lateinit

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
            super.onViewCreated(view, savedInstanceState)

            statusTextView = view.findViewById(R.id.status_text_view)
            statusTextView?.setText(R.string.auth_failed)
    }
}
```

改为：

```kotlin
class LoginFragment : Fragment() {
    private lateinit var statusTextView: TextView

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
            super.onViewCreated(view, savedInstanceState)

            statusTextView = view.findViewById(R.id.status_text_view)
            statusTextView.setText(R.string.auth_failed)
    }
}
```



### 8. 数据双向绑定——DataBinding和ViewBinding

`ViewBinding`比`DataBinding`性能要更好，这里介绍`ViewBinding`的用法：

#### 1. build.gradle中添加支持

```
android {
    buildFeatures {
        viewBinding true
    }
}
```

#### 2. Activity中使用

**使用前：**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

**使用后：**

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding	// 声明私有变量

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)	// 使用inflate()获取实例
        setContentView(binding.root)	// 设置视图
    }
}
```



#### 3. Fragment中使用

**使用前：**

```kotlin
class FirstFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_first, container, false)
    }

}
```

**使用后：**

```kotlin
class FirstFragment : Fragment() {
    private var _binding: FragmentFirstBinding? = null
    private val binding get() = _binding!!	// 获取_binging的非 null 实例

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root	// 返回视图
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }

}
```



参考博客:

[在 Android 中使用 ViewBinding](https://dev.to/theimpulson/working-with-view-binding-in-android-using-kotlin-27gn)

[DataBinding和ViewBinding](https://zhuanlan.zhihu.com/p/342447545)





### 9. Obsever

一个被观察者`(夜)`和多个观察者`(十大家族、帕格、吉黑德、昆等)`的依赖关系，当被观察者发生变化时，通知所有观察者发生改变



#### 10. Fragment的生命周期

![](./img/fragment_lifecycle.png)