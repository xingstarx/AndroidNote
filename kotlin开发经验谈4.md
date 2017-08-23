这篇讲解下kotlin下Android开发的一些代码技巧，不讲深入的，贴一下目前用的到觉得比较好的代码范式，目前都找不到关于Android开发的一些好的kotlin最佳实践，只能自己慢慢摸索了，写下来希望有点用，写的有问题的欢迎指正

## apply的使用
当我们通过as提供的java转kotlin的代码转换时，默认为我们生成的是如下代码

```java
private fun startPropertyAnim(view: View) {
        view.visibility = View.VISIBLE
        val objectAnimator = ObjectAnimator.ofFloat(view, "alpha", 0f, 1f)
        objectAnimator.duration = 300
        objectAnimator.addListener(object : AnimatorListenerAdapter() {})
        objectAnimator.start()
    }
```

通过优化代码，可以得到如下的使用范式(建议使用下面这种)

```java
private fun startPropertyAnim(view: View) {
        view.visibility = View.VISIBLE
        ObjectAnimator.ofFloat(view, "alpha", 0f, 1f).apply {
            duration = 300
            addListener(object : AnimatorListenerAdapter() {})
            start()
        }
    }
```

还可以看下这段代码

```java

var user: User? = null

fun initUser() {
    user = User()
    user?.id = "9088"
    user?.name = "xingxing"
}

fun main(args: Array<String>) {
    initUser()
    println(user?.id + ", " + user?.name)
}

class User {
    var name: String? = null
    var id: String? = null
}
```

initUser()方法中初始化的操作可以优化为

```java
fun initUser() {
    user = User().apply {
        id = "9088"
        name = "xingxing"
    }
}
```

在Android开发中类似这样的场景很多，我们在初始化一些资源的时候，可能还有一些相关的操作也需要一并处理，像上面的创建一个属性动画这样的，需要设置一些状态值，用apply是一种比较好的使用方式

## let的使用
还是看一段代码

```java
var user: User? = null

fun main(args: Array<String>) {
    user = User().apply {
        id = "9088"
        name = "xingxing"
    }

    if ("xingxing" == user?.name && "9088" == user?.id) {
        println("Hello Android")
    }
}

class User {
    var name: String? = null
    var id: String? = null
}
```
使用let后的代码

```java
fun main(args: Array<String>) {
    user = User().apply {
        id = "9088"
        name = "xingxing"
    }
    user?.let { user ->
        if ("xingxing" == user.name && "9088" == user.id) {
            println("Hello Android")
        }
    }
}
```
主要是去掉了?(可空判断，因为我明明赋值了的，user对象不会是null的，user?.let{} 会先判断user不为null，才执行let代码块里面的代码，仅仅只是语法糖的效果)

## 判断多个对象在都是非空情况下执行某段逻辑

```java
var user: User? = null
var banner: Banner? = null

fun main(args: Array<String>) {

    user = User().apply {
        id = "9088"
        name = "xingxing"
    }
    banner = Banner()

    ifAllNotNull(user, banner) {
        user, banner ->
        println("user.id == " + user.id + ", user.name == " + user.name)
    }
}

inline fun <A, B, R> ifAllNotNull(a: A?, b: B?, transform: (a: A, b: B) -> R) =
        if (a != null && b != null) transform(a, b) else null

```

```java
inline fun <A, B, R> ifAllNotNull(a: A?, b: B?, transform: (a: A, b: B) -> R) =
        if (a != null && b != null) transform(a, b) else null
```
关键代码是ifAllNotNull,方法体里面的逻辑做了判断，如果都不为null,才会执行transform方法体的代码，kotlin的一大优势就是可以把方法当做变量参数传递，真是吊吊的，函数式编程

