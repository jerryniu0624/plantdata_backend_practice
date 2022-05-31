# JAVA设计模式8讲



## 前言

我的java版本是1.8。

设计模式有3种：结构型模式、创建型模式和行为型模式。

第一种中有后文的：1、2、3

第二种中有后文的：4

第三种中有后文的：5、6、7、8



## 官网

https://www.runoob.com/design-pattern/proxy-pattern.html



## 一、适配器模式

在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。适配器就是这么一个把原本不能结合的对象和接口**”杂交“**的方法。

我有一个类

```java
public class Bike {
    public void drive(){
        System.out.println("I am driving a bike.");
    }
}
```

我有一个接口

```java
public interface Vehicle {
    void drive();
}
```

我要结合！（适配器）

```java
public class Adapter implements Vehicle{
    private Bike bike;

    public Adapter(Bike bike){
        this.bike = bike;
    }

    @Override
    public void drive(){
        bike.drive();
    }
}
```

测试一下

```java
public class Test {
    public static void main(String[] args) {
        Bike bike = new Bike();
        Vehicle adapter = new Adapter(bike);
        adapter.drive();
    }
}
```



## 二、代理模式

在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

代理就像中介一样，为其他对象提供一种代理以控制对这个对象的访问。

我这里讲的是jdk动态代理。



我有一个最终对象

```java
public class UserDao implements IUserDao{
    public void save(){
        System.out.println("saved");
    }
}
```

它实现一个接口

```java
public interface IUserDao {
    void save();
}
```

交给jdk动态代理

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyFactory {
    private UserDao userDao;
    public ProxyFactory(UserDao userDao){
        this.userDao = userDao;
    }

    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                userDao.getClass().getClassLoader(),
                userDao.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开始事务2");
                        Object returnValue = method.invoke(userDao,args);
                        System.out.println("提交事务2");
                        return returnValue;
                    }
                }
        );
    }
}
```

测试一下！

```java
public class Test {
    public static void main(String[] args) {
        UserDao userDao = new UserDao();
        System.out.println(userDao.getClass());
        IUserDao proxy = (IUserDao) new ProxyFactory(userDao).getProxyInstance();
        System.out.println(proxy.getClass());
        proxy.save();
    }
}
```



## 三、组合模式

它在我们树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以像处理简单元素一样来处理复杂元素，从而使得客户程序与复杂元素的内部结构解耦。

组合模式将对象组合成树形结构以表示"部分-整体"的层次结构。它使得用户对单个对象和组合对象的使用具有一致性。像极了学c语言时的二叉树。



我有一棵树

```java
import java.util.Enumeration;
import java.util.Vector;

public class Treenode {
    private String name;
    private Treenode parent;
    private Vector<Treenode> chiledren = new Vector<Treenode>();

    public Treenode(String name){
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Treenode getParent() {
        return parent;
    }

    public void setParent(Treenode parent) {
        this.parent = parent;
    }

    public void add(Treenode treenode){
        chiledren.add(treenode);
    }

    public void remove(Treenode treenode){
        chiledren.remove(treenode);
    }

    public Enumeration<Treenode> getChildren(){
        return chiledren.elements();
    }

    public void printChildren(){
        for(Treenode treenode : chiledren){
            System.out.println(treenode.toString());
            treenode.printChildren();
        }
    }

    @Override
    public String toString() {
        return "Treenode{" +
                "name='" + name + '\'' +
                ", parent=" + parent +
                ", chiledren=" + chiledren +
                '}';
    }
}
```

测试一下

```java
public class Tree {

    Treenode root = null;

    public Tree(String name){
        root = new Treenode(name);
    }

    @Override
    public String toString() {
        return "Tree{" +
                "root=" + root +
                '}';
    }

    public static void main(String[] args) {
        Tree tree = new Tree("A");
        Treenode nodeA = new Treenode("B");
        Treenode nodeB = new Treenode("C");
        nodeB.add(nodeA);
        tree.root.add(nodeB);
        System.out.println(tree);
    }
}
```



## 四、抽象工厂模式

主要解决接口选择的问题。

它提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。



我有一个抽象工厂

```java
public interface Factory {
    public Fruit getFruit();
}
```

我有两个具体工厂

```java
public class AppleFactory implements Factory{
    @Override
    public Fruit getFruit(){
        return new Apple();
    }
}
```

```java
public class BananaFactory implements Factory{
    @Override
    public Fruit getFruit(){
        return new Banana();
    }
}
```

它们分别身缠苹果与香蕉

```java
public class Apple implements Fruit{
    public Apple(){
        System.out.println("I am an Apple.");
    }
    @Override
    public void eat() {
        System.out.println("I am eating an Apple.");
    }
}
```

```java
public class Banana implements Fruit{
    public Banana(){
        System.out.println("I am an Banana.");
    }
    @Override
    public void eat() {
        System.out.println("I am eating an Banana.");
    }
}
```

这些水果实现了这个接口

```java
public interface Fruit {
    void eat();
}
```

测试一下

```java
public class Test {
    public static void main(String[] args) {
        new AppleFactory().getFruit().eat();
        new BananaFactory().getFruit().eat();
    }
}
```

用抽象工厂模式与普通工厂模式的区别

抽象了一层工厂。这样，如果需要新增一个种植樱桃的工厂，不用再原有的代码中改，只要重新写个实现工厂接口的具体工厂即可。



## 五、模板模式

一些方法通用，却在每一个子类都重新写了这一方法。

它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。



我有一个模板

```java
public abstract class DoDishTemplate {
    protected void dodish(){
        this.preparation();
        this.doing();
        this.carryDishes();
    }
    public abstract void preparation();
    public abstract void doing();
    public abstract void carryDishes();
}
```



我有一个模板的具体实现（子类）

```java
public class EggsWithTomatoes extends DoDishTemplate{
    @Override
    public void preparation(){
        System.out.println("准备番茄炒蛋");
    }
    @Override
    public void doing(){
        System.out.println("炒番茄炒蛋");
    }
    @Override
    public void carryDishes(){
        System.out.println("上番茄炒蛋");
    }
}
```

测试一下

```java
public class Test {
    public static void main(String[] args) {
        DoDishTemplate eggsWithTomatoes = new EggsWithTomatoes();
        eggsWithTomatoes.dodish();
    }
}
```



## 六、观察者模式

一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

微信上”订阅-发布“就适用于这样的模式。



我有一个观察者（用户端）

```java
public interface Observer {
    public void update(String message);
}
```



我有一个被观察者（服务端）

```java
public interface Observable {
    public void registerObserver(Observer observer);
    public void removeObserver(Observer observer);
    public void notifyObserver();
}
```

当服务端通知观察者时，观察者更新状态。



观察者的实现

```java
public class WechatUser implements Observer{

