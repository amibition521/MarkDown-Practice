## 函数

* 短小

* 只做一件事

* 每个函数一个抽象层级

  * 自顶向下读代码：向下原则

* `Switch`语句

  * ```
    public Money calculatePay(Employee e) throws InvalidEmployeeType{
      switch(e.type){
        case COMMISSIONED:
        	return calculateCommissionedPay(e);
        case HOURLY:
        	return calculateHourPay(e);
        case SALARIED:
        	return calculateSalariedPay(e);
        default:
        	throw new InvalidEmployeeType(e.type);
      }
    }
    ```

  * 优化方案：将`switch`语句放到抽象工厂下，相应方法由Employee接口多态地创建

    ```
    public abstract class Employee {
      public abstract boolean isPayday();
      public abstract Money calculatePay();
      public abstract void deliverPay(Money pay);
    }
    --------------------
    public interface EmployeeFactory() {
    	public Employee makeEmployee(EmployeeRecord r) throw InvalidEmployeeType;
    }
    ------------------------
    public class EmployeeFactoryImpl implements EmployeeFactory {
      public Employee make Employee (EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type){
    		case COMMISSIONED:
    			return new CommissionedEmployee(r);
    		case HOURLY:
    			return new HourlyEmployee(r);
    		case SALARIED:
    			return new SalariedEmployee(r);
    		default:
    			throe new IvalidEmployeeType(r.type);
    	}
      }
    }
    ```

* 使用描述性 的名称

  * 如果每个例程都让你感到深合己意，那就是整洁代码
  * 别害怕长名称    选择描述性的名称能理清模块的设计思路，命名方式要保持一致，使用与模块名一脉相承的短语

* 函数参数

  * 最理想的参数是零

* 无副作用

  * 一个函数只干一件事   时序性耦合

* 分隔指令和询问

  * ```
    public boolean set (String attribute,String value)
    if (set("username","unclebob"))...

    //good code
    if (attributeExits("username")){
      setAttribute("username","unclebob");
      .....
    }
    ```

* 使用异常·try/catch`替代返回错误码

* 别重复自己

  * 重复是软件一切邪恶的根源

* 结构化编程

  * 每个函数都应该有一个入口、一个出口。