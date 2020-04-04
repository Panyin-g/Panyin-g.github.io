---
layout:     post
title:      "Java常见设计模式"
subtitle:   " \"Hello World, Hello Blog\""
date:       2018-12-02 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - 设计模式
---

> “陪伴是最长情的告白！”


## 一、设计模式概论
  
  设计模式分为三种类型，共23种：
  创建型模式：单例模式，抽象工厂模式，建造者模式，工厂模式，原型模式。
  结构型模式：适配器模式，桥接模式，装饰者模式，组合模式，外观模式，享元模式，代理模式。
  行为型模式：模版方法模式，命令模式，迭代器模式，观察者模式，中介者模式，备忘录模式，解释器模式，状态模式，策略模式，职责链模式(责任链模式)，访问者模式。

## 二、单例模式
  单例模式主要是为了避免因为创建了多个实例造成资源的浪费，且多个实例由于多次调用容易导致结果出现错误，而使用单例模式能够保证整个应用中有且只有一个实例。
  简单点说，就是一个应用程序中，某个类的实例对象只有一个，你没有办法去new，因为构造器是被private修饰的，一般通过getInstance()的方法来获取它们的实例。getInstance()的返回值是一个对象的引用，并不是一个新的实例，所以不要错误的理解成多个对象。

  1、定义：只需要三步就可以保证对象的唯一性
    (1) 不允许其他程序用new对象
    (2) 在该类中创建对象
    (3) 对外提供一个可以让其他程序获取该对象的方法
  对比定义：
    (1) 私有化该类的构造函数
    (2) 通过new在本类中创建一个本类对象
    (3) 定义一个公有的方法，将在该类中所创建的对象返回

  2、饿汉式[可用]
    ```
    public class Singleton {  
        private static Singleton instance = new Singleton();  
        private Singleton (){}  
        public static Singleton getInstance() {  
          return instance;  
        }  
    }
    ```

  3、懒汉式（线程安全，去掉synchronized关键字为线程不安全,推荐用）
    ```
    public class Singleton {  
        private static Singleton instance;  
        private Singleton (){}  
        public static synchronized Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
        }  
    }
    ```
    懒汉式双重校验锁
    ```
    public class Singleton {  
        private volatile static Singleton singleton;  
        private Singleton (){}  
        public static Singleton getSingleton() {  
        if (singleton == null) {  
            synchronized (Singleton.class) {  
            if (singleton == null) {  
                singleton = new Singleton();  
            }  
            }  
        }  
        return singleton;  
        }  
    }
    ```

  4、静态内部类(推荐用)
    ```
    public class Singleton {  
        private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
        }  
        private Singleton (){}  
        public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE;  
        }  
    }
    ```

  5、枚举(推荐用)
    ```
    public enum Singleton {  
        INSTANCE;  
        public void whateverMethod() {  
        }  
    }
    ```
    这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，可谓是很坚强的壁垒啊，不过，个人认为由于1.5中才加入enum特性，用这种方式写不免让人感觉生疏。

  静态内部类写法和饿汉式写法，这两种写法能够应付绝大多数情况了。

## 三、工厂模式
  1、静态工厂模式
    项目中的辅助类，类+静态方法。

  2、简单工厂模式
    通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。
    ```
    public RoujiaMo creatRoujiaMo(String type) {
      RoujiaMo roujiaMo = null;
      switch (type) {
          case "Suan":
              roujiaMo = new ZSuanRoujiaMo();
              break;
          case "La":
              roujiaMo = new ZLaRoujiaMo();
              break;
          case "Tian":
              roujiaMo = new ZTianRoujiaMo();
              break;
          default:// 默认为酸肉夹馍
              roujiaMo = new ZSuanRoujiaMo();
              break;
      }
      return roujiaMo;
    }
    ```
  3、工厂方法模式
    定义一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法模式把类实例化的过程推迟到子类。有四个角色，抽象工厂模式，具体工厂模式，抽象产品模式，具体产品模式。不再是由一个工厂类去实例化具体的产品，而是由抽象工厂的子类去实例化产品。
    ```
    // 抽象产品角色
    public interface Moveable {
        void run();
    }
     
    // 具体产品角色
    public class Plane implements Moveable {
        @Override
        public void run() {
            System.out.println("plane....");
        }
    }
     
    public class Broom implements Moveable {
        @Override
        public void run() {
            System.out.println("broom.....");
        }
    }
     
    // 抽象工厂
    public abstract class VehicleFactory {
        abstract Moveable create();
    }
     
    // 具体工厂
    public class PlaneFactory extends VehicleFactory {
        public Moveable create() {
            return new Plane();
        }
    }
     
    public class BroomFactory extends VehicleFactory {
        public Moveable create() {
            return new Broom();
        }
    }
     
    // 测试类
    public class Test {
        public static void main(String[] args) {
            VehicleFactory factory = new BroomFactory();
            Moveable m = factory.create();
            m.run();
        }
    }
    ```

  4、抽象工厂模式
    提供一个接口，用于创建相关的或依赖对象的家族，而不需要明确指定具体类。与工厂方法模式不同的是，工厂方法模式中的工厂只生产单一的产品，而抽象工厂模式中的工厂生产多个产品。
    ```
    /抽象工厂类
    public abstract class AbstractFactory {
        public abstract Vehicle createVehicle();
        public abstract Weapon createWeapon();
        public abstract Food createFood();
    }
    //具体工厂类，其中Food,Vehicle，Weapon是抽象类，
    public class DefaultFactory extends AbstractFactory{
        @Override
        public Food createFood() {
            return new Apple();
        }
        @Override
        public Vehicle createVehicle() {
            return new Car();
        }
        @Override
        public Weapon createWeapon() {
            return new AK47();
        }
    }
    //测试类
    public class Test {
        public static void main(String[] args) {
            AbstractFactory f = new DefaultFactory();
            Vehicle v = f.createVehicle();
            v.run();
            Weapon w = f.createWeapon();
            w.shoot();
            Food a = f.createFood();
            a.printName();
        }
    }
    ```

