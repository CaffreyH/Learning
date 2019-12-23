## <center>Java8 Optional</center>

## 重要概念：

**1. isPresent() 与 obj != null 无任何区别, 我们的生活依然在步步惊心. 而没有 isPresent() 作铺垫的 get() 调用在 IntelliJ IDEA 中会收到告警。调用 Optional.get() 前不事先用 isPresent() 检查值是否可用. 假如 Optional 不包含一个值, get() 将会抛出一个异常！**

**2. 把 Optional 类型用作属性或是方法参数在 IntelliJ IDEA 中更是强力不推荐的！**

**3. 使用任何像 Optional 的类型作为字段或方法参数都是不可取的. Optional 只设计为类库方法的, 可明确表示可能无值情况下的返回类型. Optional 类型不可被序列化, 用作字段类型会出问题的！！！**

## 正确的打开姿势
如果使用了Optional，但却仍然使用If(isPresent),这将与使用if(null),没有区别，这样的用法是毫无意义的，正确的打开姿势：(实例代码在最后)

**1. 存在即返回, 无则提供默认值**

	return user.orElse(null);  //而不是 return user.isPresent() ? user.get() : null;
	return user.orElse(UNKNOWN_USER);

**2. 存在即返回, 无则由函数来产生**

	return user.orElseGet(() -> fetchAUserFromDatabase()); 
	//而不要 return user.isPresent() ? user: fetchAUserFromDatabase();
**3. 存在才对它做点什么**

	user.ifPresent(System.out::println);
	//而不要下边那样
	if (user.isPresent()) {
	  System.out.println(user.get());
	}
**4. map 函数隆重登场**
当 user.isPresent() 为真, 获得它关联的 orders的映射集合, 为假则返回一个空集合时, 我们用上面的 orElse, orElseGet 方法都乏力时, 那原本就是 map 函数的责任, 我们可以这样一行

	return user.map(u -> u.getOrders()).orElse(Collections.emptyList())

	//上面避免了我们类似 Java 8 之前的做法
	if(user.isPresent()) {
	  return user.get().getOrders();
	} else {
	  return Collections.emptyList();
	}
map 是可能无限级联的, 比如再深一层, 获得用户名的大写形式

	return user.map(u -> u.getUsername())
			   .map(name -> name.toUpperCase())
			   .orElse(null);
这要搁在以前, 每一级调用的展开都需要放一个 null 值的判断

	User user = .....
	if(user != null) {
	  String name = user.getUsername();
	  if(name != null) {
		return name.toUpperCase();
	  } else {
		return null;
	  }
	} else {
	  return null;
	}
filter() :如果有值并且满足条件返回包含该值的Optional，否则返回空Optional。

	Optional<String> longName = name.filter((value) -> value.length() > 6);  
	System.out.println(longName.orElse("The name is less than 6 characters"));//输出Sanaulla  
	
**4. flatMap()**
如果有值，为其执行mapping函数返回Optional类型返回值，否则返回空Optional。flatMap与map（Funtion）方法类似，区别在于flatMap中的mapper返回值必须是Optional。调用结束时，flatMap不会对结果用Optional封装。
flatMap方法与map方法类似，区别在于mapping函数的返回值不同。map方法的mapping函数返回值可以是任何类型T，而flatMap方法的mapping函数必须是Optional。

	upperName = name.flatMap((value) -> Optional.of(value.toUpperCase()));  
	System.out.println(upperName.orElse("No value found"));//输出SANAULLA  


## Optional基础：
以下为Optional的基本用法：参考：
https://juejin.im/post/5d834da15188250150110e82
## 一、使用Optional引言
### 1.1、代码问题引出
在写程序的时候一般都遇到过 NullPointerException，所以经常会对程序进行非空的判断：
    
    User user = getUserById(id);
    if (user != null) {
        String username = user.getUsername();
        System.out.println("Username is: " + username); // 使用 username
    }
为了解决这种尴尬的处境，JDK 终于在 Java8 的时候加入了 Optional 类，查看 Optional 的 javadoc 介绍：

`A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.`

这是一个可以包含或者不包含非 null 值的容器。如果值存在则 isPresent()方法会返回 true，调用 get() 方法会返回该对象。
### 1.2、解决进阶
我们假设 getUserById 已经是个客观存在的不能改变的方法，那么利用 isPresent 和 get 两个方法，我们现在能写出下面的代码：

	Optional<User> user = Optional.ofNullable(getUserById(id));
	if (user.isPresent()) {
		String username = user.get().getUsername();
		System.out.println("Username is: " + username); // 使用 username
	}

好像看着代码是优美了点,但是事实上这与之前判断 null 值的代码没有本质的区别，反而用 Optional 去封装 value，增加了代码量。所以我们来看看 Optional 还提供了哪些方法，让我们更好的（以正确的姿势）使用 Optional。

