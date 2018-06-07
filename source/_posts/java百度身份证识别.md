---
title: java百度身份证识别
date: 2018-06-07 15:01:18
tags: [java,百度Ocr,身份证识别,原创]
categories: 
  - java
  - 百度Ocr
  - 身份证识别

---

在项目里因客户要求在注册时要求上传身份证照片来识别身份证上信息来录入信息资料，于是采用了百度ocr文字识别，废话不多说，进入正题

### 1.登录百度云(没有就先注册)   

​	 在全部产品--人工智能--文字识别创建一个应用，在创建中选择对应要的功能，我只做了身份证识别，所以我选择了默认自带的，如图

![](1.png)

创建后则在列表能看到，点进详情能获取到两个百度自动生成的值API Key ，Secret Key 

![](2.png)

这两者是调用ocr文字识别最重要的key，接下来就可以写代码了

### 2.Access Token获取([官网地址](http://ai.baidu.com/docs#/Auth/top))

1. 导入Java SDK工具包

   在创建应用后会有个JDK包下载，下载后导入项目即可，我用的是maven，所以在pom.xml文件中写入如下：

   ```
   <!--百度文字识别接口-->
   <dependency>
      <groupId>com.baidu.aip</groupId>
      <artifactId>java-sdk</artifactId>
      <version>4.3.2</version>
   </dependency>
   ```

2.  Access Token获取方法

```

/**
 * 百度文字识别demo
 */
public class baiduOcr {
    /**
     * 获取权限token
     * @return 返回示例：
     * {
     * "access_token": "24.c9303e47f0729c40f2bc2be6f8f3d589.2592000.1530936208.282335-1234567",
     * "expires_in":2592000
     * }
     */
    public static String getAuth() {
        // 官网获取的 API Key
        String clientId = "API Key";
        // 官网获取的 Secret Key
        String clientSecret = "Secret Key ";
        return getAuth(clientId, clientSecret);
    }

    /**
     * 获取API访问token
     * 该token有一定的有效期，需要自行管理，当失效时需重新获取.
     * @param ak - 百度云的 API Key
     * @param sk - 百度云的 Securet Key
     * @return assess_token 示例：
     * "24.c9303e47f0729c40f2bc2be6f8f3d589.2592000.1530936208.282335-1234567"
     */
    public static String getAuth(String ak, String sk) {
        // 获取token地址
        String authHost = "https://aip.baidubce.com/oauth/2.0/token?";
        String getAccessTokenUrl = authHost
                // 1. grant_type为固定参数
                + "grant_type=client_credentials"
                // 2. 官网获取的 API Key
                + "&client_id=" + ak
                // 3. 官网获取的 Secret Key
                + "&client_secret=" + sk;
        try {
            URL realUrl = new URL(getAccessTokenUrl);
            // 打开和URL之间的连接
            HttpURLConnection connection = (HttpURLConnection) realUrl.openConnection();
            connection.setRequestMethod("POST");//百度推荐使用POST请求
            connection.connect();
            // 获取所有响应头字段
            Map<String, List<String>> map = connection.getHeaderFields();
            // 定义 BufferedReader输入流来读取URL的响应
            BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String result = "";
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
            System.err.println("result:" + result);
            JSONObject jsonObject = new JSONObject(result);
            String access_token = jsonObject.getString("access_token");
            return access_token;
        } catch (Exception e) {
            System.err.printf("获取token失败！");
            e.printStackTrace(System.err);
        }
        return null;
    }

}
```

调用baiduOcr.getAuth()就能获取到result，result格式如下：

```
result: {
"access_token": "24.6c5e1ff107f0e8bcef8c46d3424a0e78.2592000.1485516651.282335-8574074",
"session_key":"9mzdDZXu3dENdFZQurfg0Vz8slgSgvvOAUebNFzyzcpQ5EnbxbF+hfG9DQkpUVQdh4p6HbQcAiz5RmuBAja1JJGgIdJI",
"scope": "public wise_adapt",
"refresh_token": "25.5f706c15bfc5799897518ab954b2bc07.1234567890.1843716344.1234567-1234567",
"session_secret": "dfac94a3489fe9fca7c3221cbf7525ff",
"expires_in": 2592000
}
```

其中暂时有用的就两个值，access_token和**expires_in** 

第一个是调用文字识别必带的一个参数，第二个是Access Token 的有效期一般是一个月

### 3.将本地图片进行BASE64位编码

```
import sun.misc.BASE64Encoder;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class BASE64 {
    /**
     * 将本地图片进行Base64位编码
     *
     * @param imgUrl
     *            图片的url路径，如e:\\123.png
     * @return
     */
    public static String encodeImgageToBase64(File imageFile) {
    	// 将图片文件转化为字节数组字符串，并对其进行Base64编码处理
        // 其进行Base64编码处理
        byte[] data = null;
        // 读取图片字节数组
        try {
            InputStream in = new FileInputStream(imageFile);
            data = new byte[in.available()];
            in.read(data);
            in.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 对字节数组Base64编码
        BASE64Encoder encoder = new BASE64Encoder();
        return encoder.encode(data);// 返回Base64编码过的字节数组字符串
    }
}
```

### 4.文字识别

