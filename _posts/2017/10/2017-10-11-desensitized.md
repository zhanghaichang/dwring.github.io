---
layout: post
title:  数据脱敏
category: java
tags: [java]
---
### [](#什么是数据脱敏？ "什么是数据脱敏？")什么是数据脱敏？

百度百科是这样描述的：

> 数据脱敏是指对某些敏感信息通过脱敏规则进行数据的变形，实现敏感隐私数据的可靠保护。在涉及客户安全数据或者一些商业性敏感数据的情况下，在不违反系统规则条件下，对真实数据进行改造并提供测试使用，如身份证号、手机号、卡号、客户姓名、客户地址、等个人敏感信息都需要通过脱敏规则进行数据的变形，实现敏感隐私数据的可靠保护。这样就可以在开发、测试和其他非生产环境以及外包环境中可以安全的使用脱敏后的真实数据集。

### [](#生活中的常见例子 "生活中的常见例子")生活中的常见例子

1、火车票：

![](http://ohfk1r827.bkt.clouddn.com/ticket1.jpg-1)

2、淘宝网页上的收获地址信息：

![](http://ohfk1r827.bkt.clouddn.com/ticket2.jpg-1)

### [](#敏感数据梳理 "敏感数据梳理")敏感数据梳理

在进行数据脱敏之前我们应该要确定公司的哪些数据（哪些表、哪些字段）要作为脱敏的目标，下面从用户、公司、卖家反面分析：

1、用户：名字、手机号码、身份证号码、固定电话、收货地址、电子邮箱、银行卡号、密码等

2、卖家：名字、手机号码、身份证号码、固定电话等

3、公司：交易金额、优惠券码、充值码等

### [](#确定脱敏规则 "确定脱敏规则")确定脱敏规则

确定好了公司的哪些数据要作为脱敏目标后，我们就需要制定脱敏的规则（具体的实施方法）。

常见方法：

1、替换：如统一将女性用户名替换为F，这种方法更像“障眼法”，对内部人员可以完全保持信息完整性，但易破解。

2、重排：序号12345 重排为 54321，按照一定的顺序进行打乱，很像“替换”， 可以在需要时方便还原信息，但同样易破解。

3、加密：编号 12345 加密为 23456，安全程度取决于采用哪种加密算法，一般根据实际情况而定。

4、截断：13811001111 截断为 138，舍弃必要信息来保证数据的模糊性，是比较常用的脱敏方法，但往往对生产不够友好。（丢失字段的长度）

5、掩码: 123456 -> 1xxxx6，保留了部分信息，并且保证了信息的长度不变性，对信息持有者更易辨别， 如火车票上得身份信息。（常用方法）

6、日期偏移取整：20130520 12:30:45 -> 20130520 12:00:00，舍弃精度来保证原始数据的安全性，一般此种方法可以保护数据的时间分布密度。

目前我的脱敏规则想法是：

1、【中文姓名】只显示第一个汉字，其他隐藏为2个星号，比如：`李**`

2、【身份证号】显示最后四位，其他隐藏。共计18位或者15位，比如：`*************1234`

3、【固定电话】 显示后四位，其他隐藏，比如：`*******3241`

4、【手机号码】前三位，后四位，其他隐藏，比如：`135****6810`

5、【地址】只显示到地区，不显示详细地址，比如：`上海徐汇区漕河泾开发区***`

6、【电子邮箱】 邮箱前缀仅显示第一个字母，前缀其他隐藏，用星号代替，@及后面的地址显示，比如：`d**@126.com`

7、【银行卡号】前六位，后四位，其他用星号隐藏每位1个星号，比如：`6222600**********1234`

8、【密码】密码的全部字符都用_代替，比如：****_*_****_

根据以上规则进行数据脱敏！

具体思路目前是这样的：

从原数据源查询到的生产数据 ——> 数据脱敏 ——> 更新到目标数据源。

原数据源、目标数据源、需要脱敏的表、字段等都放在配置文件中，做到可扩展性！

### [](#脱敏工具代码 "脱敏工具代码")脱敏工具代码

根据以上规则已经写好了一份简单的脱敏规则工具类。

```java
/**
 * 数据脱敏工具类
 * Created by zhisheng_tian on 2017/10/25.
 */
public class DesensitizedUtils {
    /**
     * 【中文姓名】只显示第一个汉字，其他隐藏为2个星号，比如：李**
     *
     * @param fullName
     * @return
     */
    public static String chineseName(String fullName) {
        if (StringUtils.isBlank(fullName)) {
            return "";
        }
        String name = StringUtils.left(fullName, 1);
        return StringUtils.rightPad(name, StringUtils.length(fullName), "*");
    }
    /**
     * 【身份证号】显示最后四位，其他隐藏。共计18位或者15位，比如：*************1234
     *
     * @param id
     * @return
     */
    public static String idCardNum(String id) {
        if (StringUtils.isBlank(id)) {
            return "";
        }
        String num = StringUtils.right(id, 4);
        return StringUtils.leftPad(num, StringUtils.length(id), "*");
    }
    /**
     * 【固定电话】 显示后四位，其他隐藏，比如：*******3241
     *
     * @param num
     * @return
     */
    public static String fixedPhone(String num) {
        if (StringUtils.isBlank(num)) {
            return "";
        }
        return StringUtils.leftPad(StringUtils.right(num, 4), StringUtils.length(num), "*");
    }
    /**
     * 【手机号码】前三位，后四位，其他隐藏，比如：135****6810
     *
     * @param num
     * @return
     */
    public static String mobilePhone(String num) {
        if (StringUtils.isBlank(num)) {
            return "";
        }
        return StringUtils.left(num, 3).concat(StringUtils.removeStart(StringUtils.leftPad(StringUtils.right(num, 4), StringUtils.length(num), "*"), "***"));
    }
    /**
     * 【地址】只显示到地区，不显示详细地址，比如：上海徐汇区漕河泾开发区***
     *
     * @param address
     * @param sensitiveSize 敏感信息长度
     * @return
     */
    public static String address(String address, int sensitiveSize) {
        if (StringUtils.isBlank(address)) {
            return "";
        }
        int length = StringUtils.length(address);
        return StringUtils.rightPad(StringUtils.left(address, length - sensitiveSize), length, "*");
    }
    /**
     * 【电子邮箱】 邮箱前缀仅显示第一个字母，前缀其他隐藏，用星号代替，@及后面的地址显示，比如：d**@126.com
     *
     * @param email
     * @return
     */
    public static String email(String email) {
        if (StringUtils.isBlank(email)) {
            return "";
        }
        int index = StringUtils.indexOf(email, "@");
        if (index <= 1)
            return email;
        else
            return StringUtils.rightPad(StringUtils.left(email, 1), index, "*").concat(StringUtils.mid(email, index, StringUtils.length(email)));
    }
    /**
     * 【银行卡号】前六位，后四位，其他用星号隐藏每位1个星号，比如：6222600**********1234
     *
     * @param cardNum
     * @return
     */
    public static String bankCard(String cardNum) {
        if (StringUtils.isBlank(cardNum)) {
            return "";
        }
        return StringUtils.left(cardNum, 6).concat(StringUtils.removeStart(StringUtils.leftPad(StringUtils.right(cardNum, 4), StringUtils.length(cardNum), "*"), "******"));
    }
    /**
     * 【密码】密码的全部字符都用*代替，比如：******
     *
     * @param password
     * @return
     */
    public static String password(String password) {
        if (StringUtils.isBlank(password)) {
            return "";
        }
        String pwd = StringUtils.left(password, 0);
        return StringUtils.rightPad(pwd, StringUtils.length(password), "*");
    }
}
```
### [](#最后 "最后")最后

转载请注明地址：[http://www.54tianzhisheng.cn/2017/10/28/Data-Desensitization/](http://www.54tianzhisheng.cn/2017/10/28/Data-Desensitization/)
