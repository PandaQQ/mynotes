# React Native - 在浏览器中打开连接
> 需求描述： 就是希望在react-native中显示一个html文件，然后点击里面的<a>的连接之后会打开该用户的浏览器。

### 方案1: 并不行，因为 this.webview.stopLoading(); 在Android还是会影响原先 webview的跳转。

```
import React, { Component } from 'react';
import { WebView, Linking } from 'react-native';

export default class WebViewThatOpensLinksInNavigator extends Component {
  render() {
    const uri = 'http://stackoverflow.com/questions/35531679/react-native-open-links-in-browser';
    return (
      <WebView
        ref={(ref) => { this.webview = ref; }}
        source={{ uri }}
        onNavigationStateChange={(event) => {
          if (event.url !== uri) {
            this.webview.stopLoading();
            Linking.openURL(event.url);
          }
        }}
      />
    );
  }
}
```

### 方案2: 并不行，无法收到js的通知
```
import {
  View,
  WebView,
  Linking,
} from 'react-native';

const injectScript = `
  (function () {
    window.onclick = function(e) {
      e.preventDefault();
      window.postMessage(e.target.href);
      e.stopPropagation()
    }
  }());
`;

class MyWebView extends React.Component {

  onMessage({ nativeEvent }) {
    const data = nativeEvent.data;

    if (data !== undefined && data !== null) {
      Linking.openURL(data);
    }
  }

  render() {
    return (
      <WebView
        source={{ html: this.props.html }}
        injectedJavaScript={injectScript}
        onMessage={this.onMessage}
      />
    )
  }
}
```

### 方案3: 居然可以！！！！
> 使用“ stopLoading（）”的一个大问题是，在Android上，它会禁用该源页面上任何其他链接的进一步点击。
> WebView组件正在从核心RN中分离出来，并交到社区的手中。如果改用该版本（https://github.com/react-native-community/react-native-webview），则可> 以在iOS和Android上都使用“ onShouldStartLoadWithRequest”道具，这使它更加优雅。
> 以Damien Varron真正有用的答案为基础，下面是一个示例，说明如何利用该道具避免在跨平台工作的stopLoading：

```
onShouldStartLoadWithRequest={event => {
    if (event.url !== uri) {
        Linking.openURL(event.url)
        return false
    }
    return true
}}

```
> 如果您的源是HTML而不是URI，则可以按照以下方法操作：
```
onShouldStartLoadWithRequest={event => {
    if (event.url.slice(0,4) === 'http') {
        Linking.openURL(event.url)
        return false
    }
    return true
}}

```
> 您也可以用这种方式（我已经添加了about：blank部分）。我计划进行更多的反复试验，看看哪种方法最有效。

```
onShouldStartLoadWithRequest={event => {
    if (!/^[data:text, about:blank]/.test(event.url)) {
        Linking.openURL(event.url)
        return false
    }
    return true
}}
```
