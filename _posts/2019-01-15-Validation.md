---
layout:     post
title:      "参数校验"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-16 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - validation
---

> “Yeah It's on. ”

* content
{:toc}
我们可能会经常需要对传入的参数进行校验，如果数据比较少的时候还比较容易处理，但当数据比较多的时候会显得比较麻烦，而且处理不当的时候，还会代码重复。这时候就需要使用使用Validation对数据进行校验了。

## JSR303
JSR-303 是JAVA EE 6 中的一项子规范，叫做Bean Validation，Hibernate Validator 是 Bean Validation 的参考实现 . Hibernate Validator 提供了 JSR 303 规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。

## JSR303 基本的校验规则
空检查 
@Null 验证对象是否为null 
@NotNull 验证对象是否不为null, 无法查检长度为0的字符串 
@NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格. 
@NotEmpty 检查约束元素是否为NULL或者是EMPTY.

Booelan检查 
@AssertTrue 验证 Boolean 对象是否为 true 
@AssertFalse 验证 Boolean 对象是否为 false

长度检查 
@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内 
@Length(min=, max=) Validates that the annotated string is between min and max included.

日期检查 
@Past 验证 Date 和 Calendar 对象是否在当前时间之前，验证成立的话被注释的元素一定是一个过去的日期 
@Future 验证 Date 和 Calendar 对象是否在当前时间之后 ，验证成立的话被注释的元素一定是一个将来的日期 
@Pattern 验证 String 对象是否符合正则表达式的规则，被注释的元素符合制定的正则表达式，regexp:正则表达式 flags: 指定 Pattern.Flag 的数组，表示正则表达式的相关选项。

数值检查 
建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，但可以转换为Stirng为”“,Integer为null 
@Min 验证 Number 和 String 对象是否大等于指定的值 
@Max 验证 Number 和 String 对象是否小等于指定的值 
@DecimalMax 被标注的值必须不大于约束中指定的最大值. 这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度 
@DecimalMin 被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度 
@Digits 验证 Number 和 String 的构成是否合法 
@Digits(integer=,fraction=) 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。 
@Range(min=, max=) 被指定的元素必须在合适的范围内 
@Range(min=10000,max=50000,message=”range.bean.wage”) 
@Valid 递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验.(是否进行递归验证) 
@CreditCardNumber信用卡验证 
@Email 验证是否是邮件地址，如果为null,不进行验证，算通过验证。 
@ScriptAssert(lang= ,script=, alias=) 
@URL(protocol=,host=, port=,regexp=, flags=)

## 自定义校验信息和错误处理
Validation注解中都有一个message的属性可以通过这个属性自定义错误信息。如下：
```
public class Book {
    private Integer id;
    @NotBlank(message = "书名不能为空")
    private String bookName;
    @NotBlank(message = "作者不能为空")
    private String bookAuthor;
    // 省略get和set方法
}
```
Spring Boot的Controller方法中可以传一个BindingResult或者Errors类型的参数，值得注意的一点是这个参数的位置必须是参数列表的被@Valid注解修饰的参数紧跟着的（也就是被@Valid修饰的下一个参数）后面，如果不是下一个，那么这个参数将不起作用。
下面的代码是如何进行自定义错误处理：
```
@RestController
public class BookController {
    @PostMapping("/book")
    public Book getBook(@Valid Book book, BindingResult result, HttpServletResponse response) {
        if (result.hasErrors()) {
            result.getAllErrors().forEach((error) -> {
                FieldError fieldError = (FieldError) error;
                // 属性
                String field = fieldError.getField();
                // 错误信息
                String message = fieldError.getDefaultMessage();
                System.out.println(field + ":" + message);
            });

        }
        // ...
        return book;
    }
}
```

## 全局校验异常处理

当定义了JSR303校验器后，校验不通过都会产生一个BindException，输出错误信息。若要对异常处理，我们可以通过ResponseEntityExceptionHandler类或者定义一个全局异常处理的拦截器来进行处理。

### ResponseEntityExceptionHandler
SpringBoot 中有一个专门处理错误信息的一个类叫做ResponseEntityExceptionHandler。其中有很多关于400的错误处理，也就是参数错误的处理，其中就有一个专门用来处理没有通过校验的参数的方法。我们重写这个类的这个方法即可。
```
@ControllerAdvice   // Spring 的异常处理的注解
public class BadRequestExceptionHandler extends ResponseEntityExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());


    @Override
    protected ResponseEntity<Object> handleBindException(BindException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {

        Map<String, String> messages = new HashMap<>();
        BindingResult result = ex.getBindingResult();
        if (result.hasErrors()) {
            List<ObjectError> errors = result.getAllErrors();
            for (ObjectError error : errors) {
                FieldError fieldError = (FieldError) error;
                messages.put(fieldError.getField(), fieldError.getDefaultMessage());
            }
            logger.error(messages.toString());
        }
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(messages);
    }
}
```

### @ControllerAdvice
```
/**该注解会 适用所有的@RequestMapper() 结合@ExceptionHander 实现全局异常处理
 *  */
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class) /*定义拦截 异常的范围 此时 是拦截所有异常*/
    public Result<String> exceptionHandler(HttpServletRequest request,Exception e){
        e.printStackTrace();
        if (e instanceof GlobalException){
            GlobalException globalException = (GlobalException)e;
            return Result.error(globalException.getCodeMsg());
        }else if (e instanceof BindException){
            /*注意：此处的BindException 是 Spring 框架抛出的Validation异常*/
            BindException ex = (BindException)e;
            List<ObjectError> errors = ex.getAllErrors();/*抛出异常可能不止一个 输出为一个List集合*/
            ObjectError error = errors.get(0);/*取第一个异常*/
            String errorMsg = error.getDefaultMessage(); /*获取异常信息*/
            return Result.error(CodeMsg.BIND_ERROR.fillArgs(errorMsg));
        }else {
            return Result.error(CodeMsg.SERVER_ERROR);
        }
    }
}
```
@ControllerAdvice是一个@Component，用于定义@ExceptionHandler，@InitBinder和@ModelAttribute方法，适用于所有使用@RequestMapping方法。

意思就是该注解会对所有@RequestMapping方法进行检查，拦截。并进行异常处理。

拦截的范围 可以选定拦截异常范围，最终输出，以Result（CodeMsg）的形式输出。将异常信息放入 msg中。

于是我们的业务异常和运行时的其他异常可以在这个类中一并处理。
