## 1. Kaptcha
```xml
<!--Kaptcha-->
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

```java
//Kaptcha配置
@Configuration
public class KaptchaConfig {
    @Bean
    public DefaultKaptcha producer() {
        //Properties类
        Properties properties = new Properties();
        // 图片边框
        properties.setProperty("kaptcha.border", "yes");
        // 边框颜色
        properties.setProperty("kaptcha.border.color", "105,179,90");
        // 字体颜色
        properties.setProperty("kaptcha.textproducer.font.color", "blue");
        // 图片宽
        properties.setProperty("kaptcha.image.width", "110");
        // 图片高
        properties.setProperty("kaptcha.image.height", "40");
        // 字体大小
        properties.setProperty("kaptcha.textproducer.font.size", "30");
        // session key
        properties.setProperty("kaptcha.session.key", "code");
        // 验证码长度
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        // 字体
        properties.setProperty("kaptcha.textproducer.font.names", "宋体,楷体,微软雅黑");
        //图片干扰
        properties.setProperty("kaptcha.noise.impl","com.google.code.kaptcha.impl.DefaultNoise");
        //Kaptcha 使用上述配置
        Config config = new Config(properties);
        //DefaultKaptcha对象使用上述配置, 并返回这个Bean
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}
```

```java
@Component
public class UUIDUtil {
    /**
     * 生成32位的随机UUID
     * @return 字符形式的小写UUID
     */
    @Bean
    public String getUUID32() {
        return UUID.randomUUID().toString()
                .replace("-", "").toLowerCase();
    }
}


@Component
//Captcha 生成工具
public class CaptchaUtil {
    @Autowired
    private DefaultKaptcha producer;
    @Autowired
    private CaptchaService captchaService;
    //生成catchCreator的map
    public Map<String, Object> catchaImgCreator() throws IOException {
        //生成文字验证码
        String text = producer.createText();
        //生成文字对应的图片验证码
        BufferedImage image = producer.createImage(text);
        //将图片写出
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        ImageIO.write(image, "jpg", outputStream);
        //对写出的字节数组进行Base64编码 ==> 用于传递8比特字节码
        BASE64Encoder encoder = new BASE64Encoder();
        //生成token
        Map<String, Object> token = captchaService.createToken(text);
        token.put("img", encoder.encode(outputStream.toByteArray()));
        return token;
    }
}
```

```java
@Service
public class CaptchaServiceImpl implements CaptchaService {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    @Autowired
    private UUIDUtil uuidUtil;
    @Autowired
    private CaptchaUtil captchaUtil;
    //从SpringBoot的配置文件中取出过期时间
    @Value("${server.servlet.session.timeout}")
    private Integer timeout;
    //UUID为key, 验证码为Value放在Redis中
    @Override
    public Map<String, Object> createToken(String captcha) {
        //生成一个token
        String key = uuidUtil.getUUID32();
        //生成验证码对应的token  以token为key  验证码为value存在redis中
        ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        valueOperations.set(key, captcha);
        //设置验证码过期时间
        redisTemplate.expire(key, timeout, TimeUnit.MINUTES);
        Map<String, Object> map = new HashMap<>();
        map.put("token", key);
        map.put("expire", timeout);
        return map;
    }
    //生成captcha验证码
    @Override
    public Map<String, Object> captchaCreator() throws IOException {
        return captchaUtil.catchaImgCreator();
    }
    //验证输入的验证码是否正确
    @Override
    public String versifyCaptcha(String token, String inputCode) {
        //根据前端传回的token在redis中找对应的value
        ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        if (redisTemplate.hasKey(token)) {
            //验证通过, 删除对应的key
            if (valueOperations.get(token).equals(inputCode)) {
                redisTemplate.delete(token);
                return "true";
            } else {
                return "false";
            }
        } else {
            return "false";
        }
    }
}

@RestController
public class LoginController {
    @Autowired
    CaptchaService captchaService;
    @GetMapping("/captcha")
    public Map<String, Object> captcha() throws IOException {
        return captchaService.captchaCreator();
    }
    @GetMapping("/login1")
    public String login(@RequestParam("token") String token,
                              @RequestParam("inputCode") String inputCode) {
        return captchaService.versifyCaptcha(token, inputCode);
    }
}
```

## 2. EasyCaptcha
```xml
<dependency> <groupId>com.github.whvcse</groupId>
	<artifactId>easy-captcha</artifactId>
	<version>1.6.2</version>
</dependency>
```

```java
@RequestMapping("/captcha")
public void captcha(HttpServletRequest request, HttpServletResponse response) throws Exception{
    GifCaptcha gifCaptcha = new GifCaptcha(130,48,4);
    CaptchaUtil.out(gifCaptcha, request, response);
    String verCode = gifCaptcha.text().toLowerCase();
    request.getSession().setAttribute("CAPTCHA",verCode);  //存入session 
}
```

## 3. HappyCaptcha
```xml
<dependency>
	<groupId>com.ramostear</groupId>
	<artifactId>Happy-Captcha</artifactId>
	<version>1.0.1</version>
</dependency>
```


