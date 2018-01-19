```java
public static String setHtmlCotentSupportImagePreview(String body) {
    // 读取用户设置：是否加载文章图片--默认有wifi下始终加载图片
    if (AppContext.get(AppConfig.KEY_LOAD_IMAGE, true)
            || TDevice.isWifiOpen()) { 
        // 过滤掉 img标签的width,height属性
        body = body.replaceAll("(<img[^>]*?)\\s+width\\s*=\\s*\\S+", "$1");
        body = body.replaceAll("(<img[^>]*?)\\s+height\\s*=\\s*\\S+", "$1");
        // 添加点击图片放大支持
        // 添加点击图片放大支持
        body = body.replaceAll("(<img[^>]+src=\")(\\S+)\"",
                "$1$2\" onClick=\"showImagePreview('$2')\"");
    } else {
        // 过滤掉 img标签
        body = body.replaceAll("<\\s*img\\s+([^>]*)\\s*>", "");
    }
    return body;
}
```