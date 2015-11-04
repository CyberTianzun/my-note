# Android中WebView中拦截所有请求并替换URL

## 需求背景

接到这样一个需求，需要在 WebView 的所有网络请求中，在请求的url中，加上一个xxx=1的标志位。

例如 `http://www.baidu.com` 加上标志位就变成了 `http://www.baidu.com?xxx=1`

## 寻找解决方案

从 Android API 11 (3.0) 开始，WebView 开始在 `WebViewClient` 内提供了这样一条 API ，如下：

```
public WebResourceResponse shouldInterceptRequest(WebView view, String url)
```

就是说只要实现 `WebViewClient` 的 `shouldInterceptRequest` 方法，然后调用 `WebView` 的 `setWebViewClient` 就可以了。

但是，在 API21 以上又弃用了上述 API，使用了一条新的 API，如下：

```
public WebResourceResponse shouldInterceptRequest(WebView view, final WebResourceRequest request)
```

好吧，为了支持尽量多的版本，看来两个都需要实现了，发现一看就非常好用的 `String url` 变成了一个 `WebResourceRequest request`。`WebResourceRequest` 这个东西是一个接口，并且是这样定义的：

```
public interface WebResourceRequest {
    Uri getUrl();
    boolean isForMainFrame();
    boolean hasGesture();
    String getMethod();
    Map<String, String> getRequestHeaders();
}
```

在其中没有发现任何可以直接替换请求的方法。

然后搜索了一下 Android 代码中对他的引用，[点我搜索](http://androidxref.com/5.1.0_r1/search?q=WebResourceRequest&defs=&refs=&path=&hist=&project=frameworks)。然后发现 `private static class WebResourceRequestImpl implements WebResourceRequest ` 它的内部实现仅仅是一个单纯的实体。那这个东西要替换就非常好办了，三个方法都可以做：

1. 动态代理
2. 反射
3. 重新实现

## 实现

方案确定了，剩下的就简单了。直接上代码。

首先是往URL字符串加那个标志位的方法

```
public static String injectIsParams(String url) {
    if (url != null && !url.contains("xxx=") {
        if (url.contains("?")) {
            return url + "&xxx=1";
        } else {
            return url + "?xxx=1";
        }
    } else {
        return url;
    }
}
```

然后要拦截所有请求了

```
webView.setWebViewClient(new WebViewClient() {

    @SuppressLint("NewApi")
    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
        if (request != null && request.getUrl() != null && request.getMethod().equalsIgnoreCase("get")) {
            String scheme = request.getUrl().getScheme().trim();
            if (scheme.equalsIgnoreCase("http") || scheme.equalsIgnoreCase("https")) {
                try {
                    URL url = new URL(injectIsParams(request.getUrl().toString()));
                    URLConnection connection = url.openConnection();
                    return new WebResourceResponse(connection.getContentType(), connection.getHeaderField("encoding"), connection.getInputStream());
                } catch (MalformedURLException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }


    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
        if (!TextUtils.isEmpty(url) && Uri.parse(url).getScheme() != null) {
            String scheme = Uri.parse(url).getScheme().trim();
            if (scheme.equalsIgnoreCase("http") || scheme.equalsIgnoreCase("https")) {
                try {
                    URL url = new URL(injectIsParams(request.getUrl().toString()));
                    URLConnection connection = url.openConnection();
                    return new WebResourceResponse(connection.getContentType(), connection.getHeaderField("encoding"), connection.getInputStream());
                } catch (MalformedURLException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }

});
```

大功告成。

欢迎指出代码中的问题~~一起学习进步

> **注意:** 注意保护 URL 的 Scheme，在代码中特地过滤了 http 和 https。

## 引申

上边的 API 中发现还能有更多的玩法，比如：

* 替换 `WebResourceResponse`，构造一个自己的 `WebResourceResponse`。比如下列代码，用一个包里的本地文件替换掉要请求的网络图片。

```
WebResourceResponse response = null;
if (url.contains("logo")) {
    try {
        InputStream is = getAssets().open("test.png");
        response = new WebResourceResponse("image/png", "UTF-8", is);
    } catch (IOException e) {
        e.printStackTrace();
    }        
}
return response;
```

* 在 API 21 (5.0) 以上的版本使用了 `WebResourceRequest` 接口，这个接口能修改发出请求的 Header

```
@Override
public Map<String, String> getRequestHeaders() {
    return request.getRequestHeaders();
}
```

* 在 API 21 (5.0) 以上的版本中可以区分 GET 请求和 POST 请求，在某些情况下，需要区分 AJAX 的不同种类请求的时候可以用到。

* 更多敬请期待


