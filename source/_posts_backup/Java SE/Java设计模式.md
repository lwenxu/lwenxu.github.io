---
title: GOF23 常用设计模式
date: 2018-08-06 08:25:29
tags: Java
categories:
    -Java
---



# GOF23 常用设计模式

## 一、单例设计模式

### 1.饿汉式

```
public class GOF23 {
    // 饿汉式
    private static final List list = new ArrayList();

    public static List getInstance() {
        return list;
    }
}
```

### 2.懒汉式

```
    // 懒汉式
    // 使用双检锁，并且由于 jvm 指令重排序我们需要使用 volatile 关键字抑制指令重排序
public class GOF{
    private static volatile List list1;
    public static List getInstanceLazy() {
        synchronized (Object.class) {
            if (list1 == null) {
                synchronized (Object.class){
                    list1 = new ArrayList();
                    return list;
                }
            }else {
                return list;
            }
        }
    }
}
```

### 3.静态内部类

```
public class GOF{
     // 静态内部类
    public static class SingletonClass{
        private static final List list2 = new ArrayList();
    }
    // 首先要去加载这个静态内部类，然后这个时候会调用 cinit 方法这个是天然的线程安全的，然后返回这个对象即可  可以延时加载
    public List getStaticInstance(){
        return SingletonClass.list2;
    }
}
```

### 4.枚举

```
public class GOF23 {
    // 采用枚举的方式  不能延时加载  由 jvm 底层完成的
    public enum SingleInstanceEnum{
        INSTANCE_ENUM
    }
}
```

## 二、工厂设计模式

工厂设计模式的好处就是把对象的创建和使用分开。在软公众需要遵循的三个原则：开闭原则（对修改关闭，对拓展开放）、依赖翻转（面向接口依赖而不是类的强依赖）、迪米特法则（和朋友通讯而不是陌生人）。

### 1.简单工厂

一般也称之为静态工厂，因为他的方法是一个静态的方法，并且必须要有一个父接口去容纳这些被创建的对象。

```
public interface Car {
    public void run();
}

public class Audi implements Car {
    @Override
    public void run() {
        System.out.println("audi run");
    }
}

public class Daben implements Car {
    @Override
    public void run() {
        System.out.println("daben run");
    }
}

public class CarFactory {
    public static Car createCar(String type) {
        if (type.equals("daben")) {
            return new Daben();
        } else if (type.equals("audi")) {
            return new Audi();
        }else {
            return null;
        }
    }
}

public class CarClient {
    public static void main(String[] args) {
        Daben daben = (Daben) CarFactory.createCar("daben");
        daben.run();
    }
}
```

### 2.工厂方法

这个比上面好一点的就是我们不用直接修改原来的代码而是可以面向拓展而不是修改就是开闭原则是满足的。其实我们在项目中主要使用的是简单工厂而不是这个，因为这个的类文件剧增。

```
public interface CarFactory {
    public Car createFactory();
}

public class BenChiFactory implements CarFactory {
    @Override
    public Car createFactory() {
        return new Daben();
    }
}

public class AudiFactory implements CarFactory {
    @Override
    public Car createFactory() {
        return new Audi();
    }
}
```

### 3.抽象工厂

抽象工厂其实和上面的两个方式区别很大，主要是因为上面两个的维度是创建产品而这个地方的维度是创建一个产品组。也就是比如我们有一个车的工厂，其实这个工厂是创建很多车的零部件而不是一个车，那么我们有多个车的工厂的目的就是为了创建不同层次的车零件。比如下面这个例子，就是他的维度是横向的一个维度。

```
// 座椅类型
public interface Seat {
    public void desc();
}

class LuxurySeat implements Seat {

    @Override
    public void desc() {
        System.out.println("high quality seat");
    }
}

class LowSeat implements Seat {

    @Override
    public void desc() {
        System.out.println("low seat");
    }
}

// 引擎类型
public interface Engine {
    public void run();
}

class LuxuryEngine implements Engine {

    @Override
    public void run() {
        System.out.println("high speed engine");
    }
}

class LowEngine implements Engine {

    @Override
    public void run() {
        System.out.println("low engine");
    }
}

// 车的抽象工厂
public interface AbstractCarFactory {
    public Seat createSeat();
    public Engine creaateEngine();
}

// 两个工厂的实现，分别是高级车工厂，低级车工厂
public class LuxuryCarFactory implements AbstractCarFactory {
    @Override
    public Seat createSeat() {
        return new LuxurySeat();
    }

    @Override
    public Engine creaateEngine() {
        return new LuxuryEngine();
    }
}

public class LowCarFactory implements AbstractCarFactory{
    @Override
    public Seat createSeat() {
        return new LowSeat();
    }

    @Override
    public Engine creaateEngine() {
        return new LowEngine();
    }
}
```

## 三、建造者模式

建造者模式分为两部分，第一个部分其实就是工厂模式创建对象，然后另外一部分就是组装创建整个大对象。

```
// 创建工厂
public interface BuilderFactory {
    Engine createEngine();
    Seat createSeat();
}

public class AirShipBuilderFactory implements BuilderFactory {

    @Override
    public Engine createEngine() {
        return new Engine() {
            @Override
            public void run() {

            }
        };
    }

    @Override
    public Seat createSeat() {
        return new Seat() {
            @Override
            public void desc() {


            }
        };
    }
}
// 组装工厂
public class AirDirectFactory {

    private BuilderFactory builderFactory;

    public AirShip buildAirShip() {
        Engine engine = builderFactory.createEngine();
        Seat seat = builderFactory.createSeat();
        return new AirShip(engine, seat);
    }

    public AirDirectFactory(BuilderFactory builderFactory) {
        this.builderFactory = builderFactory;
    }
}
```

