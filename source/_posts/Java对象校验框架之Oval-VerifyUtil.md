---
title: Java对象校验框架之Oval-VerifyUtil
categories: 编程
copyright: true
---

## Java对象校验框架之Oval-VerifyUtil

## Quick Start

### Oval框架

[Maven]
```bash
<dependency>
    <groupId>net.sf.oval</groupId>
    <artifactId>oval</artifactId>
    <version>1.81</version>
</dependency>
```

[官方文档] (http://dozer.sourceforge.net/documentation/about.html)

<!-- more -->

### VerifyUtil.java
```bash
import lombok.extern.slf4j.Slf4j;
import net.sf.oval.ConstraintViolation;
import net.sf.oval.Validator;

import java.lang.reflect.Field;
import java.util.List;

/**
 * Created by Administrator on 2017-01-02.
 */
@Slf4j
public class VerifyUtil {

    private static Validator validator = new Validator();

    public static void validateObject(Object object) {
        List<ConstraintViolation> list = validator.validate(object);
        if (null != list && !list.isEmpty()) {
            log.info("校验参数异常:{}", list.get(0).getMessage());
            throw new RuntimeException(list.get(0).getMessage());
        }
    }

    public static void validateObject(Object object, String[] profiles) {
        validator.disableAllProfiles();
        for (int i = 0; i < profiles.length; i++) {
            validator.enableProfile(profiles[i]);
        }

        List<ConstraintViolation> list = validator.validate(object);

        if (null != list && !list.isEmpty()) {
            log.info("校验参数异常:{}", list.get(0).getMessage());
            throw new RuntimeException(list.get(0).getMessage());
        }
    }

    public static void validateObject(Class clazz, String filedName, Object filedValue) {

        try{
            Field f = clazz.getDeclaredField(filedName);
            List<ConstraintViolation> list = validator.validateFieldValue(clazz.newInstance(), clazz.getDeclaredField(filedName), filedValue);
            if (null != list && !list.isEmpty()) {
                log.info("校验参数异常:{}", list.get(0).getMessage());
                throw new RuntimeException(list.get(0).getMessage());
            }
        }catch (Exception e){
            log.error("校验参数，反射异常：{}", e.getMessage());
        }
    }
}
```

### 配置JavaBean中的注解
```bash
@NotNull @NotEmpty
private String id;

@Length(min=5,max=20,profiles="nc2")
private String userName;
@NotNull @NotEmpty
private String userCode;
@Length(min=5,max=20,profiles="nc")
private String loginName;
```

### 调用示例
1、完全验证  VerifyUtil.validateObject(user);

2、选择性验证（多个验证中选择性验证） VerifyUtil.validateObject(user, profiles);
   注：根据profiles进行选择性验证entity对象，可以指定多个profiles，String[] profiles。

3、验证对象的某个字段
   public List<ConstraintViolation> validateFieldValue(final Object validatedObject, final Field validatedField,final Object fieldValueToValidate) 

三种方法在实际都能用到，oval设计的真是堪称完美！


### 复杂一些的注解验证写法，可以参考
参考博文：（https://www.cnblogs.com/koal/p/6871742.html）

```bash
@Data
public class NewHouseInputParam {

    @NotNull(errorCode = "-10001", message = "orderId不能为空")
    private Long orderId;// 订单Id
    @NotNull(errorCode = "-1", message = "INPUT订单状态不能为空")
    private Integer status; //(1,"录入报备",""),(2,"录入到访",""),(3,"录入无效",""),(4,"录入下定",""),(5,"录入成交",""),(6,"录入开票",""),(7,"录入结佣",""),

    //报备录入
    @NotNull(profiles = {"profile_1"},errorCode = "-1", message = "报备日期不能为空")
    @ValidateWithMethod(profiles = {"profile_1"},methodName = "isValidDate",parameterType = String.class, errorCode = "-1", message = "报备日期不合法")
    private String applyDate;// 报备日期

    //到访录入
    @NotNull(profiles = {"profile_2"},errorCode = "-1", message = "到访日期不能为空")
    @ValidateWithMethod(profiles = {"profile_2"},methodName = "isValidDate",parameterType = String.class, errorCode = "-1", message = "到访日期不合法")
    private String visitDate;// 到访日期
    @NotNull(profiles = {"profile_2"},errorCode = "-1", message = "到访确认单照片不能为空")
    private String visitImgKey;// 到访确认单照片

    //下定录入
    @NotNull(profiles = {"profile_4"},errorCode = "-1", message = "下定合同照片不能为空")
    private String bookImgKey;// 下定合同照片
    @NotNull(profiles = {"profile_4"},errorCode = "-1", message = "产品id不能为空")
    private Long bookProductId;// 产品id
    @NotNull(profiles = {"profile_4"},errorCode = "-1", message = "下定合同编号不能为空")
    @MaxLength(value = 20,profiles = {"profile_4"},errorCode = "-1", message = "下定合同编号不合法")
    private String bookContractCode;// 下定合同编号

    @NotNull(profiles = "profile_4",errorCode = "-1", message = "下定金额不能为空")
    @ValidateWithMethod(profiles = {"profile_4"},methodName = "isValid6Money",parameterType = String.class,
            errorCode = "-1", message = "下定金额不合法")
    private String bookPrice;// 下定金额
    @NotNull(profiles = "profile_4",errorCode = "-1", message = "购房总价不能为空")
    @ValidateWithMethod(profiles = {"profile_4"},methodName = "isValid9Money",parameterType = String.class,
            errorCode = "-1", message = "购房总价不合法")
    private String bookTotalPrice;// 购房总价
    @NotNull(profiles = "profile_4",errorCode = "-1", message = "预计佣金不能为空")
    @ValidateWithMethod(profiles = {"profile_4"},methodName = "isValid9Money",parameterType = String.class,
            errorCode = "-1", message = "预计佣金不合法")
    private String bookPossibleCommission;// 预计佣金
    @NotNull(profiles = "profile_4",errorCode = "-1", message = "下定日期不能为空")
    @ValidateWithMethod(profiles = {"profile_4"},methodName = "isValidDate",parameterType = String.class,
            errorCode = "-1", message = "下定日期不合法")
    private String bookDate;// 下定日期

    //成交录入
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "合同照片不能为空")
    @MinLength(profiles = "profile_5",value = 1,errorCode = "-1",message = "合同照片不能为空")
    private String dealImgKey;// 合同照片
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "合同总价不能为空")
    @ValidateWithMethod(profiles = {"profile_5"},methodName = "isValid6Money",parameterType = String.class,
            errorCode = "-1", message = "合同总价不合法")
    private String dealContractPrice;// 合同总价
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "产品Id不能为空")
    private Long dealProductId;// 产品Id
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "合同编号不能为空")
    @MinLength(profiles = "profile_5",value = 1,errorCode = "-1",message = "合同编号不能为空")
    private String dealContractCode;// 合同编号
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "应收佣金不能为空")
    @ValidateWithMethod(profiles = {"profile_5"},methodName = "isValid6Money",parameterType = String.class,
            errorCode = "-1", message = "应收佣金不合法")
    private String dealReceivableCommission;// 应收佣金
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "成交日期不能为空")
    @ValidateWithMethod(profiles = {"profile_5"},methodName = "isValidDate",parameterType = String.class,
            errorCode = "-1", message = "成交日期不合法")
    private String dealDate;// 成交日期
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "室不能为空")
    @Range(profiles = {"profile_5"},min = 0,max = 9,errorCode = "-1", message = "室不合法")
    private Integer bedroomSum;// 室
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "厅不能为空")
    @Range(profiles = {"profile_5"},min = 0,max = 9,errorCode = "-1", message = "厅不合法")
    private Integer livingRoomSum;// 厅
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "卫不能为空")
    @Range(profiles = {"profile_5"},min = 0,max = 9,errorCode = "-1", message = "卫不合法")
    private Integer wcSum;// 卫
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "面积不能为空")
    @ValidateWithMethod(profiles = {"profile_5"},methodName = "isValid4Money",parameterType = String.class,
            errorCode = "-1", message = "面积不合法")
    private String spaceArea;// 面积
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "室号不能为空")
    @MaxLength(profiles = {"profile_5"},value = 10,errorCode = "-1", message = "室号非法")
    private String room;// 室号
    @NotNull(profiles = {"profile_5"},errorCode = "-1", message = "楼栋号不能为空")
    @MaxLength(profiles = {"profile_5"},value = 10,errorCode = "-1", message = "楼栋号非法")
    private String building;// 楼栋号

    //开票录入
    @NotNull(profiles = {"profile_6"},errorCode = "-1", message = "开票金额不能为空")
    @ValidateWithMethod(profiles = {"profile_6"},methodName = "isValid6Money",parameterType = String.class,
            errorCode = "-1", message = "开票金额不合法")
    private String invoicePrice;// 开票金额
    @NotNull(profiles = {"profile_6"},errorCode = "-1", message = "开票日期不能为空")
    @ValidateWithMethod(profiles = {"profile_6"},methodName = "isValidDate",parameterType = String.class,
            errorCode = "-1", message = "开票日期不合法")
    private String invoiceDate;// 开票日期

    //结佣录入
    @NotNull(profiles = {"profile_7"},errorCode = "-1", message = "结佣金额不能为空")
    @ValidateWithMethod(profiles = {"profile_7"},methodName = "isValid6Money",parameterType = String.class,
            errorCode = "-1", message = "结佣金额不合法")
    private String commissionPrice;// 结佣金额
    @NotNull(profiles = {"profile_7"},errorCode = "-1", message = "结佣日期不能为空")
    @ValidateWithMethod(profiles = {"profile_7"},methodName = "isValidDate",parameterType = String.class,
            errorCode = "-1", message = "结佣日期不合法")
    private String commissionDate;// 结佣日期

    //失效录入
    @NotNull(profiles = {"profile_3"},errorCode = "-1", message = "失效原因不能为空")
    @MaxLength(profiles = {"profile_3"},value = 100,errorCode = "-1", message = "失效原因不合法")
    private String invalidDesc;// 失效原因

    /**
     * 验证日期格式是否合法
     * @param date
     * @return
     */
    public boolean isValidDate(String date){
        Date d = DateUtil.stringToDate(date, "YYYY-MM-dd");
        return d != null;
    }

    /**
     * 验证范围{0.00-9999.99}
     * @param money
     * @return
     */
    private boolean isValid4Money(String money){
        money = numericalMax2Points(money);
        if(money != null){
            float f = Float.parseFloat(money);
            if(f >= 0 && f < 10000){
                return true;
            }
        }
        return false;
    }

    /**
     * 验证范围{0.00-999999.99}
     * @param money
     * @return
     */
    private boolean isValid6Money(String money){
        money = numericalMax2Points(money);
        if(money != null){
            float f = Float.parseFloat(money);
            if(f >= 0 && f < 1000000){
                return true;
            }
        }
        return false;
    }

    /**
     * 验证范围{0.00-999,999,999.99}
     * @param money
     * @return
     */
    private boolean isValid9Money(String money){
        money = numericalMax2Points(money);
        if(money != null){
            float f = Float.parseFloat(money);
            if(f >= 0 && f < 1000000000){
                return true;
            }
        }
        return false;
    }

    /**
     * 判断在去除","后是否是数字，并且最多两位小数
     * 正确则返回处理后的money
     * 否则返回null
     * @param money
     * @return
     */
    private String numericalMax2Points(String money){
        try {
            if (money != null) {
                if (money.indexOf(",") > 0 || money.indexOf("，") > 0) {
                    money = StringUtil.remove(money, ',');
                    money = StringUtil.remove(money, '，');
                }
                if (money.matches("-?[0-9]+.?([0-9]{0,2})")) {
                    return money;
                }
            }
        }catch (Exception e){}
        return null;
    }
}
```

