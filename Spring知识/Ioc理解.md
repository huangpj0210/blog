### Ioc理解

##### 1.什么是IOC？

IoC 全称为 `Inversion of Control`，翻译为 “控制反转”，它还有一个别名为 DI（`Dependency Injection`）,即依赖注入。

1. **谁控制谁**：在传统的开发模式下，我们都是采用直接 new 一个对象的方式来创建对象，也就是说你依赖的对象直接由你自己控制，但是有了 IOC 容器后，则直接由 IoC 容器来控制。所以“谁控制谁”，当然是 IoC 容器控制对象。
2. **控制什么**：控制对象。
3. **为何是反转**：没有 IoC 的时候我们都是在自己对象中主动去创建被依赖的对象，这是正转。但是有了 IoC 后，所依赖的对象直接由 IoC 容器创建后注入到被注入的对象中，依赖的对象由原来的主动获取变成被动接受，所以是反转。
4. **哪些方面反转了**：所依赖对象的获取被反转了。

##### 2.IOC注入的方式?


  在`Spring`中，共有四种方式为`bean`的属性注入值，分别是：

- **set方法注入**
- **构造器注入**
- **静态工厂注入**
- **实例工厂注入**

1. set法注入

在演示前，我们需要准备几个类，我使用下面两个类来进行注入的演示，这两个类分别是`User`和`Car`类：

```java
public class Car {
    // 只包含基本数据类型的属性
    private int speed;
    private double price;
    
    public Car() {
    }
    public Car(int speed, double price) {
        this.speed = speed;
        this.price = price;
    }
    
    public int getSpeed() {
        return speed;
    }
    public void setSpeed(int speed) {
        this.speed = speed;
    }
    public double getPrice() {
        return price;
    }
    public void setPrice(double price) {
        this.price = price;
    }
    @Override
    public String toString() {
        return "Car{" +
                "speed=" + speed +
                ", price=" + price +
                '}';
    }
}

public class User {
	
    private String name;
    private int age;
    // 除了上面两个基本数据类型的属性，User还依赖Car
    private Car car;
    
    public User() {
    }
    public User(String name, int age, Car car) {
        this.name = name;
        this.age = age;
        this.car = car;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Car getCar() {
        return car;
    }
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", car=" + car +
                '}';
    }
}
```

有了上面两个类，我们就可以演示`set`注入了。需要注意一点，如果我们需要使用`set`注入，那么必须要为属性提供`set`方法，`Spring`容器就是通过调用`bean`的`set`方法为属性注入值的。而在`xml`文件中，使用`set`注入的方式就是通过`property`标签，如下所示：

```xml
<!-- 定义car这个bean，id为myCar -->
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 
        为car的属性注入值，因为speed和price都是基本数据类型，所以使用value为属性设置值；
        注意，这里的name为speed和price，不是因为属性名就是speed和price，
        而是set方法分别为setSpeed和setPrice，名称是通过将set删除，然后将第一个字母变小写得出；
    -->
    <property name="speed" value="100"/>
    <property name="price" value="99999.9"/>
</bean>

<!-- 定义user这个bean -->
<bean id="user" class="cn.tewuyiang.pojo.User">
    <property name="name" value="aaa" />
    <property name="age" value="123" />
    <!-- car是引用类型，所以这里使用ref为其注入值，注入的就是上面定义的myCar 
         基本数据类型或Java包装类型使用value，
         而引用类型使用ref，引用另外一个bean的id 
    -->
    <property name="car" ref="myCar" />
</bean>
```

  通过上面的配置，就可以为`Car`和`User`这两个类型的`bean`注入值了。需要注意的是，**property的name属性，填写的不是属性的名称，而是set方法去除set，然后将第一个字符小写后的结果。对于基本数据类型，或者是Java的包装类型（比如String），使用value注入值，而对于引用类型，则使用ref，传入其他bean的id。**接下来我们就可以测试效果了：



```java
@Test
public void test1() {
    ApplicationContext context =
        new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取user这个bean
    User user = context.getBean(User.class);
    // 输出产看结果
    System.out.println(user);
}
```

  由于`user`包含`car`的引用，所以我们直接输出`user`，也能够看到`car`的情况，输入结果如下：



```java
User{name='aaa', age=123, car=Car{speed=100, price=99999.9}}
```



2. 构造器注入

  下面我们来说第二种方式——构造器注入。听名字就可以知道，这种注入值的方式，就是通过调用`bean`所属类的带参构造器为`bean`的属性注入值。这也就意味着，**我们如果需要使用构造器注入，就得为类提供包含参数的构造方法**。构造器注入，实际上有多种匹配属性值的方式，下面我们就来一一列举。我们这里依然使用`2.2`中定义的`Car`和`User`这两个类，测试方法以及类的定义都不需要变，需要改变的仅仅是`xml`配置文件。

**（一）匹配构造器的参数名称**

  我们需要通过`constructor-arg`标签为构造器传入参数值，但是每个`constructor-arg`标签对应哪一个参数值呢？这就有多种方式指定了。第一种就是直接匹配参数名，配置如下：

