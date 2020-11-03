# java基础
## 一.Object类中的方法
### 1.equals()
设计原则:
```
1.自反性:自己必须跟自己相等
2.传递性:x=y,y=z,则x=z
3.对称性:x.equals(y)则y.equals(x)
4.一致性:多次调用equal应返回同样的结果
5.null与任何非空不相等
```
一般编码步骤:
```
1.检测是否为同一个引用
if(this == otherObj) return true;

2.检测非空
if(otherObj == null) return false;

3.检测继承关系(即是否为同一个类)
1)若相等关系由父类决定,则
if(! (otherObj instanceof()) ) return false;
2)若相等关系由子类决定,则
if(getClass() != otherObj.getClass()) return false

4.将otherOBj转换成this相同的类型,并进行比较他们的属性是否一致
``` 
## HashCode()
注意事项:
此方法与equals()息息相关,重新定义了equals就必须重新定义HashCode;
原因是:**x.equals(y)==true,则x和y的HashCode就必须相等,即对象相等则hashcode一定相同.**,若不重写,则可能会出现equals相等,但hashcode不相等的错误

## toString()  