那么三个变量不为null执行某段逻辑怎么写呢？可以改成这样
```java
inline fun <A, B, C, R> ifAllNotNull(a: A?, b: B?, c: C?, transform: (a: A, b: B, c: C) -> R) =
        if (a != null && b != null && c!= null) transform(a, b, c) else null
```
更多的可以自己发挥了
## 定义函数变量
在以前的Android开发中，为了解耦合，我们都是定义接口实现通讯交互的，常见的就是各种XXXListener,那么在kotlin中这种语言下，可以怎么用呢？

这里还是通过举例子描述，一个listActivity,一个listFragment

```java
class ListActivity : AppCompatActivity() {
    var listFragment: ListFragment? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        listFragment = ListFragment.newInstance() as ListFragment
        listFragment?.onClickListener = {
            parent, view, position, id ->
            Toast.makeText(this, "Hello 来自ListActivity", Toast.LENGTH_SHORT).show()
        }
        supportFragmentManager.beginTransaction().replace(R.id.container, listFragment).commit()
    }
}
```

```java

class ListFragment : Fragment() {
    var bookAdapter: BookAdapter? = null
    var onClickListener: ((parent: AdapterView<*>, view: View, position: Int, id: Long) -> Unit)? = null
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        val view = inflater.inflate(R.layout.fragment_list, container, false)
        return view
    }

    fun generateBooks(count: Int): MutableList<Book> {
        val list = mutableListOf<Book>()
        var index = 0
        while (index < count) {
            var book: Book
            if (index % 3 == 0) {
                book = Book("Android开发艺术探索$index", "任玉刚$index")
            } else if (index % 3 == 1) {
                book = Book("Android群英传$index", "徐宜生$index")
            } else {
                book = Book("Android群英传-神兵利器$index", "徐宜生$index")
            }
            list.add(book)
            index++
        }
        return list
    }

    override fun onViewCreated(view: View?, savedInstanceState: Bundle?) {
        bookAdapter = BookAdapter().apply {
            books = generateBooks(10)
            listView.adapter = this
            listView.setOnItemClickListener { parent, view, position, id ->
                Toast.makeText(context, "Hello 来自ListFragment", Toast.LENGTH_SHORT).show()
                onClickListener?.invoke(parent, view, position, id)
            }
        }
    }

    class BookAdapter : BaseAdapter() {
        var books: MutableList<Book> = mutableListOf()

        override fun getView(position: Int, convertView: View?, container: ViewGroup): View {
            val data = getItem(position) as Book
            return (convertView ?: LayoutInflater.from(container.context).inflate(R.layout.view_book_list_item, container, false).apply {
                tag = ViewHolder(this)
            }).apply {
                (tag as ViewHolder).apply {
                    name.text = data.name
                    author.text = data.author
                }
            }
        }

        override fun getItem(position: Int): Any {
            return books[position]
        }

        override fun getItemId(position: Int): Long {
            return position.toLong()
        }

        override fun getCount(): Int {
            return books.size
        }

    }

    class ViewHolder(itemView: View) {
        val author: TextView = itemView.findViewById(R.id.author)
        val name: TextView = itemView.findViewById(R.id.name)
    }

    companion object {
        fun newInstance(): Fragment {
            return ListFragment()
        }
    }

}
```
在ListFragment里面声明了一个函数变量`var onClickListener: ((parent: AdapterView<*>, view: View, position: Int, id: Long) -> Unit)? = null` 形式就跟OnClickListener的代码类似,只不过是一个函数,在listView设置ItemClickListener事件中，调用onClickListener的逻辑，当然了这个逻辑由ListActivity(我们假设存在这样的业务场景，肯定是有的，只不过暂时没想到一个好的场景)实现,我们在listFragment里面实现这样的代码就可以了
```java
listFragment?.onClickListener = {
            parent, view, position, id ->
            Toast.makeText(this, "Hello 来自ListActivity", Toast.LENGTH_SHORT).show()
        }
```

暂时就讲这么多了，越用越觉得有趣...
O(∩_∩)O哈哈~

参考资料
[helloKotlin](https://github.com/xingstarx/helloKotlin)