```xml
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <!-- 通过constructor-arg的name属性，指定构造器参数的名称，为参数赋值 -->
    <constructor-arg name="speed" value="100" />
    <constructor-arg name="price" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <constructor-arg name="name" value="aaa" />
    <constructor-arg name="age" value="123" />
    <!-- 
         和之前一样，基本数据类型或Java包装类型使用value，
         而引用类型使用ref，引用另外一个bean的id 
    -->
    <constructor-arg name="car" ref="myCar" />
</bean>
```

  这样就完成了，测试代码和之前一样，运行结果也一样，我这里就不贴出来了。有人看完之后，可能会觉得这里的配置和`set`注入时的配置几乎一样，除了一个使用`property`，一个使用`constructor-arg`。确实，写法上一样，但是表示的含义却完全不同。**property的name属性，是通过set方法的名称得来；而constructor-arg的name，则是构造器参数的名称**。

**（二）匹配构造器的参数下标**

  上面是通过构造器参数的名称，匹配需要传入的值，那种方式最为直观，而`Spring`还提供另外两种方式匹配参数，这里就来说说通过参数在参数列表中的下标进行匹配的方式。下面的配置，请结合`2.2`节中`User`和`Car`的构造方法一起阅读，配置方式如下：

```xml
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 下标编号从0开始，构造器的第一个参数是speed，为它赋值100 -->
    <constructor-arg index="0" value="100" />
    <!-- 构造器的第二个参数是price，为它赋值99999.9 -->
    <constructor-arg index="1" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 与上面car的配置同理 -->
    <constructor-arg index="0" value="aaa" />
    <constructor-arg index="1" value="123" />
    <constructor-arg index="2" ref="car" />
</bean>
```

上面就是通过参数的下标为构造器的参数赋值，需要注意的是，**参实的下标从0开始**。使用上面的方式配置，若赋值的类型与参数的类型不一致，将会在容器初始化`bean`的时候抛出异常。如果`bean`存在多个参数数量一样的构造器，`Spring`容器会自动找到类型匹配的那个进行调用。比如说，`Car`有如下两个构造器，`Spring`容器将会调用第二个，因为上面的配置中，`index = 1`对应的`value`是`double`类型，与第二个构造器匹配，而第一个不匹配：

```java
public Car(double price, int speed) {
    this.speed = speed;
    this.price = price;
}
// 将使用匹配这个构造器
public Car(int speed, double price) {
    this.speed = speed;
    this.price = price;
}
```

还存在另外一种特殊情况，那就是多个构造器都满足`bean`的配置，此时选择哪一个？假设当前`car`的配置是这样的：

```xml
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 两个下标的value值都是整数 -->
    <constructor-arg index="0" value="100" />
    <constructor-arg index="1" value="999"/>
</bean>
```

假设`Car`还是有上面两个构造器，两个构造器都是一个`int`类型一个`double`类型的参数，只是位置不同。而配置中，指定的两个值都是`int`类型。但是，`int`类型也可以使用`double`类型存储，所以上面两个构造器都是匹配的，此时调用哪一个呢？结论就是调用第二个。自己去尝试就会发现，**若存在多个构造器匹配bean的定义，Spring容器总是使用最后一个满足条件的构造器**。

**（三）匹配构造器的参数类型**

下面说最后一种匹配方式——匹配构造器的参数类型。直接看配置文件吧：

```xml
<bean id="car" class="cn.tewuyiang.pojo.Car">
    <!-- 使用type属性匹配类型，car的构造器包含两个参数，一个是int类型，一个是double类型 -->
    <constructor-arg type="int" value="100" />
    <constructor-arg type="double" value="99999.9"/>
</bean>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 对于引用类型，需要使用限定类名 -->
    <constructor-arg type="java.lang.String" value="aaa" />
    <constructor-arg type="int" value="123" />
    <constructor-arg type="cn.tewuyiang.pojo.Car" ref="car" />
</bean>
```

  上面应该不难理解，直接通过匹配构造器的参数类型，从而选择一个能够完全匹配的构造器，调用这个构造器完成`bean`的创建和属性注入。需要注意的是，上面的配置中，类型并不需要按构造器中声明的顺序编写，`Spring`也能进行匹配。这也就意味着可能出现多个能够匹配的构造器，和上一个例子中一样。比如说，`Car`还是有下面两个构造器：

```java
public Car(double price, int speed) {
    // 输出一句话，看是否调用这个构造器
    System.out.println(111);
    this.speed = speed;
    this.price = price;
}
// 将使用匹配这个构造器
public Car(int speed, double price) {
    // 输出一句话，看是否调用这个构造器
    System.out.println(222);
    this.speed = speed;
    this.price = price;
}
```

  上面两个构造器都是一个`int`，一个`double`类型的参数，都符合xml文件中，`car`这个`bean`的配置。通过测试发现，**Spring容器使用的永远都是最后一个符合条件的构造器**，这和上面通过下标匹配是一致的。**需要说明的一点是，这三种使用构造器注入的方式，可以混用**。

