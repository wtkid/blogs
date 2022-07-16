> # Base64相关

## base64

Base64 图片保存是以 String 的形式存储的，页面上 img 标签显示图片的方式：

①src 直接填写这个字符串是可以显示的，

②src 填写的如果是一个 url，并且这个 url 是以流的方式输出的图片。

那如果 url 不是以流的形式输出的，而是返回 base64 的一串 string，name 直接填这个 url 能否显示图片呢，答案是不能，所以我们需要将这个 String 转换成流的方式输出，以下是转流的工具类。

注意：spring 自己也有一套 org.springframework.util.Base64Utils 也可以用。

> 我自己的 Base64utils

```java

/**
 * @author: wangtao
 * @Date:12:09 2017/10/20
 * @Email:386427665@qq.com
 */
public class Base64Utils {

    public static String encode(InputStream is) throws IOException {
        BASE64Encoder encoder = new BASE64Encoder();
        byte[] data = new byte[is.available()];
        is.read(data);
        is.close();
        String str = encoder.encode(data);
        return str;
    }

    public static InputStream decode(String file) throws Exception {
        BASE64Decoder decoder = new BASE64Decoder();
        byte[] bytes = decoder.decodeBuffer(file);
        //测试时发现不调整异常数据也正确，为了避免以后出错，加上这一段
        for (int i = 0; i < bytes.length; ++i) {
            if (bytes[i] < 0) {// 调整异常数据
                bytes[i] += 256;
            }
        }
        InputStream in = new ByteArrayInputStream(bytes);
        return in;
    }

    public static void toOutputStream(String file, OutputStream os) {
        try {
            InputStream in = decode(file);
            inputStreamToOutputStream(in, os);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void inputStreamToOutputStream(InputStream in, OutputStream os) throws IOException {
        byte[] b = new byte[4096];
        Integer len = -1;
        while ((len = in.read(b)) != -1) {
            os.write(b, 0, len);
        }
        os.flush();
        in.close();
        os.close();
    }
}
```