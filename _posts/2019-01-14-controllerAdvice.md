---
layout:     post
title:      "SpringMVC异常处理"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-01-12 22:00:00
author:     "Panying"
header-img: "img/post-bg-2015.jpg"
tags:
    - SpringMVC
---

* content
{:toc}

## 一、异常处理的原则

1、调用方法的时候返回布尔值来代替返回null，这样可以 NullPointerException。由于空指针是java异常里最恶心的异常。

2、 catch块里别不写代码。空catch块是异常处理里的错误事件，因为它只是捕获了异常，却没有任何处理或者提示。通常你起码要打印出异常信息，当然你最好根据需求对异常信息进行处理。

3、能抛受控异常（checked Exception）就尽量不抛受非控异常(unchecked Exception[Error或者RuntimeException的异常])。通过去掉重复的异常处理代码，可以提高代码的可读性。

4、 绝对不要让你的数据库相关异常显示到客户端。由于绝大多数数据库和SQLException异常都是受控异常，在Java中，你应该在DAO层把异常信息处理，然后返回处理过的能让用户看懂并根据异常提示信息改正操作的异常信息。

5、 在Java中，一定要在数据库连接，数据库查询，流处理后，在finally块中调用close()方法。

## 二、示例说明

本示例以“前后端分离模式”进行演示，调试用的异常信息通过日志的形式打印出来，代码并不完整，仅从异常处理进行部分代码示例。

### 1、创建异常类

```
 1 @Getter //通过lombok插件实现省写setter或者getter方法
 2 public class SellException extends RuntimeException {
 3 
 4     private Integer code;
 5     private String message;
 6 
 7     public SellException(ResultEnum resultEnum) {
 8         super(resultEnum.getMessage());
 9         this.code = resultEnum.getCode();
10     }
11 
12     public SellException(Integer code,String message) {
13         this.code = code;
14         this.message = message;
15     }
16 }
```
### 2、使用Handler类捕获异常，统一格式返回给前端

```
  @ControllerAdvice
  public class SellExceptionHandler {
  
      @ExceptionHandler(value = SellException.class)
      @ResponseBody
      public Object handlerSellerException(SellException e){
          if(isJson(request)) {
            return ResultVOUtil.error(e.getCode(),e.getMessage());
          } else {
            ModelAndView modelAndView = initModelAndView();
            //if (e.getCode().equalsIgnoreCase("login_first")) {
            //    modelAndView.setViewName("redirect:/list");
            //}
            //if (e.getCode().equalsIgnoreCase("real_name_not_set")) {
            //    modelAndView.setViewName("redirect:/account");
            //}else{
            //    modelAndView.setViewName("/404");
            //}
            modelAndView.setViewName("/error");
            modelAndView.addObject("exception", e);
            return modelAndView;
          }
     }

  
  private Boolean isJson(HttpServletRequest request){
        String header = request.getHeader("content-type");
        return header != null && header.contains("json");
    }

  }
```
　　统一格式类：

```
 1 public class ResultVOUtil {
 2 
 3     public static ResultVO success(Object object) {
 4         ResultVO resultVO = new ResultVO();
 5         resultVO.setData(object);
 6         resultVO.setCode(0);
 7         resultVO.setMsg("成功");
 8         return resultVO;
 9     }
10 
11     public static ResultVO success() {
12         return success(null);
13     }
14 
15     public static ResultVO error(Integer code,String msg) {
16         ResultVO resultVO = new ResultVO();
17         resultVO.setCode(code);
18         resultVO.setMsg(msg);
19         return resultVO;
20     }
21 
22 }
```
```
 1 @Data
 2 public class ResultVO<T> implements Serializable{
 3 
 4     private static final long serialVersionUID = 8960474786737581150L;
 5 
 6     /**
 7      * 错误码
 8      */
 9     private Integer code;
10     /**
11      *提示信息
12      */
13     private String msg;
14     /**
15      * 具体内容
16      */
17     private T data;
18 
19 }
```
### 3、异常的信息通过枚举统一定义，方便定义管理

```
 1 @Getter
 2 public enum ResultEnum {
 3 
 4     SUCCESS(0,"成功"),
 5 
 6     PARAM_ERROR(1,"参数不正确"),
 7 
 8     PRODUCT_NOT_EXIST(10,"商品不存在"),
 9 
10     PRODUCT_STOCK_ERROR(11,"商品库存不正确"),
11 
12     ORDER_NOT_EXIST(12,"订单不存在"),
13 
14     ORDERDETAIL_NOT_EXIST(13,"订单详情不存在"),
15 
16     ORDER_STATUS_ERROR(14,"订单状态不正确"),
17 
18     ORDER_UPDATE_FAIL(15,"订单更新失败"),
19 
20     ORDER_DETAIL_EMPTY(16,"订单详情为空"),
21 
22     CART_EMPTY(18,"购物车为空"),
23 
24     ORDER_OWNER_ERROR(19,"该订单不属于当前用户"),
25     ;
26 
27     private Integer code;
28     private String message;
29 
30     ResultEnum(Integer code, String message) {
31         this.code = code;
32         this.message = message;
33     }
34 
35 }
```
### 4、使用异常类

