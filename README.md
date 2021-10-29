# sensitive-util

# 简介

`sensitive-util` Java脱敏工具类。

1. 基于注解进行脱敏，也支持非注解形式脱敏(非注解方式可以对Map对象进行脱敏)。
2. 支持复杂对象脱敏，利用反射深度解析对象。
3. 可以很方便的新增脱敏字段类型。（在`FieldType`中新增枚举值）
4. 可以依据业务配置灵活控制脱敏。比如：每个用户有自己的脱敏字段配置，需要依据客户的配置，灵活脱敏

# 快速开始
1. 将此项目代码copy到自己的项目中。
2. 开始使用，工具类入口为`SensitiveUtil`类。具体使用请参考示例代码

# 简单示例
```java
@Data
@ToString
class Person{
    private String name;
    @Sensitive(FieldType.MOBILE)
    private String mobile;
    @Sensitive(FieldType.ADDRESS)
    private String address;
    @Sensitive(FieldType.PASSWORD)
    private String password;
}

public class Test{
    public static void main(String[] args) {
        Person person = new Person();
        person.setName("张三");
        person.setMobile("18019295001");
        person.setPassword("PA23235454");
        person.setAddress("上海市松江区佘山镇");
        // 执行脱敏方法
        SensitiveUtil.apply(person);
        System.out.println(person);
    }
}
```
输出结果：`Person{name='张三', mobile='180****5001', address='上海市松江区***', password='**********'}`


# 更多示例

## 复杂对象脱敏演示
张三有个朋友叫李四。用`SensitiveUtil.apply(zhangsan);`脱敏张三的信息时。
李四这个朋友作为一个List<Person>集合存在与张三这个对象中，也会被脱敏。
`SensitiveUtil`工具类使用反射深度扫描`zhangsan`这个对象中一切被`@Sensitive`注解的字段。

```java
public class 复杂对象脱敏演示 {
    public static void main(String[] args) throws Exception {
        Person zhangsan = new Person();
        zhangsan.setName("张三");
        zhangsan.setMobile("18019295001");
        zhangsan.setPassword("PA23235454");
        zhangsan.setAddress("上海市松江区佘山镇");

        Person lisi = new Person();
        lisi.setName("李四");
        lisi.setMobile("18018732893");
        lisi.setPassword("HAHAHAHAHA");
        lisi.setAddress("上海市嘉定区南翔镇");

        zhangsan.setFriends(Arrays.asList(lisi));
        SensitiveUtil.apply(zhangsan);
        System.out.println(zhangsan);
    }

    @Data
    @ToString
    static class Person {
        private String name;
        @Sensitive(FieldType.MOBILE)
        private String mobile;
        @Sensitive(FieldType.ADDRESS)
        private String address;
        @Sensitive(FieldType.PASSWORD)
        private String password;

        private List<Person> friends;
    }
}

```
`Person{name='张三', mobile='180****5001', address='上海市松江区***', password='**********', friends=[Person{name='李四', mobile='180****2893', address='上海市嘉定区***', password='**********', friends=null}]}`

## 非注解方式脱敏

如果要脱敏的对象中有Map类型，或者要脱敏的对象处于一个不可更改的状态，无法添加注解。
那么可以非注解脱敏的方式就可以发挥作用了。

```java
public class 非注解方式脱敏 {

    public static void main(String[] args) throws Exception {
        Person person = new Person();
        person.setName("洛空");
        person.setMobile("18017648954");
        person.setAddress("杭州市余杭区知北路128号");

        Map<String, Object> params = new HashMap<>();
        params.put("bankcard", "62226043567892341234");
        params.put("idcard", "14010156783451234");
        person.setParams(params);
        // 脱敏
        SensitiveUtil.parse(person);
        System.out.println(person);
    }

    @Data
    @ToString
    static class Person {
        private String name;
        private String mobile;
        private String address;
        private Map<String, Object> params;
    }
}
```

非注解方式的脱敏，依赖于`FieldType.fieldTypeMap`这个Map的配置。
需要在这个map中，配置好字段名与字段类型之间的关系。


## 添加脱敏字段类型

## 根据业务动态配置脱敏字段