3. 静态工厂注入

  静态工厂注入就是我们编写一个静态的工厂方法，这个工厂方法会返回一个我们需要的值，然后在配置文件中，我们指定使用这个工厂方法创建`bean`。首先我们需要一个静态工厂，如下所示：

```java
public class SimpleFactory {

    /**
     * 静态工厂，返回一个Car的实例对象
     */
    public static Car getCar() {
        return new Car(12345, 5.4321);
    }
}
```

  下面我们需要在`xml`中配置car这个bean，并指定它由工厂方法进行创建。配置如下：

```xml
<!-- 
	注意，这里的配置并不是创建一个SimpleFactory对象，取名为myCar，
    这一句配置的意思是，调用SimpleFactory的getCar方法，创建一个car实例对象，
    将这个car对象取名为myCar。
-->
<bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- name和age使用set注入 -->
    <property name="name" value="aaa"/>
    <property name="age" value="123"/>
    <!-- 将上面配置的car，注入到user的car属性中 -->
    <property name="car" ref="car"/>
</bean>
```

  以上就配置成功了，测试方法以及执行效果如下，注意看`car`的属性值，就是我们在静态工厂中配置的那样，这说明，`Spring`容器确实是使用我们定义的静态工厂方法，创建了`car`这个`bean`：

```java
@Test
public void test1() {
    ApplicationContext context =
        new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取静态工厂创建的car
    Car car = (Car) context.getBean("car");
    // 获取user
    User user = context.getBean(User.class);
    System.out.println(car);
    System.out.println(user);
}
```

  输出如下所示：

```java
Car{speed=12345, price=5.4321}
User{name='aaa', age=123, car=Car{speed=12345, price=5.4321}}
```

4. 实例工厂注入

实例工厂与静态工厂类似，不同的是，静态工厂调用工厂方法不需要先创建工厂类的对象，因为静态方法可以直接通过类调用，所以在上面的配置文件中，并没有声明工厂类的`bean`。但是，实例工厂，需要有一个实例对象，才能调用它的工厂方法。我们先看看实例工厂的定义：

```java
public class SimpleFactory {

    /**
     * 实例工厂方法，返回一个Car的实例对象
     */
    public Car getCar() {
        return new Car(12345, 5.4321);
    }

    /**
     * 实例工厂方法，返回一个String
     */
    public String getName() {
        return "tewuyiang";
    }

    /**
     * 实例工厂方法，返回一个int，在Spring容器中会被包装成Integer
     */
    public int getAge() {
        return 128;
    }
}
```

在上面的工厂类中，共定义了三个工厂方法，分别用来返回`user`所需的`car`，`name`以及`age`，而配置文件如下：

```xml
<!-- 声明实例工厂bean，Spring容器需要先创建一个SimpleFactory对象，才能调用工厂方法 -->
<bean id="factory" class="cn.tewuyiang.factory.SimpleFactory" />

<!-- 
    通过实例工厂的工厂方法，创建三个bean，通过factory-bean指定工厂对象，
    通过factory-method指定需要调用的工厂方法
-->
<bean id="name" factory-bean="factory" factory-method="getName" />
<bean id="age" factory-bean="factory" factory-method="getAge" />
<bean id="car" factory-bean="factory" factory-method="getCar" />

<bean id="user" class="cn.tewuyiang.pojo.User">
    <!-- 将上面通过实例工厂方法创建的bean，注入到user中 -->
    <property name="name" ref="name"/>
    <property name="age" ref="age"/>
    <property name="car" ref="car"/>
</bean>
```

我们尝试从`Spring`容器中取出`name`，`age`，`car`以及`user`，看看它们的值，测试代码如下：

```java
@Test
public void test1() {
    ApplicationContext context =
        new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
    // 获取静态工厂创建的car，name和age这三个bean
    Car car = (Car) context.getBean("car");
    String name = (String) context.getBean("name");
    Integer age = (Integer) context.getBean("age");
    // 获取user这个bean
    User user = context.getBean(User.class);
    System.out.println(car);
    System.out.println(name);
    System.out.println(age);
    System.out.println(user);
}
```

以下就是输出结果，可以看到，我们通过工厂创建的`bean`，都在`Spring`容器中能够获取到：

```java
Car{speed=12345, price=5.4321}
tewuyiang
128
User{name='tewuyiang', age=128, car=Car{speed=12345, price=5.4321}}
```

5. 使用注解注入

假如需要使用注解的方式为`bean`注入属性值，应该这么操作呢？首先，如果`bean`依赖于其他`bean`（比如`User`依赖`Car`），那么我们可以使用`@Autowired`或者`@Resource`这两个注解进行依赖注入，这个大家应该都知道。但是如果要为基本数据类型或者是`Java`的封装类型（比如`String`）赋值呢？这时候可以使用`@Value`注解。这里我就不演示了，感兴趣的可以自行去研究，应该是比`xml`的方式简单多了。

> [1.]: https://www.cnblogs.com/tuyang1129/p/12873492.html	"Spring中bean的四种注入方式"
>
> 

