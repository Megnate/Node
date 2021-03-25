### java基础

```java
package test;

public class Test01{
    public static void main(String[] args) {
        //可以通过类名的方式调用静态方法和静态变量
        System.out.println(VTest.country);
        VTest.m1();
        //对于实例变量，需要new创建新的类对象并根据构造方法赋予新的值
        VTest v1 = new VTest(12, "mei");
        VTest v3 = new VTest(13, "nan");
        VTest v2 = new VTest();
        v3.m2();
        v1.m2();
        System.out.println(v1.getName());
        System.out.println(v2.getId());
        Test2 t1 = new Test2(15, "gui");
        t1.m2();
        Test2.m1();
        System.out.println(Test2.country);
        System.out.println(Test2.person);
        VTest t2 = new Test2(15, "meigui");
        //父类对象想调用子类中特有的方法，需要向下转型为子类对象
        if (t2 instanceof VTest){
            //所有的向下转型，都必须要进行instanceof判断
            ((Test2) t2).m3();
        }
        t1.m3();
    }

    static{
        System.out.println("静态代码块中的代码只加载一次，且在main方法之前执行，可以存在多个");
    }
}

class VTest{
    //实例变量
    private int id;
    private String name;
    private int thisId;
    //静态变量，不需要改变，所以不会占用多余的内存
    static String country = "China";
    //常量，无法被修改
    static final String person = "person";
    //构造方法
    public VTest(){

    }
    //有参数的构造方法
    public VTest(int id, String name){
        this.id = id;
        this.name =  name;
    }
    public VTest(int id){
        this.id = id;
    }

    //静态方法，里面内容不会改变
    public static void m1(){
        //this不能写在静态方法中
        //但是静态方法可以new一个新的对象来调用类中的实例变量
        System.out.println("静态方法");
    }
    public void m2(){
        //this指代的是当前对象，无论new创建了多少个对象，都是看当时引用这个方法的对象是谁，就使用什么对象中构造方法的值
        System.out.println(this.name + "是一个好学生");
    }

    public int getThisId() {
        return thisId;
    }
    public void setThisId(int thisId) {
        this.thisId = thisId;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }
}

class Test2 extends VTest{
    public Test2() {
    }

    public Test2(int id, String name) {
        //super不能使用在静态方法中，出现在构造方法的第一行，表示调用父类构造方法
        //构造方法无法继承，但是可以通过super进行调用
        //super也可以访问父类的方法
        super(id, name);
    }

    public Test2(int id){
        //表示去调用携带两个对应数据类型的构造方法
        this(2, "meinan");
    }
    //子类可以继承父类的静态方法，静态变量和常量，常量是无法被修改的

    public void m3(){
        System.out.println("这是子类独有的方法");
        //super可以调用父类中的这个方法，而不是调用在子类中重写的方法
        super.m2();
    }

    @Override
    public String toString() {
        return "Test2{}继承的是VTest{}";
    }

    @Override
    public void m2() {
        super.m2();
        System.out.println("此时继承了父类中的方法，对其进行重写，但是也使用super方法进行了调用");
    }

    @Override
    public int getThisId() {
        return super.getThisId();
    }

    @Override
    public void setThisId(int thisId) {
        super.setThisId(thisId);
    }

    @Override
    public int getId() {
        return super.getId();
    }

    @Override
    public String getName() {
        return super.getName();
    }

    @Override
    public void setId(int id) {
        super.setId(id);
    }

    @Override
    public void setName(String name) {
        super.setName(name);
    }
}
final class T3{
    //这个类无法被继承
}
class T4{
    //此时这个在调用的时候保存的内存地址无法被修改，所以i这个值就不能被修改
    final int i = 10;
    //常量怎么样都无法修改
    final static String name = "mei";

    //这个类可以被继承，但是这个方法不可以被继承
    public final void t(){

    }
    public T4(){
        //这种方法是可以的，但是不建议这样写
        //this.i = 20;
    }
}

abstract class Do{
    //抽象类无法被实例化，可以通过子类继承它来实例化其中的方法。但是可以存在构造方法
    public Do(){
        System.out.println("在子类中通过super进行调用");
    }
    //抽象方法必须出现在抽象类中，但是抽象类中不一定有抽象方法
    public abstract void doSome();
}
class Do2 extends Do{
    //对于多种类，尽量使用抽象的方式
    public Do2(){
        super();
    }
    @Override
    public void doSome() {
        System.out.println("子类如果不是抽象类，继承了抽象类，就必须重写抽象父类中的抽象方法");
    }
}
interface TT{
    //接口中只允许存在常量和抽象类
    final static int i = 10;
    public abstract void Do();
    //这种出现在接口中，所以前面的表明可以省略，但是不推荐
    int Dosome();
    int a = 20;
}
interface Tt extends TT{
    //接口可以继承，并且可以多继承
}
class tt implements TT{
    //类实现了接口，就必须要要重写接口中的方法（抽象）
    //类可以实现多个接口
    @Override
    public void Do() {

    }

    @Override
    public int Dosome() {
        return 0;
    }
}
```

