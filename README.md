
## 简言

用正则表达式做用户密码强度的通过性判定，过于简单粗暴，不但用户体验差，而且用户帐号安全性也差。那么如何准确评价用户密码的强度，保护用户帐号安全呢？本文分析介绍了几种基于规则评分的密码强度检测算法，并给出了相应的演示程序。大家可以根据自己项目安全性需要，做最适合于自己的方案选择。


## 1 方案1 （简单）

方案1算法通过密码构成分析，结合权重分派，统计得出密码强度得分。得分越高，表示密码强度越大，也就越安全。方案1算法思想简单，实现容易。

### 1.1 方案1评分标准

一、密码长度：
- 5 分: 小于等于4 个字符
- 10 分: 5 到7 字符
- 25 分: 大于等于8 个字符

二、字母：
- 0 分: 没有字母
- 10 分: 全都是小（大）写字母
- 20 分: 大小写混合字母

三、数字:
- 0 分: 没有数字
- 10 分: 1 个数字
- 20 分: 大于1 个数字

四、符号:
- 0 分: 没有符号
- 10 分: 1 个符号
- 25 分: 大于1 个符号

五、奖励:
- 2 分: 字母和数字
- 3 分: 字母、数字和符号
- 5 分: 大小写字母、数字和符号

### 1.2 方案1等级划分

根据密码评分，将密码划分成以下7个等级：

- \>= 90: 非常安全（VERY_SECURE）
- \>= 80: 安全（SECURE）
- \>= 70: 非常强（VERY_STRONG）
- \>= 60: 强（STRONG）
- \>= 50: 一般（AVERAGE）
- \>= 25: 弱（WEAK）
- \>= 0:  非常弱（ VERY_WEAK）

该评分标准及等级划分，实际使用时，可小做调整，但不建议做大的变动。

### 1.3 方案1演示程序