## 二、Optional三个静态构造方法
### 1）概述：

JDK 提供三个静态方法来构造一个 Optional：

Optional.of(T value)

    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

该方法通过一个非 null 的 value 来构造一个 Optional，返回的 Optional 包含了 value 这个值。对于该方法，传入的参数一定不能为 null，否则便会抛出 NullPointerException。

Optional.ofNullable(T value)

    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

该方法和 of 方法的区别在于，传入的参数可以为 null —— 但是前面 javadoc 不是说 Optional 只能包含非 null 值吗？我们可以看看 ofNullable 方法的源码。

原来该方法会判断传入的参数是否为 null，如果为 null 的话，返回的就是 Optional.empty()。


	Optional.empty()
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

该方法用来构造一个空的 Optional，即该 Optional 中不包含值 —— 其实底层实现还是 如果 Optional 中的 value 为 null 则该 Optional 为不包含值的状态，然后在 API 层面将 Optional 表现的不能包含 null值，使得 Optional 只存在 包含值 和 不包含值 两种状态。

### 2）分析：

前面 javadoc 也有提到，Optional 的 isPresent() 方法用来判断是否包含值，get() 用来获取 Optional 包含的值 —— 值得注意的是，如果值不存在，即在一个Optional.empty 上调用 get() 方法的话，将会抛出 NoSuchElementException异常。

### 3）总结：

1）Optional.of(obj): 它要求传入的 obj 不能是 null 值的, 否则还没开始进入角色就倒在了 NullPointerException 异常上了. 2）Optional.ofNullable(obj): 它以一种智能的, 宽容的方式来构造一个 Optional 实例. 来者不拒, 传 null 进到就得到 Optional.empty(), 非 null 就调用 Optional.of(obj). 那是不是我们只要用 Optional.ofNullable(obj) 一劳永逸, 以不变应二变的方式来构造 Optional 实例就行了呢? 那也未必, 否则 Optional.of(obj) 何必如此暴露呢, 私有则可。

### 三、Optional常用方法详解
#### 3.1、Optional常用方法概述
**Optional.of(T t)**

将指定值用 Optional 封装之后返回，如果该值为 null，则抛出一个 NullPointerException 异常。

**Optional.empty()**

创建一个空的 Optional 实例。

**Optional.ofNullable(T t)**

将指定值用 Optional 封装之后返回，如果该值为 null，则返回一个空的 Optional 对象。

**isPresent**

如果值存在返回true,否则返回false

**Optional.get()**

如果该值存在，将该值用 Optional 封装返回，否则抛出一个 NoSuchElementException 异常。

**orElse(T t)**

如果调用对象包含值，返回该值，否则返回t。

**orElseGet(Supplier s)**

如果调用对象包含值，返回该值，否则返回 s 获取的值。

**orElseThrow()**

它会在对象为空的时候抛出异常。

**map(Function f)**

如果值存在，就对该值执行提供的 mapping 函数调用。

**flatMap(Function mapper)**

如果值存在，就对该值执行提供的mapping 函数调用，返回一个 Optional 类型的值，否则就返回一个空的 Optional 对象。

## 实例
基本类
    
    @Data
    class User {
        String username;
        int age;
        public User(String username) {
            this.username = username;
        }
    }
 使用类
    
     class myTest {
        public static void main(String[] args) {
            myTest test = new myTest();
            //System.out.println(test.getUser2().getUsername());
            // System.out.println(test.getUser4());
            System.out.println(test.getUser5());
        }
        // orElse
        User getUser1()
        {
            User user = null;
            Optional<User> optionalUser = Optional.ofNullable(user);
            return optionalUser.orElse(new User("default"));
        }
        // orElseGEt
        User getUser2()
        {
            User user = null;
            Optional<User> optionalUser = Optional.ofNullable(user);
            return optionalUser.orElseGet(()->getUser1());
        }
        // ifPresent
        User getUser3()
        {
            User user = new User("xiaoming");
            Optional<User> optionalUser = Optional.ofNullable(user);
            optionalUser.ifPresent(System.out::println);
            return optionalUser.orElseGet(()->getUser1());
        }
        // map
        int getUser4()
        {
            User user = new User("xiaoming");
            Optional<User> optionalUser = Optional.ofNullable(user);
            return optionalUser.map(user1 -> user1.getAge()).orElse(1);
        }
        // filter
        int getUser5()
        {
            User user = new User("xiaoming");
            user.setAge(5);
            Optional<User> optionalUser = Optional.ofNullable(user);
            Optional<Integer> a = optionalUser.map(user1 -> user1.getAge()).filter((value)->value>10);
            return a.orElse(10);
        }
     }