## 四、适配器模式
  将一个类的接口转换成客户期望的另一个接口，适配器让原本接口不兼容的类可以相互合作。这个定义还好，说适配器的功能就是把一个接口转成另一个接口。
  ```
  public class Test {
      public static void main(String[] args) {
          Phone phone = new Phone();
          VoltageAdapter adapter = new VoltageAdapter();
          phone.setAdapter(adapter);
          phone.charge();
      }
  }
   
  // 手机类
  class Phone {
   
      public static final int V = 220;// 正常电压220v，是一个常量
   
      private VoltageAdapter adapter;
   
      // 充电
      public void charge() {
          adapter.changeVoltage();
      }
   
      public void setAdapter(VoltageAdapter adapter) {
          this.adapter = adapter;
      }
  }
   
  // 变压器
  class VoltageAdapter {
      // 改变电压的功能
      public void changeVoltage() {
          System.out.println("正在充电...");
          System.out.println("原始电压：" + Phone.V + "V");
          System.out.println("经过变压器转换之后的电压:" + (Phone.V - 200) + "V");
      }
  }
  ```

## 五、装饰者模式
  若要扩展功能，装饰者提供了比集成更有弹性的替代方案，动态地将责任附加到对象上。
  当我们设计好了一个类，我们需要给这个类添加一些辅助的功能，并且不希望改变这个类的代码，这时候就是装饰者模式大展雄威的时候了。这里还体现了一个原则：类应该对扩展开放，对修改关闭。
  对已有的业务逻辑进一步的封装，使其增加额外的功能，如Java中的IO流就使用了装饰者模式，用户在使用的时候，可以任意组装，达到自己想要的效果。 举个栗子，我想吃三明治，首先我需要一根大大的香肠，我喜欢吃奶油，在香肠上面加一点奶油，再放一点蔬菜，最后再用两片面包夹一下，很丰盛的一顿午饭，营养又健康。（ps：不知道上海哪里有卖好吃的三明治的，求推荐～）那我们应该怎么来写代码呢？ 首先，我们需要写一个Food类，让其他所有食物都来继承这个类，看代码：
  ```
  public class Food {
 
      private String food_name;
   
      public Food() {
      }
   
      public Food(String food_name) {
          this.food_name = food_name;
      }
   
      public String make() {
          return food_name;
      };
  }
  //面包类
  public class Bread extends Food {
   
      private Food basic_food;
   
      public Bread(Food basic_food) {
          this.basic_food = basic_food;
      }
   
      public String make() {
          return basic_food.make()+"+面包";
      }
  }
   
  //奶油类
  public class Cream extends Food {
   
      private Food basic_food;
   
      public Cream(Food basic_food) {
          this.basic_food = basic_food;
      }
   
      public String make() {
          return basic_food.make()+"+奶油";
      }
  }
   
  //蔬菜类
  public class Vegetable extends Food {
   
      private Food basic_food;
   
      public Vegetable(Food basic_food) {
          this.basic_food = basic_food;
      }
   
      public String make() {
          return basic_food.make()+"+蔬菜";
      }
   
  }
  public class Test {
      public static void main(String[] args) {
          Food food = new Bread(new Vegetable(new Cream(new Food("香肠"))));
          System.out.println(food.make());
      }
  }
  ```
  运行结果：“香肠+奶油+蔬菜+面包”

## 六、观察者模式
  定义了对象之间的一对多的依赖，这样一来，当一个对象改变时，它的所有的依赖者都会收到通知并自动更新。
  ```
  public interface Person {
      //老王和老李通过这个接口可以接收到小美发过来的消息
      void getMessage(String s);
  }
  public class LaoWang implements Person {
 
      private String name = "老王";
   
      public LaoWang() {
      }
   
      @Override
      public void getMessage(String s) {
          System.out.println(name + "接到了小美打过来的电话，电话内容是：" + s);
      }
   
  }
   
  public class LaoLi implements Person {
   
      private String name = "老李";
   
      public LaoLi() {
      }
   
      @Override
      public void getMessage(String s) {
          System.out.println(name + "接到了小美打过来的电话，电话内容是：->" + s);
      }
   
  }
  public class XiaoMei {
      List<Person> list = new ArrayList<Person>();
       public XiaoMei(){
       }
   
       public void addPerson(Person person){
           list.add(person);
       }
   
       //遍历list，把自己的通知发送给所有暗恋自己的人
       public void notifyPerson() {
           for(Person person:list){
               person.getMessage("今天家里就我一个人，你们过来吧，谁先过来谁就能得到我!");
           }
       }
  }
  public class Test {
      public static void main(String[] args) {
   
          XiaoMei xiao_mei = new XiaoMei();
          LaoWang lao_wang = new LaoWang();
          LaoLi lao_li = new LaoLi();
   
          //老王和老李在小美那里都注册了一下
          xiao_mei.addPerson(lao_wang);
          xiao_mei.addPerson(lao_li);
   
          //小美向老王和老李发送通知
          xiao_mei.notifyPerson();
      }
  }
  ```
  测试结果：“老王接到了小美打过来的电话，电话内容是：->今天家里就我一个人，你们过来吧，谁先过来谁就能得到我!
        老李接到了小美打过来的电话，电话内容是：->今天家里就我一个人，你们过来吧，谁先过来谁就能得到我!”

---

## 后记

如果你恰好逛到了这里，希望这篇文章对你有帮助。

—— Panying 后记于 2018.12