    private String name;
    private String message;

    public WechatUser(String name){
        this.name = name;
    }

    @Override
    public void update(String message){
        this.message = message;
        read();
    }

    public void read(){
        System.out.println(message+name+"正在读书");
    }
}
```

被观察者的实现

```java
import java.util.ArrayList;
import java.util.List;


public class WechatServer implements Observable{

    private List<Observer> list = new ArrayList<Observer>();
    private String message;

    @Override
    public void registerObserver(Observer observer){
        list.add(observer);
    }

    @Override
    public void removeObserver(Observer observer){
        if(!list.isEmpty())
            list.remove(observer);
    }

    @Override
    public void notifyObserver(){
        for(int i = 0 ;i < list.size(); i++){
            Observer observer = list.get(i);
            observer.update(message);
        }
    }

    public void setMessage(String message){
        this.message = message;
        notifyObserver();
    }
}
```



## 七、策略模式

与模板模式差不多，只不过少了个模板。

在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。

它定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。



我有大一统的算法接口

```java
public class Context {
    Strategy strategy;
    public Context(Strategy strategy){
        this.strategy = strategy;
    }
    public void contextInterface(){
        strategy.algorithmInterface();
    }
}
```

它是一个承上启下的类，避免上层直接调用具体算法



我有两种算法

```java
public class ConcreteStrategyA extends Strategy{
    @Override
    public void algorithmInterface(){
        System.out.println("算法A实现");
    }
}
```

```java
public class ConcreteStrategyB extends Strategy{
    @Override
    public void algorithmInterface(){
        System.out.println("算法B实现");
    }
}
```



他们实现了算法接口

```java
public abstract class Strategy {
    public abstract void algorithmInterface();
}
```



测试一下

```java
public class Test {
    public static void main(String[] args) {
        Context context;
        context = new Context(new ConcreteStrategyA());
        context.contextInterface();
        context = new Context(new ConcreteStrategyB());
        context.contextInterface();
    }
}
```



## 八、访问者模式

稳定的数据结构和易变的操作耦合问题。

它主要将数据结构与数据操作分离。



我有一个结点接口

```java
public abstract class Node {
    /**
     * 接受操作
     */
    public abstract void accept(Visitor visitor);
}
```

两个相应的具体实现

```java
public class NodeA extends Node{
    /**
     * 接受操作
     */
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    /**
     * NodeA特有的方法
     */
    public String operationA(){
        return "NodeA";
    }

}
```

```java
public class NodeB extends Node{
    /**
     * 接受方法
     */
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    /**
     * NodeB特有的方法
     */
    public String operationB(){
        return "NodeB";
    }
}
```



我有一个访问者接口

```java
public interface Visitor {
    /**
     * 对应于NodeA的访问操作
     */
    public void visit(NodeA node);
    /**
     * 对应于NodeB的访问操作
     */
    public void visit(NodeB node);
}
```

两个相应的具体实现

```java
public class VisitorA implements Visitor {
    /**
     * 对应于NodeA的访问操作
     */
    @Override
    public void visit(NodeA node) {
        System.out.println(node.operationA());
    }
    /**
     * 对应于NodeB的访问操作
     */
    @Override
    public void visit(NodeB node) {
        System.out.println(node.operationB());
    }

}
```

```java
public class VisitorB implements Visitor {
    /**
     * 对应于NodeA的访问操作
     */
    @Override
    public void visit(NodeA node) {
        System.out.println(node.operationA());
    }
    /**
     * 对应于NodeB的访问操作
     */
    @Override
    public void visit(NodeB node) {
        System.out.println(node.operationB());
    }

}
```



我有一个数据结构，里面有根节点。

```java
import java.util.ArrayList;
import java.util.List;

public class ObjectStructure {

    private List<Node> nodes = new ArrayList<Node>();

    /**
     * 执行方法操作
     */
    public void action(Visitor visitor){

        for(Node node : nodes)
        {
            node.accept(visitor);
        }

    }
    /**
     * 添加一个新元素
     */
    public void add(Node node){
        nodes.add(node);
    }
}
```

测试一下

```java
public class Client {

    public static void main(String[] args) {
        //创建一个结构对象
        ObjectStructure os = new ObjectStructure();
        //给结构增加一个节点
        os.add(new NodeA());
        //给结构增加一个节点
        os.add(new NodeB());
        //创建一个访问者
        Visitor visitor = new VisitorA();
        os.action(visitor);
    }

}
```



**适配器模式与装饰模式的区别**

前者对象的接口改变了，后者没有。