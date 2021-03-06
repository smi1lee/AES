# AES算法实现
### 样例输入

```
明文：abcdefghijklmno
密钥：qqqqwwwweeeerrrr
加密后：b9 9b e3 6c 97 e4 4f cf b9 65 36 f6 14 f4 b8 73
再次输入密钥：qqqqwwwweeeerrrr
解密后：a b c d e f g h i j k l m n o
```
密钥是128位

分组大小为128比特

利用Dev-C++实现。`Dev-C++`是一个Windows环境下的一个轻量级 C/C++ 集成开发环境（IDE）。

## 算法讲解

![AES](/aes.png)

### 算法实现步骤

加密模式分为以下几步

1. 密钥拓展
2. 轮密钥加操作
3. **9轮**的  “字节代替”、“行移位”、“列混淆”、“轮密钥加” 操作
4. 最后一轮进行 “字节代替”、“行移位”、“轮密钥加” 操作

经过以上步骤即可得到密文，解密操作是加密操作的逆操作。

### 详细讲解

1. S盒变换（简单替换）

   S盒是一个类型为`unsigned char`的16*16的矩阵。无符号可以保证计算机都是以无符号的类型进行运算加密。

   我们输入的每个字符为8比特，其中前4位作为行号，后四位作为列号，变换为S盒中对应位置的值。

   **tips:** 利用按位与的方式得到字符的每一个比特的值（掩码思想）

2. 密钥扩展（各种异或操作）

   AES算法由输入的128bits密钥生成10轮加密密钥的操作，流程如下

   1. 将初始密钥转化为4个32bits的字，分别记为w[0...3]

   2. 按照如下方式，依次求解w[j]，其中j是整数并且属于[4,43]

   3. 若j%4=0，则w[j]=w[j-4]⊕ g(w[j-1])，否则w[j]=w[j-4]⊕w[j-1]

      其中，函数g的流程说明如下：

      1. 将w循环左移一个字节

      2. 分别对每个字节按S盒进行映射,前四位是行，后四位为列

      3. 与**32bits**的常量 (RC[j/4],0,0,0) 进行异或，RC是一个一维数组，其值如下，记住，下面每个元素是32bits，在C语言中需要用到4个unsigned char表示。

         ``` c
         RC = {00, 01, 02, 04, 08, 10, 20, 40, 80, 1B, 36}
         ```

3. AES 算法行移位（位置替换）

   经过以上的两部，字符串已经面目全非了，但是！还不过，我们将16个字符，变成4*4的矩阵，第一行不变，第二行左移一位，第三行左移两位，第四行左移三位。到时逆过程就是相应的右移就行。                             

4. AES算法列混淆

   还不够乱！老子嫩死你。给我列混淆，其实就是将行移位后的矩阵左乘一矩阵得到新的矩阵。需要注意这个操作定义在`2^8`上的，所以**加法对应异或，乘法对应左移**，如果该值乘以2以后最高位为1，则还需要将结果异或00011011(0x1b)防止溢出。有了简单加法和乘法其他的都可以实现，例如

   ``` matlab
   b * 0x0d = b * ((0x08 + 0x04 + 0x01) 
   = (b * 0x08) + (b * 0x04) + (b * 0x01) 
   = (b * 0x02 * 0x02 * 0x02) + (b * 0x02 * 0x02) + (b * 0x01))
   =  b<<3 + b<<2 + b
   ```

5. 将上步骤得到的结果与密钥进行异或以进行加密。这里需要注意将第一次的轮密钥加操作**单独提出来写**（因为两个轮密钥的加操作不一样，具体看源码），进行初始设置。所以总结起来有1 + 4*9 + 3  = 40 个步骤。

6. 安照步骤综合以上的函数将加密函数实现。

7. 加密的逆过程就是解密。