## 四、原型模式

可以称之为克隆或者拷贝，然后在拷贝的对象上修改就可以产生新对象，其实js 的原型就是这么干的。

这里需要注意一下这里的一个深拷贝的问题，如果没有作深拷贝而是直接的做了一个 super 的调用可能导致状态关联问题。

```
@NoArgsConstructor
@AllArgsConstructor
@Builder(toBuilder = true)
@Data
@Accessors(chain = true,fluent = true)
public class Sheep implements Cloneable{
    private String name;
    private Date birthday;

    /**
     * 注意在克隆的时候需要进行深拷贝负责就会出现错误结果
     * @return
     * @throws CloneNotSupportedException
     */
    @Override
    protected Object clone() throws CloneNotSupportedException {

        Sheep clone = (Sheep) super.clone();
        clone.birthday = (Date) this.birthday.clone();
        return clone;
    }
}

public class PrototypeClient {
    public static void main(String[] args) throws CloneNotSupportedException {
        Sheep sheep1 = new Sheep("A", new Date());
        Sheep sheep2 = (Sheep) sheep1.clone();
        sheep2.birthday(new Date()).name("sss");
        System.out.println(sheep2);
    }
}
```

**另外我们除了使用上面的克隆方法我们还可以使用序列化的方式直接生成一个一模一样的对象出来。具体来说就是我们可以用输入输出流来生成对象**

## 五、适配器模式

将一个接口转化成另外一个接口，从而使两个原来不能一起工作的组件一起工作。

我们可以想像一下一个 usb 转 hdmi ，那么就有一个目标方和一个被适配方。也就是两个接口再有一个适配器就完成了。

```
// 目标方的接口
public interface Target {
    void use();
}
// 被适配方 其实最好是一个接口这里直接写成了一个类
public class BeAdapter {
    public void hello(){
        System.out.println("say hello");
    }
}

// 适配器 实现了目标方，组合了被适配方
@Data
@AllArgsConstructor
public class AdapterDemo implements Target {
    // 其实这里可以进一步抽象 也就是所有的被适配对象另起一个接口即可
    BeAdapter beAdapter;

    public void use() {
        beAdapter.hello();
    }
}

public class AdapterClient {
    public static void test(Target target){
        target.use();
    }

    public static void main(String[] args) {
        AdapterDemo adapterDemo = new AdapterDemo(new BeAdapter());
        test(adapterDemo);
    }
}
```

## 六、代理模式

### 1.静态代理

这种代理模式就是在代理对象上保留一个被代理对象的指针，然后有些事情直接调用被代理对象的方法完成，并且可以做一些前置或者后置的操作。

### 2.动态代理

#### 1.JDK 代理

其实关键部分就两个：第一个就是一个处理器，所有的对代理的请求的方法都走到了处理器。另外一个就是生成代理对象过程。

处理器

```
@AllArgsConstructor
public class SimpleHandler implements InvocationHandler {
    RealObject real;
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("heihei");
        return (Object) method.invoke(real, args);
    }
}
```

被代理对象

```
public interface ObjectInterface {
    public void hello();
}

public class RealObject implements ObjectInterface {
    public void hello(){
        System.out.println("hello");
    }
}
```

生成代理对象的过程

```
public class JdkClient {
    public static void main(String[] args) {
        RealObject realObject = new RealObject();
        SimpleHandler simpleHandler = new SimpleHandler(realObject);
        ObjectInterface proxyInstance = (ObjectInterface) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{ObjectInterface.class}, simpleHandler);
        proxyInstance.hello();
    }
}
```

## 七、桥接模式

桥接模式就是为了处理两个维度的事情，比如说我们定义一个电脑类，那么我们如果需要售卖笔记本、平板、台式机而这些东西又有不同的品牌那么我们就需要建立很多的类在电脑这个类下面。但是我们采用桥接的话我们抽出一个接口这个接口代表了品牌，然后有一个抽象类（电脑类型）包含了品牌的引用，我们只需要在不同的电脑类型下面建立各个类型，在创建一个品牌类型的电脑的时候直接 new 出来就行了。

```
// 品牌
public interface Brand {

}
class Dell implements Brand {

}
class Levon implements Brand {

}

// 电脑的类型
@AllArgsConstructor
public abstract class AbstractComputer {
    private Brand brand;
    public void sell(){
        System.out.println("sell...");
    }
}

class DesktopComp extends AbstractComputer {
    public DesktopComp(Brand brand) {
        super(brand);
    }
}

class PadComp extends AbstractComputer {
    public PadComp(Brand brand) {
        super(brand);
    }
}
//使用
public class BridgeClient {
    public static void main(String[] args) {
        DesktopComp levonDesktopComp = new DesktopComp(new Levon());
    }
}
```

## 八、策略模式

策略模式就是将一个方法传入到一个地方，简单点来说就是传入一个回调方法，真实是使用的时候看我们传递进去的那个方法作用。

## 九、模板设计模式

在上层的抽象类中放一些抽象方法先调用上，子类在继承的时候实现这些类就行了。

## 10、观察者模式

在对一些变量修改或者方法调用的时候胡触发观察者的方法调用或者状态的转变，其实就相当于埋点。