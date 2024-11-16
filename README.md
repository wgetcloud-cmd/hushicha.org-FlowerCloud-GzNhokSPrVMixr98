
# 前言


有些时候，我们可能对输出的某些字段要做特殊的处理在输出到前端，比如：身份证号，电话等信息，在前端展示的时候我们需要进行脱敏处理，这时候通过自定义注解就非常的有用了。在Jackson中要自定义注解，我们可以通过**@JacksonAnnotationsInside**注解来实现，如下示例：


## 一、自定义注解




```
import com.fasterxml.jackson.annotation.JacksonAnnotationsInside;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonSerialize(using = SensitiveSerializer.class)
public @interface Sensitive {

    //加密开始位置
    int start()default 0 ;

    //加密结束位置
    int end() default 0 ;

    //加密掩码
    String mask() default "*" ;
}
```


## 二、自定义序列化处理器SensitiveSerializer




```
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.BeanProperty;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.ContextualSerializer;
import org.springframework.util.StringUtils;
import java.io.IOException;
import java.util.Collections;

/**
 * @author songwp
 * @date 2024-11-15
 * @desc 自定义序列化器，用于对敏感字段进行脱敏处理
 */
public class SensitiveSerializer extends JsonSerializer implements ContextualSerializer {

    private Sensitive sensitive;

    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        String val = value;
        if (sensitive != null && StringUtils.hasLength(val)) {
            String m = sensitive.mask();
            int start = sensitive.start();
            int end = sensitive.end();
            int totalLength = value.length();
            if (totalLength <= 2) {
                val = totalLength == 1 ? value + m : value.substring(0, 1) + m;
            } else if (totalLength <= 6) {
                val = value.substring(0, 1) + String.join("", Collections.nCopies(totalLength - 2, m)) + value.substring(totalLength - 1);
            } else {
                int prefixLength = Math.min(start, totalLength - 1);
                int suffixLength = Math.min(end, totalLength - 1);
                if (prefixLength > totalLength) {
                    prefixLength = totalLength / 2;
                }
                if (suffixLength > totalLength) {
                    suffixLength = totalLength / 2;
                }
                int maskLength = Math.max(0, totalLength - (prefixLength + suffixLength));
                if (maskLength == 0) {
                    prefixLength -= 2;
                    suffixLength -= 2;
                    maskLength = Math.max(2, totalLength - (prefixLength + suffixLength));
                }
                prefixLength = Math.min(prefixLength, totalLength - 1);
                suffixLength = Math.min(suffixLength, totalLength - 1);
                maskLength = totalLength - prefixLength - suffixLength;
                val = value.substring(0, prefixLength) + String.join("", Collections.nCopies(maskLength, m)) + value.substring(totalLength - suffixLength);
            }
        }
        gen.writeString(val);
    }

    @Override
    public JsonSerializer createContextual(SerializerProvider prov, BeanProperty property) throws JsonMappingException {
        sensitive = property.getAnnotation(Sensitive.class);
        return this;
    }
}
```


## 三、在输出的Java Bean中使用上面的注解




```
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import com.songwp.config.Sensitive;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.io.Serializable;

/**
 * @author songwp
 * @version 1.0
 * @date 2024-11-15
 * @description: user domain
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    @JsonSerialize(using = ToStringSerializer.class)
    private Long id;
    @Sensitive(start = 2, end = 4)
    private String name;
    @Sensitive(start = 6, end = 4)
    private String idCard;
    @Sensitive(start = 4, end = 3)
    private String phone;
}
```


## 四、在前端展示结果如下：


[![](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241115153306337-910576366.png)](https://img2024.cnblogs.com/blog/2156747/202411/2156747-20241115153306337-910576366.png)


**敏感数据得到了脱敏处理。**


  * [前言](#tid-7Cm3PG)
* [一、自定义注解](#tid-SQkXEX)
* [二、自定义序列化处理器SensitiveSerializer](#tid-F85rc8)
* [三、在输出的Java Bean中使用上面的注解](#tid-M6pTwA)
* [四、在前端展示结果如下：](#tid-wamJHS):[veee加速器](https://liuyunzhuge.com)

   \_\_EOF\_\_

   ![](https://github.com/songweipeng)奋 斗  - **本文链接：** [https://github.com/songweipeng/p/18548072](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