① controller层处理页面传来参数的校验，如参数不正确，抛出自定义异常，由handler捕获返回给页面。

```
 1 @RestController
 2 @RequestMapping("/buyer/order")
 3 @Slf4j
 4 public class BuyerOrderController {
 5 
 6     @Autowired
 7     private OrderService orderService;
 8 
 9     @Autowired
10     private BuyerService buyerService;
11 
12     //创建订单
13     @PostMapping("/create")
14     public ResultVO<Map<String,String>> create(@Valid OrderForm orderForm,
15                                                BindingResult bindingResult){
16         if (bindingResult.hasErrors()){
17             log.error("[创建订单] 参数不正确,orderForm={}",orderForm);
18             throw new SellException(ResultEnum.PARAM_ERROR.getCode(),
19                     bindingResult.getFieldError().getDefaultMessage());
20         }
21         OrderDTO orderDTO = OrderForm2OrderDTOConverter.convert(orderForm);
22         if (CollectionUtils.isEmpty(orderDTO.getOrderDetailList())){
23             log.error("[创建订单] 购物不能为空");
24             throw new SellException(ResultEnum.CART_EMPTY);
25         }
26 
27         OrderDTO createResult = orderService.create(orderDTO);
28         Map<String,String> map = new HashMap<>();
29         map.put("orderId",createResult.getOrderId());
30 
31         return ResultVOUtil.success(map);
32 
33     }
34     //订单列表
35     @GetMapping("/list")
36     public ResultVO<List<OrderDTO>> list(@RequestParam("openid") String openid,
37                                          @RequestParam(value = "page",defaultValue = "0") Integer page,
38                                          @RequestParam(value = "size",defaultValue = "10") Integer size){
39         if (StringUtils.isEmpty(openid)){
40             log.error("[查询订单列表] openid为空");
41             throw new SellException(ResultEnum.PARAM_ERROR);
42         }
43         PageRequest request = new PageRequest(page, size);
44         Page<OrderDTO> orderDTOPage = orderService.findList(openid, request);
45 
46         return ResultVOUtil.success(orderDTOPage.getContent());
47     }
48 
49     //订单详情
50     @GetMapping("/detail")
51     public ResultVO<OrderDTO> detail(@RequestParam("openid") String openid,
52                                      @RequestParam("orderId") String orderId){
53 
54         OrderDTO orderDTO = buyerService.findOrderOne(openid, orderId);
55         return ResultVOUtil.success(orderDTO);
56     }
57 
58     //取消订单
59     @PostMapping("/cancel")
60     public ResultVO<OrderDTO> cancel(@RequestParam("openid") String openid,
61                                      @RequestParam("orderId") String orderId) {
62 
63         buyerService.cancelOrderOne(openid, orderId);
64         return ResultVOUtil.success();
65     }
66 }
```
② service处理返回的结果的异常，抛出自定义异常，由handler捕获返回给页面。

```
 1 @Service
 2 @Slf4j
 3 public class BuyerServiceImpl implements BuyerService {
 4 
 5     @Autowired
 6     private OrderService orderService;
 7 
 8     @Override
 9     public OrderDTO findOrderOne(String openid, String orderId) {
10        return checkOrderOwner(openid, orderId);
11     }
12 
13     @Override
14     public OrderDTO cancelOrderOne(String openid, String orderId) {
15         OrderDTO orderDTO = checkOrderOwner(openid, orderId);
16         if (orderDTO == null){
17             log.error("[取消订单] 查不到该订单,orderDTO={}",orderId);
18             throw new SellException(ResultEnum.ORDER_NOT_EXIST);
19         }
20         return orderService.cancel(orderDTO);
21     }
22 
23     private OrderDTO checkOrderOwner(String openid, String orderId) {
24         OrderDTO orderDTO = orderService.findOne(orderId);
25         if (orderDTO == null){
26             return null;
27         }
28         if (!orderDTO.getBuyerOpenid().equalsIgnoreCase(openid)){
29             log.error("[查询订单] 订单的openid不一致,openid={},orderDTO={}",openid,orderDTO);
30             throw new SellException(ResultEnum.ORDER_OWNER_ERROR);
31         }
32         return orderDTO;
33     }
34 
35 }
```
