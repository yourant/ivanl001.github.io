```java
package im.bool.a11_huffmanCode;

import java.io.*;
import java.util.Map;

/**
 * @author : 不二
 * @date : 2021/5/16-下午1:41
 * @desc : Huffman文件编码
 **/
public class HuffmanFileCode {


    public static void huffmanDepressFile(String srcPath, String dstPath) {
        BufferedInputStream bufferedInputStream = null;
        BufferedOutputStream bufferedOutputStream = null;

        try {
            FileInputStream fileInputStream = new FileInputStream(srcPath);
            bufferedInputStream = new BufferedInputStream(fileInputStream);
            // 直接设置全部读取
            ObjectInputStream objectInputStream = new ObjectInputStream(bufferedInputStream);

          // 这里直接读取文件
            byte[] bytes1 = (byte[])objectInputStream.readObject();
            Map<Byte, String> huffmanCodeMap = (Map<Byte, String>)objectInputStream.readObject();
            System.out.println("ivanllll");

            HuffmanCode huffmanCode = new HuffmanCode();
            byte[] resultBytes = huffmanCode.huffmanDepress(bytes1, huffmanCodeMap);


            FileOutputStream fileOutputStream = new FileOutputStream(dstPath, false);
            bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
            bufferedOutputStream.write(resultBytes);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (bufferedInputStream != null) {
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bufferedOutputStream != null) {
                try {
                    bufferedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    public static void huffmanCompressFile(String srcPath, String dstPath) {
        BufferedInputStream bufferedInputStream = null;
        BufferedOutputStream bufferedOutputStream = null;

        try {
            FileInputStream fileInputStream = new FileInputStream(srcPath);
            bufferedInputStream = new BufferedInputStream(fileInputStream);
            // 直接设置全部读取
            System.out.println(bufferedInputStream.available());
            byte[] bytes = new byte[bufferedInputStream.available()];
            bufferedInputStream.read(bytes);
            System.out.println(new String(bytes));


            FileOutputStream fileOutputStream = new FileOutputStream(dstPath, false);
            bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(bufferedOutputStream);


            HuffmanCode huffmanCode = new HuffmanCode();
            byte[] bytes1 = huffmanCode.huffmanCompress(bytes);
            Map<Byte, String> huffmanCodeMap = huffmanCode.huffmanCodeMap;

          // 这里直接写入文件
            objectOutputStream.writeObject(bytes1);
            objectOutputStream.writeObject(huffmanCodeMap);


            //这里不再这么一个一个读取了
            /*while ((offset = bufferedInputStream.read(bytes)) != -1) {
                //System.out.print(new String(bytes, 0, offset));
                //
                byte[] bytes1 = huffmanCode.huffmanCompress(bytes);
                bufferedOutputStream.write(bytes1);
                System.out.println("读取一次-----");
            }*/

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (bufferedInputStream != null) {
                try {
                    bufferedInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bufferedOutputStream != null) {
                try {
                    bufferedOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }


}

```