[演示程序](http://www.42du.cn/run/48)

### 1.4 方案1测试分析

```javascript
// 评分 25，纯小写字母无法通过验证
console.log("aaaaaaaa".score());
// 评分 45，纯数字无法通过验证
console.log("11111111".score());
// 评分 47，小写+数字无法通过验证
console.log("aa111111".score());
// 评分 45，小写+大写无法通过验证
console.log("aaaaAAAA".score());
// 评分 50，4位密码不可能通过验证
console.log("11!!".score());
// 评分 70，5位密码可通过验证
console.log("0aA!!".score());
// 评分 67，小写+大写+数字可通过验证（8位）
console.log("aA000000".score());
// 评分 70，数字+符号可通过验证
console.log("000000!!".score());
```
从以上测试结果中，我们可以看出算法是十分的有效的，基本能够保证密码具有一定的安全性。但是存在的问题也很明显，其中最主要的问题是对重复或连续的字符评分过高。以测试用例中最后一个为例： `000000!!` 可以得到70分，但显然并不是一个非常强壮的密码。

另外，方案1最高可以得到95分，也就是说没有100分（绝对安全）的密码，这一点也是很有智慧的设计。

## 2 方案2 

针对方案1中的不足，方案2中引入了减分机制。对于重复出现，连续出现的字符给予适当的减分，以使得密码评分更准确。同时在方案2中密码的评分基数及计算过程都十分的复杂，要想理解其中每一步的含义，请保持足够的耐心。

### 2.1 方案2加分项

一、密码长度：
- 公式 ：+(n*4)，其中n表示密码长度

二、大写字母：
- 公式：+((len-n)*2)，其中n表示大写字母个数，len表示密码长度

三、小写字母：
- 公式：+((len-n)*2)，其中n表示小写字母个数，len表示密码长度

四、数字：
- 公式：+(n*4)，其中n表示数字个数
- 条件：满足n < len，才能得到加分，len表示密码长度

五、符号：
- 公式：+(n*6)，其中n表示符号个数

六、位于中间的数字或符号：
- 公式：+(n*2)，其中n表示位于中间的数字或符号个数

七、最低条件得分：
- 公式：+(n*2)，其中n表示满足的最低条件条目数
- 条件：只有满足最低条件，才能得到加分

其中最低条件的条目如下：

- 1.密码长度不小于8位
- 2.包含大写字母
- 3.包含小写字母
- 4.包含数字
- 5.包含符号

最低条件要求满足条目1并至少满足条目2-5中的任意三条。

### 2.2 方案2减分项

一、只有字母：
- 公式：-n，其中n表示字母个数

二、只有数字：
- 公式：-n，其中n表示数字个数

三、重复字符数（大小写敏感）：

该项描述复杂，具体计算方法见如下示例程序：

```javascript
var pass = "1111aaDD";  //示意密码
var repChar = 0;
var repCharBonus = 0;  //得分
var len = pass.length;
for(var i = 0; i < len; i++) {
    var exists = false;
    for (var j = 0; j < len; j++) {
        if (pass[i] == pass[j] && i != j) {
            exists = true;
            repCharBonus += Math.abs(len/(j-i));
        }
    }
    if (exists) {
        repChar++;
        var unqChar = len - repChar;
        repCharBonus = (unqChar) ? Math.ceil(repCharBonus/unqChar) : Math.ceil(repCharBonus);
    }
}
```
四、连续大写字母：
- 公式：-(n*2)，其中n表示连续大写字母出现的次数
- 举例：如输入AUB，则n=2

五、连续小写字母：
- 公式：-(n*2)，其中n表示连续小写字母出现的次数
- 举例：如输入aub，则n=2

六、连续数字：
- 公式：-(n*2)，其中n表示连续数字出现的次数
- 举例：如输入381，则n=2

七、正序或逆序字母：
- 公式：-(n*3)，其中n表示连续发生的次数
    - 正序或逆序是指字母表中的顺序
    - 不区分大小写
- 条件：只有连续3个字母或以上，才会减分，
- 例1：如输入ABC，则n=1
- 例2：如输入dcBA，则n=2

八、正序或逆序数字：
- 公式：-(n*3)，其中n表示连续发生的次数
- 条件：只有连续3个数字或以上，才会减分
- 例1：如输入123，则n=1，
- 例2：如输入4321，则n=2
- 例3：如输入12，则不会减分

九、正序或逆序符号：
- 公式：-(n*3)，其中n表示连续发生的次数
- 条件：只有连续3个符号或以上，才会减分

### 2.3 方案2等级划分

根据密码评分，将密码划分成以下5个等级：

- \>= 80: 非常强（VERY_STRONG）
- \>= 60: 强（STRONG）
- \>= 40: 好（GOOD）
- \>= 20: 弱（WEAK）
- \>= 0:  非常弱（ VERY_WEAK）

### 2.4 方案2演示程序

[演示程序](http://www.42du.cn/run/49)

### 2.5 方案2测试分析

```javascript
// 评分 0
console.log("11111111".score());
// 评分 2
console.log("aa111111".score());
// 评分 38
console.log("000000!!".score());
// 评分 76
console.log("Asdf2468".score());
// 评分 76
console.log("Mary2468".score());
// 评分 60
console.log("@dmin246".score());
```
从以上测试可以看出方案2较方案1有了比较大的改进和提升，尤其是对连续或重复字符上表现出色。但是方案2也存在明显的不足，主要缺点包括对人名（Mary）、单词（Story）、键盘上相连的键（Asdf）、L33T（@dmin）没法识别。

L33T：是指把拉丁字母换成数字或是特殊符号的书写形式。例如把E写成3、A写成@、to写成2、for写成4。

## 3 方案3 zxcvbn

### 3.1 简要说明

针对方案2中的不足，引入了方案3，进一步的提长密码强度。方案3完全引入一个第三方检验工具zxcvbn。

zxcvbn是一个受密码破解启发而来的密码强度估算器。它通过模式匹配和保守估计，大概可以识别大约30K左右的常规密码。主要基于美国人口普查数据，维基，美国电影，电视流行词以及其它一些常用模式，像日期，重复字符，序列字符，键盘模式和L33T会话等。

从算法的设计思想上，该方案完全秒杀基于构成的统计分析方法（前两种方法）。同时zxcvbn支持多种开发语言。因其模式的复杂及字典的存在，当前版本的zxcvbn.js大约有800多K。

要了解项目的详情及算法见zxcvbn官网：

[github zxcvbn](https://github.com/dropbox/zxcvbn)

### 3.2 方案3演示程序

[演示程序](http://www.42du.cn/run/50)

以上是三胖对密码强度检测算法和方案的理解和分析，不足之处还请大家多多指正！

[原文链接](http://30ke.cn/doc/pass-strength)