```
public static String request(String httpUrl, String httpArg) {
    BufferedReader reader = null;
    String result = null;
    StringBuffer sbf = new StringBuffer();
    try {
        //用java JDK自带的URL去请求
        URL url = new URL(httpUrl);
        HttpURLConnection connection = (HttpURLConnection) url
                .openConnection();
        //设置该请求的消息头
        //设置HTTP方法：POST
        connection.setRequestMethod("POST");
        //设置其Header的Content-Type参数为application/x-www-form-urlencoded
        connection.setRequestProperty("Content-Type",
                "application/x-www-form-urlencoded");
        // 填入apikey到HTTP header
        connection.setRequestProperty("apikey", "uml8HFzu2hFd8iEG2LkQGMxm");
        //将第二步获取到的token填入到HTTP header
        connection.setRequestProperty("access_token", baiduOcr.getAuth());
        connection.setDoOutput(true);
        connection.getOutputStream().write(httpArg.getBytes("UTF-8"));
        connection.connect();
        InputStream is = connection.getInputStream();
        reader = new BufferedReader(new InputStreamReader(is, "UTF-8"));
        String strRead = null;
        while ((strRead = reader.readLine()) != null) {
            sbf.append(strRead);
            sbf.append("\r\n");
        }
        reader.close();
        result = sbf.toString();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return result;
}
```

写到如此，身份证的文字识别代码就差不多都写完了，赶紧调用看看效果吧

我写了个main方法去调用，当然也可以写在controller中或者Action中调用

```
  public static void main(String[] args) {
  		//获取本地的绝对路径图片
        File file = new File("C:\\Users\\Administrator\\Desktop\\123.png");
        //进行BASE64位编码
        String imageBase = BASE64.encodeImgageToBase64(file);
        imageBase = imageBase.replaceAll("\r\n","");
        imageBase = imageBase.replaceAll("\\+","%2B");
        //百度云的文字识别接口,后面参数为获取到的token
        String httpUrl="
        https://aip.baidubce.com/rest/2.0/ocr/v1/idcard?access_token="+baiduOcr.getAuth();
        String httpArg = "detect_direction=true&id_card_side=front&image="+imageBase;
        String jsonResult = request(httpUrl, httpArg);
        System.out.println("返回的结果--------->"+jsonResult);
        HashMap<String, String> map = getHashMapByJson(jsonResult);
        Collection<String> values=map.values();
        Iterator<String> iterator2=values.iterator();
        while (iterator2.hasNext()){
            System.out.print(iterator2.next()+", ");
        }
    }
    
```

请求参数我就直接把官网上的弄下来了

| 参数             | 是否必选 | 类型    | 可选值范围  | 说明                                                         |
| ---------------- | -------- | ------- | ----------- | ------------------------------------------------------------ |
| detect_direction | false    | boolean | true、false | 是否检测图像旋转角度，默认不检测，即：false。朝向是指输入图像是正常方向、逆时针旋转90/180/270度。可选值包括: - true：检测旋转角度并矫正识别； - false：不检测旋转角度，针对摆放情况不可控制的情况建议本参数置为true。 |
| id_card_side     | true     | string  | front、back | front：身份证含照片的一面；back：身份证带国徽的一面          |
| image            | true     | string  | -           | 图像数据，base64编码后进行urlencode，要求base64编码和urlencode后大小不超过4M，最短边至少15px，最长边最大4096px,支持jpg/png/bmp格式 |
| detect_risk      | false    | string  | true,false  | 是否开启身份证风险类型(身份证复印件、临时身份证、身份证翻拍、修改过的身份证)功能，默认不开启，即：false。可选值:true-开启；false-不开启 |

获取到的结果是json格式，于是我把他转换成了HashMap，代码如下：

```
 public static HashMap<String,String> getHashMapByJson(String jsonResult){
       HashMap map = new HashMap<String,String>();
       try {
           JSONObject jsonObject = new JSONObject(jsonResult);
           JSONObject words_result= jsonObject.getJSONObject("words_result");
           Iterator<String> it = words_result.keys();
           while (it.hasNext()){
               String key = it.next();
               JSONObject result = words_result.getJSONObject(key);
               String value=result.getString("words");
               switch (key){
                   case "姓名":
                       map.put("name",value);
                       break;
                   case "民族":
                       map.put("nation",value);
                       break;
                   case "住址":
                       map.put("address",value);
                       break;
                   case "公民身份号码":
                       map.put("IDCard",value);
                       break;
                   case "出生":
                       map.put("Birth",value);
                       break;
                   case "性别":
                       map.put("sex",value);
                       break;
               }
           }
       } catch (Exception e) {
           e.printStackTrace();
       }
       return map;
   }
```

至此全部代码都结束了，我把身份证附上(百度官网上弄下来的测试身份证，总不能用我自己吧)

![](123.png)

让我们来运行看下效果吧

![](3.png)

```
{
	"log_id": 6403608607836186924,
	"words_result_num": 6,
	"direction": 0,
	"image_status": "normal",
	"words_result": {
		"住址": {
			"location": {
				"width": 197,
				"top": 150,
				"height": 37,
				"left": 78
			},
			"words": "北京市海淀区上地十号七栋2单元110室"
		},
		"出生": {
			"location": {
				"width": 148,
				"top": 111,
				"height": 15,
				"left": 79
			},
			"words": "19890601"
		},
		"姓名": {
			"location": {
				"width": 63,
				"top": 32,
				"height": 25,
				"left": 77
			},
			"words": "百度熊"
		},
		"公民身份号码": {
			"location": {
				"width": 252,
				"top": 243,
				"height": 15,
				"left": 139
			},
			"words": "532101198906010015"
		},
		"性别": {
			"location": {
				"width": 20,
				"top": 76,
				"height": 15,
				"left": 71
			},
			"words": "男"
		},
		"民族": {
			"location": {
				"width": 12,
				"top": 76,
				"height": 15,
				"left": 172
			},
			"words": "汉"
		}
	}
}
```

其中main方法中的map打印为：北京市海淀区上地十号七栋2单元110室, 532101198906010015, 汉, 男, 百度熊, 19890601

OK啦，完美获取，在获取到返回结果后你想返回List还是像我一样返回Map都是可以的，这个随需求来定

本人学艺不精，上面有什么问题请大佬多指导，看到后会马上修改，谢谢啦