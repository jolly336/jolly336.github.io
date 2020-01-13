---
title: Android native和js简单交互
date: 2017-02-28 23:04:18
tags: Android
categories: Android
---

> 使用Android自身js注解来实现native和js的交互，WebView提供了一个接口**[addJavascriptInterface(Object object, String name)](https://developer.android.google.cn/reference/android/webkit/WebView.html#addJavascriptInterface%28java.lang.Object,%20java.lang.String%29)**，可以让我们注入Java对象到js页面中，这样页面中的JavaScript就能直接访问Java对象函数，从而实现Java和Js的简单交互；而Android调用js时，只需loadUrl即可。

<!-- more -->

## 测试js代码

```html
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=gb2312">
    <script type="text/javascript">
    // java代码调用js方法
    function javaCallJs() {
        document.getElementById("content").innerHTML +=
         "<br\>java调用了js函数";
    }

    </script>
</head>
<body>
this is my html <br/>
//js调用java方法,所有js全局对象、函数以及变量均自动成为window对象的成员
<a onClick="window.java2Js.JsCallJava()">点击调用java代码</a><br/>
<br/>
<div id="content">内容显示</div>
</body>
</html>
```

## Android本地代码

```java
public class MainActivity extends AppCompatActivity {

    private WebView webView;

    @SuppressLint("SetJavaScriptEnabled")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        webView = (WebView) findViewById(R.id.webView);
        // 启用webView中的JavaScript支持
        webView.getSettings().setJavaScriptEnabled(true);
        // 加载assets目录下的test.html
        webView.loadUrl("file:///android_asset/test.html");
        // 注入java对象到webView中
        webView.addJavascriptInterface(this, "java2Js");

        findViewById(R.id.btn_javaCallJs).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // --native button,java call js--
                webView.loadUrl("javascript:javaCallJs()");
            }
        });

    }

    /**
     * 注意：JELLY_BEAN_MR1 4.4后，只有public且添加了@JavascriptInterface注解的方法才能被调用。这也是为了安全考虑。
     * 毕竟页面可以直接操作Native App，有点不安全！没加注解，导致反射失败，js会调用不起来native函数。
     */
    @JavascriptInterface
    public void JsCallJava() {
        Toast.makeText(MainActivity.this, "js call java", Toast.LENGTH_LONG).show();
    }
}
```


