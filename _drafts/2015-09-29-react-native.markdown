---
layout: post
title: 'React Native 介紹'
date: 2015-09-29 05:30:00
categories: Javascript, React, Mobile
---

React Native 讓我們可以在原生平台上(iOS, Android)使用基於 Javascript 與 React 一致的開發經驗來建立世界級應用程式
React Native 專注在讓開發者有效率的實現跨平台開發, 但注意它不是屬於 write once, run anywhere 即一份程式碼共用在所有平台, 而是 learn once, write anywhere. 先別急著對這樣的方式下結論, 認真思考一下事實上你的確很難做到一份程式碼共用, 最大的原因應屬如果你要打造世界級的應用程式你就需要注重不同裝置使用者的 UXUI 體驗, 至少在介面上你不容易做到只寫一次卻讓所有使用者擁有很好的體驗.

現在, Facebook 已經使用 React Native 在多個產品上並且會持戲投入 React Native


# 原生組件 Native Components

透過 React Native 你可以使用標準的原生平台組件元件例如 iOS 的 UITabBar 以及 Android 的 Drawer 元件.
這麼一來你的程式就會具備與該平台一致的使用體驗. 這些元件可以很輕易的使用, 在程式中透過 React 風格來組織. 舉例 TabBarIOS 和 DrawerLayoutAndroid

{% highlight js %}
// iOS

var React = require('react-native');
var { TabBarIOS, NavigatorIOS } = React;

var App = React.createClass({
  render: function() {
    return (
      <TabBarIOS>
        <TabBarIOS.Item title="React Native" selected={true}>
          <NavigatorIOS initialRoute={{title: 'React Native'}} />
        </TabBarIOS.Item>
      </TabBarIOS>
    );
  }
})
{% endhighlight %}

{% highlight js %}
// Android

var React = require("react-native");
var { DrawerLayoutAndroid, ProgressBarAndroid } = React;

var App = React.createClass({
  render: function() {
    return (
      <DrawerLayoutAndroid
        renderNavigationView={() => <Text>React Native</Text>}
      >
        <ProgressBarAndroid />

      </DrawerLayoutAndroid>
    );
  }
})
{% endhighlight %}

# 非同步執行

所有操作在 javascript 程式碼與原生平台之間都是非同步執行, 原生的模組也可以採用額外的執行緒. 上面這段描述表示的是我們可以非主線執行緒上解析圖片
在背景程序另存檔案, 測量文字計算佈局而不會中止介面上的操作
最後的結果就是 React Native 自然流暢的互動反饋. 關於資料傳輸方面也可以完全序列化, 意為我們可以利用 Chrome 開發者工具來替 Javascript 的部分偵錯

# 觸控處理

React Native 實作了強大的 Responder Chain 系統來處理複雜視圖結構上的觸控行為並且提供高度抽象的元件, 例如 `TouchableHighlight` 可以良好的與 scroll view 以及其他元素整合而不需要額外的設定 

{% highlight js %}
var TouchDemo = React.createClass({
  render: function() {
    reutrn (
      <ScrollView>
        <TouchableHighlight onPress={() => console.log('pressed')}>
          <Text>Proper Touch Handling</Text>
        </TouchableHighlight>
      </ScrollView>
    );
  }
});
{% endhighlight %}

# Flexbox 樣式

關於佈局應該要輕鬆簡單, 這也是為什麼開發團隊採用了 `flexbox layout model`. Flexbox 讓佈局變得單純簡單
React Native 同時也支援一般的網頁樣式例如 `fontWeight`, 在這個部分 StyleSheet 物件可以協助我們去定義那些樣式, 就像我們在使用 CSS 一樣.

{% highlight js %}
// iOS & Android

var React = require('react-native');
var { Image, StyleSheet, Text, View } = React;

var ReactNative = React.createClass({
  render: function() {
    return (
      <View style={styles.row}>
        <Image
          source={{uri: 'http://facebook.github.io/react/img/logo_og.png'}}
          style={styles.image}
        />
        <View style={styles.text}>
          <Text style={styles.title}>
            React Native
          </Text>
          <Text style={styles.subtitle}>
            Build high quality mobile apps using React
          </Text>
        </View>
      </View>
    );
  },
});
var styles = StyleSheet.create({
  row: { flexDirection: 'row', margin: 40 },
  image: { width: 40, height: 40, marginRight: 10 },
  text: { flex: 1, justifyContent: 'center'},
  title: { fontSize: 11, fontWeight: 'bold' },
  subtitle: { fontSize: 10 },
});
{% endhighlight %}

# Polyfills

React Native 的重點是改變在介面程式碼上的撰寫方式, 剩下的部分我們期望就直接使用 web 通用的標準以及相容支援(polyfill)那些 API.
你可以使用 npm 來安裝 js 的函式庫將其結合進 React Native 例如, XMLHttpRequest, window.requestAnimationFrame, navigator.geolocation
官方開發團隊也不斷的在增加可使用的 API, 並且期待開放原始碼社群也能夠為其貢獻

{% highlight js %}
var React = require('react-native');
var {Text} = React;

var GeoInfo = React.createClass({
  getInitialState: function() {
    return {position: 'unknown'};
  },
  componentDidMount: function() {
    navigator.geolocation.getCurrentPosition(
      (position) => this.setState({position}),
      (error) => console.error(error)
    );
  },
  render: function() {
    return (
      <Text>
        Position: {JSON.stringify(this.state.position)}
      </Text>
    );
  }
});
{% endhighlight %}

