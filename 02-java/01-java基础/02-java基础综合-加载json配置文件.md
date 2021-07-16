[toc]

## 1, 加载json配置文件

```java
package org.bool.a09_jsonConfigFile;

import org.apache.commons.io.IOUtils;
import org.springframework.boot.system.ApplicationHome;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;


/**
 * @author : 不二
 * @date : 2021/6/10-上午7:58
 * @desc :
 **/
public class ConfigUtils {

    public static String loadJsonFile(String fileName){

        String priorityPath = getJarDir() + "/" + fileName;

        System.out.println("优先获取文件位置:" + priorityPath);
        File file = new File(priorityPath);
        InputStream resourceAsStream = null;
        try {
            if(file.exists()){
                // 我知道了，这里是先获取jar包下是否有相同文件名的文件，如果有，优先获取这里的
                resourceAsStream = new FileInputStream(file);
            }else{
                System.out.println("优先获取位置无该文件，从资源目录下获取该文件");
                resourceAsStream = Thread.currentThread().getContextClassLoader().getResourceAsStream(fileName);
            }
            String json = IOUtils.toString(resourceAsStream, "utf-8");
            return json;
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("配置文件"+fileName+"读取异常");
        }

    }


    public static String getJarDir() {
        File file = getJarFile();
        return file.getParent();
    }



    private static File getJarFile()
    {

        ApplicationHome h = new ApplicationHome(ConfigUtils.class);
        File jarF = h.getSource();
        System.out.println(jarF.getParentFile().toString());

//         String path = ConfigUtil.class.getProtectionDomain().getCodeSource().getLocation().getFile();
//        try
//        {
//            path = java.net.URLDecoder.decode(path, "UTF-8"); // 转换处理中文及空格
//        }
//        catch (java.io.UnsupportedEncodingException e)
//        {
//            return null;
//        }
        return jarF;
    }
}

```

