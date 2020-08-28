# java隐藏域，域的隐藏


通过继承，子类可以把父类中所有的域和方法都继承下来成为自身的域和方法，但是子类也可以根据需要定义与继承自父类的同名的域，则在子类中只能访问到子类中定义的域而不能访问到继承自父类的同名域，这种情况称为域的隐藏。子类中的方法不能访问父类中被隐藏的域。
域隐藏举例

```java
class A {
 int val;
 public void setVarA(int v){
  val=v;
  //设置域值
 }
 public int getVarA(){
  return val;
  //获得域值
 }
}
class B extends A{
 private long val;
 public void setVarB(long v) {
  val=v;
  //设置域值
 }
 public long getVarB(){
  return val;
 }
}
public class Example20{
 public static void main(String[] args) {
  B b=new B();//创建对象b，b中有两个同名域var
  b.setVarA(1234567890);
  //调用继承自父类的方法设置继承自父类的var
  b.setVarB(12345678900L);
  //调用本身定义的方法设置本身定义的var
  System.out.println(b.getVarA());//输出继承自父类的var
  System.out.println(b.getVarB());//输出继承自子类的var
 }
}
```

————————————————
版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
本文链接：https://blog.csdn.net/LiAphrodite/article/details/99591797