# 可擴展性

我們肯定能夠建立一個優質的 App 使用 React Native 卻不需要轉寫任何一行原生的程式碼, 不過 React native 也提供了簡單的方式來擴展客制的原生模組和視圖(views) - 意為我們可以重複使用任何我們已經建置好的原生模組將其匯入

## iOS 模組

如果要建立一個簡單的 iOS 模組, 只要建立一個新的類別並實作 `RCTBridgeModule` 協定, 然後將你希望在 JS 中使用的 function 包在 `RCT_EXPORT_METHOD` 即可 

{% highlight obj-c %}
#import "RCTBridgeModule.h"

@interface MyCustomModule : NSObject <RCTBridgeModule>
@end

@implementation MyCustomModule
RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(processString:(NSString *)input callback:(RCTResponseSenderBlock)callback)
{
  callback(@[[input stringByReplacingOccurrencesOfString:@"Goodbye" withString:@"HELLO"]]);
}
@end
{% endhighlight %}

{% highlight js %}
var React = require('react-native');
var {NativeModules, Text} = React;

var Message = React.createClass({
  getInitialState() {
    return { text: 'Goodbye world'}
  },
  componentDidMount() {
    NativeModule.MyCustomModule.processString(this.state.text, (text) => {
      this.setState({text});
    });
  },
  render: function() {
    return (
      <Text>{this.state.text}</Text>
    );
  }
});
{% endhighlight %}

## 建置 iOS 視圖

自訂 iOS views 則透過 RCTViewManager 子類別的方式, 實作一個 `-(UIVIew *) view` 方法然後透過 `RCT_EXPORT_VIEW_PROPERTY` 匯出屬性

{% highlight obj-c %}
#import "RCTViewManager.h"

@interface MyCustomViewManager : RCTViewManager
@end

@implementation MyCustonViewManager
  RCT_EXPORT_MODULE()

  -(UIView *) view
  {
    return [[MyCustomView alloc] init];
  }

  RCT_EXPORT_VIEW_PROPERTY(myCustomProperty, NSString);
@end
{% endhighlight %}

{% highlight js %}
var React = require('react-native');
var { requireNativeComponent } = React;

class MyCustomView extends React.Component {
  render() {
    return (
      <NativeMyCustomView {...this.props} />
    );
  }
}

MyCustomView.propTypes = {
  myCustomProperty: React.PropTypes.oneOf(['a', 'b'])
};

var NativeMyCustomView = requireNativeComponent('MyCustomView', MyCustomView); 
module.exports = MyCustomView;
{% endhighlight %}

## Android 模組

同樣的, Android 也支援自訂擴充模組, 不過方始有一點不同.

要建立一個簡單的 Android 模組, 一樣先建立一個新的類別繼承 `ReactContextBaseJavaModule` 類別
並且透過 `@ReactMethod` 註記您想要在 JS 中使用的方法, 此外類別本身必須要註冊到 `ReactPackage` 裡面

{% highlight java %}

public class MyCustonModule extends ReactContextBaseJavaModule {
  // Available as NativeModules.MyCustomModule.processString
  @ReactMethod 
  public void processString(String input, Callback callback) { 
    callback.invoke(input.replace("Goodbye", "Hello")); 
  }
}
{% endhighlight %}

{% highlight js %}
var React = require('react-native');
var { NativeModules , Text} = React;

var Message = React.createClass({
  getInitialState() {
    return {text: 'Goodbye World'};
  },
  componentDidMount() {
    NativeModules.MyCustomModule.processString(this.state.text, (text) => {
      this.setState({text});
    });
  },
  render() {
    return (
      <Text>{this.state.text}</Text>
    );
  }
})
{% endhighlight %}

## 自訂 Android 視圖

自訂 Android views 則透過繼承 SimpleViewManger 實作 `createViewInstance` 和 `getName` 方法並且透過 `@UIProp` 來匯出屬性

{% highlight java %}
public class MyCustonViewManager extends SimpleViewManager<MyCustonView> {
  private static final String REACT_CLASS = "MyCustomView";

  @UIProp(UIProp.Type.STRING)
  public static final String PROP_MY_CUSTOM_PROPERTY = 'myCustonProperty';

  @Override
  public String getName() { 
    return REACT_CLASS; 
  } 

  @Override protected MyCustomView createViewInstance(ThemedReactContext reactContext) { 
    return new MyCustomView(reactContext); 
  } 

  @Override public void updateView(MyCustomView view, CatalystStylesDiffMap props) { 
    super.updateView(view, props); 
    if (props.hasKey(PROP_MY_CUSTOM_PROPERTY)) { 
      view.setMyCustomProperty(props.getString(PROP_MY_CUSTOM_PROPERTY)); 
    } 
  }
}
{% endhighlight %}

{% highlight js %}
var React = require('react-native');
var {requireNativeComponent} = React;

class MyCustomView extends React.Component {
  render () {
    return (
      <NativeMyCustomView {...this.props} />
    );
  }
}

MyCustomView.propTypes = {
  myCustomProperty: React.PropTypes.oneOf(['a', 'b'])
};

var NativeMyCustomView = requireNativeComponent('MyCustomView', MyCustomView); 
module.exports = MyCustomView;
{% endhighlight %}