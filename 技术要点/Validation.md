`Validation`是用于检查程序代码中参数的有效性的框架，作为`Spring`中的一个参数校验工具，集成在`Spring-Context`中
- `@AssertFalse`：适用于`Boolean`，验证注解元素值为`false`
- `@AssertTrue`：适用于`Boolean`，验证注解元素值为`true`
- `@NotNull`：适用于任意类型，验证注解的元素值不是`null`
- `@Null`：适用于任意类型，验证注解的元素值是`null`
- `@Min(value=值)`：适用于`BigDecimal`、`BigInteger`、`byte`、`short`、`int`、`long`、`任意Number或存储数字的CharSequence`，验证注解的元素值大于等于`@Min`指定的`value`值
- `@Max(value=值)`：适用于`BigDecimal`、`BigInteger`、`byte`、`short`、`int`、`long`、`任意Number或存储数字的CharSequence`，验证注解的元素值小于等于`@Max`指定的`value`值
- `@DecimalMin(value=值)`：与`@Min`基本一致，但`value`可以填写`decimal`
- `@DecimalMax(value=值)`：与`@Max`基本一致，但`value`可以填写`decimal`
- `@Digits(integer=整数位数, fraction=小数位数)`：适用类型与`@Min`一致，但验证元素值的整数位数与小数位数
- `@Size(min=下限, max=上限)`：适用于字符串、`Collection`、`Map`、数组，验证字符串或者集合的大小上下限
- `@Past`：适用于`java.util.Date java.util.Calendar java.Time`类库的日期类型，验证注解的元素值（日期类型）比当前时间早
- `@Future`：适用于`java.util.Date java.util.Calendar java.Time`类库的日期类型，验证注解的元素值（日期类型）比当前时间晚
- `@NotBlank`：适用于`CharSequence`子类型，验证注解的元素值不为空（不为`null`、去除首位空格后长度为0），不同于`@NotEmpty`，`@NotBlank`只应用于字符串且在比较时会去除字符串的首位空格
- `@Length(min=下限, max=上限)`：适用于`CharSequence`子类型，验证注解的元素值长度在`min`和`max`区间内
- `@NotEmpty`：适用于`CharSequence`子类型、`Collection`、`Map`、数组，验证注解的元素值不为`null`且不为空（字符串长度不为0、集合大小不为0）
- `@Range(min=最小值, max=最大值)`：适用于`BigDecimal, BigInteger, CharSequence, byte, short, int, long`等原子类型和包装类型，验证注解的元素值在最小值和最大值之间
- `@Email(regexp=正则表达式,flag=标志的模式)`
- `@Pattern(regexp=正则表达式,flag=标志的模式)`
- `@Valid`：指定递归验证关联的对象；如用户对象中有个地址对象属性，如果想在验证用户对象时一起验证地址对象的话，在地址对象上加`@Valid`注解即可级联验证

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
/**
 * 用户类
 */
@Setter
@Getter
@NoArgsConstructor
public class User implements Serializable {

   /**
    * 主键id
    */
   private Integer id;

   /**
    * 用户名
    */
   @NotEmpty(message = "用户名不能为空！")
   private String username;

   /**
    * 密码
    */
   @Size(min = 8, message = "密码长度不能小于8！")
   @NotEmpty(message = "密码不能为空！")
   private String password;

   /**
    * 邮箱
    */
   @Email(message = "邮箱格式错误！")
   private String email;

   /**
    * 年龄
    */
   @Min(value = 18, message = "年龄不能小于18！")
   @Max(value = 150, message = "年龄不能小于150！")
   private int age;

}

@PostMapping("/add")
public String add(@RequestBody @Valid User user, BindingResult errors) {
   if (errors.hasErrors()) {
      return errors.getFieldError().getDefaultMessage();
   }
   userService.add(user);
   return "添加成功！";
}
```

`Controller`方法参数中多了一个`BindingResult`类型的参数，该对象存放`validation`校验的错误信息，一般通过`hasErrors`方法判断是否有错误，通过`errors.getFieldError().getDefaultMessage()`获取错误信息

同为一个需要校验的类或者对象，可能在一个业务场景需要做A类型的校验，而在另一个业务场景需要做B类型的校验（`User`类，在`userAdd`接口需要`password`不为空，而在`userUpdate`接口则不需要填写`password`）

```java
package com.example.validationtest.param;

/**
 * 校验规则类
 */
public class ValidationRules {

   /**
    * 注册（添加）用户规则
    */
   public interface UserAdd {

   }

   /**
    * 更新（修改）用户规则
    */
   public interface UserUpdate {

   }

}


package com.example.validationtest.dataobject;

import com.example.validationtest.param.ValidationRules;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.validation.constraints.*;
import java.io.Serializable;

/**
 * 用户类
 */
@Setter
@Getter
@NoArgsConstructor
public class User implements Serializable {

   /**
    * 主键id，将其设定分组为ValidationRules.UserUpdate，表示用户信息修改时校验该规则
    */
   @NotNull(groups = ValidationRules.UserUpdate.class, message = "用户id不能为空！")
   private Integer id;

   /**
    * 用户名，将其设定分组为ValidationRules.UserAdd，表示添加用户时校验该规则
    */
   @NotEmpty(groups = ValidationRules.UserAdd.class, message = "用户名不能为空！")
   private String username;

   /**
    * 密码，将长度校验规则同时加入到ValidationRules.UserAdd和ValidationRules.UserUpdate组，表示添加用户和用户信息修改时都要校验这个规则，空值校验只有添加时校验
    */
   @Size(groups = {ValidationRules.UserAdd.class, ValidationRules.UserUpdate.class}, 
	   min = 8, message = "密码长度不能小于8！")
   @NotEmpty(groups = ValidationRules.UserAdd.class, message = "密码不能为空！")
   private String password;

   // 下面都是一回事

   /**
    * 邮箱
    */
   @Email(groups = {ValidationRules.UserAdd.class, ValidationRules.UserUpdate.class}, 
	   message = "邮箱格式错误！")
   private String email;

   /**
    * 年龄
    */
   @Min(groups = {ValidationRules.UserAdd.class, ValidationRules.UserUpdate.class}, 
	   value = 18, message = "年龄不能小于18！")
   @Max(groups = {ValidationRules.UserAdd.class, ValidationRules.UserUpdate.class}, 
	   value = 150, message = "年龄不能小于150！")
   private int age;

}


package com.example.validationtest.api;

import com.example.validationtest.dataobject.User;
import com.example.validationtest.param.ValidationRules;
import org.springframework.validation.BindingResult;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 测试校验（这里不做实际的用户添加删除工作，只是测试validation校验）
 */
@RestController
@RequestMapping("/api/user")
public class UserAPI {

   /**
    * 模拟添加用户，这里设置校验规则为ValidationRules.UserAdd
    */
   @PostMapping("/add")
   public String add(@RequestBody @Validated(ValidationRules.UserAdd.class) User user, 
	   BindingResult errors) {
      if (errors.hasErrors()) {
         return errors.getFieldError().getDefaultMessage();
      }
      return "添加成功！";
   }

   /**
    * 模拟修改用户，这里设置校验规则为ValidationRules.UserUpdate
    */
   @PostMapping("/update")
   public String update(@RequestBody @Validated(ValidationRules.UserUpdate.class) User user, 
	   BindingResult errors) {
      if (errors.hasErrors()) {
         return errors.getFieldError().getDefaultMessage();
      }
      return "修改成功！";
   }
}
```