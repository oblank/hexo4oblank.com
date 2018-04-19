---
title: React Native 入门问题集
date: 2018-04-09 18:17:53
tags:
  - ReactNative
  - 程序员
categories:
  - 开发
---
记录一些自己入门遇到的问题与解决方法


## 怎么将navigation传递到子级组件？
当需要在子组件中使用`navigation`导航时，一种方法是直接在引入组件时将 navigation 作为props传递，另一种是使用 `withNavigation`，后者更优雅。

### 方法一：引用时传递	
```
<Component navigation={this.props.navigation} ...otherProps />
```
### 方法二：使用 `withNavigation`， [参见官方文档](https://reactnavigation.org/docs/connecting-navigation-prop.html)
```
import React from 'react';  
import { Button } from 'react-native';  
import { withNavigation } from 'react-navigation';  

class Component extends React.Component {  
	render() {  
		return <Button title="Back" onPress={() => { this.props.navigation.goBack() }} />;
	}  
}  

// 用withNavigation包一下
export default withNavigation(Component);
```
然后在引入时更直接：
```
<Component ...otherProps />
```
## RN在iOS真机、模拟器调试时网络兼容问题
在RN生成的XCode项目里，默认是通过远程方式调试，只要手机、模拟器处于同一网络即可，但我们的手机极有可能会切换网络（譬如上下班、吃饭、上厕所...等等），而这会导到远程调试的应用无法打开，可以兼容处理。

打开XCode项目，修改 `AppDelegate.m`，将原有的
```
jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
```
修改成
```
#if TARGET_IPHONE_SIMULATOR
  // remote debug
 jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
 #else
  // release
 jsCodeLocation = [[NSBundle  mainBundle] URLForResource:@"main"  withExtension:@"jsbundle"];
 #endif
```
这会将真机像发布时一样把文件打包进APP，不受网络切换影响

## 获取组件的屏幕位置
当使用`react-native-modal-popover`组件时，有一个参数`fromRect`用来指明组件的位置，这时候我们取要获取组件的位置，使用`NativeModules.UIManager.measure`
```
import React, { Component } from 'react'; 
import {
  StyleSheet,
  Text,
  View,
  TouchableOpacity,
  findNodeHandle,
  NativeModules 
} from 'react-native';
import Popover, { PopoverTouchable } from 'react-native-modal-popover';

....

const handle = findNodeHandle(this.refs[ 'popoverAnchor' ]); 
if (handle) {
    NativeModules.UIManager.measure(handle, (x0, y0, width, height, x, y) => {
    this.setState({
		popoverAnchor: { x, y, width, height },
	});
  }); 
}

....

<SimpleLineIcons
    name="options"
    ref="popoverAnchor"
    ...otherProps
/>
<Popover
    visible={this.state.showPopover}
    fromRect={this.state.popoverAnchor}
    ...otherProps
>
    {children}
</Popover>
```

## 基于webview实现简易浏览器
在APP内经常需要打开一些URL链接，如果通过外部浏览器打开、返回体验不好，一般会做一个内部的导航，用RN的WebView组件很容易实现，主要是利用`onNavigationStateChange`获得各种状态并处理各导航菜单的表现，同时利用`navigation.setParams`动态设置浏览器的标题栏。

```
...
goBack() {
    this.refs[ WEB_VIEW_REF ].goBack(); }

goForward() {
    this.refs[ WEB_VIEW_REF ].goForward(); }

reload() {
    self.refs[ WEB_VIEW_REF ].reload(); }

openShare() {
    Share.share({ message: 'content', title: 'testshare' })
        .catch((error) => console.error(error)); }

openWithSafari() {
    Linking.openURL(this.state.currentUrl)
        .catch(err => console.error('An error occurred', err)); }

onNavigationStateChange(navState) {
	this.setState({
	  backButtonEnabled : navState.canGoBack,
	  forwardButtonEnabled: navState.canGoForward,
	  currentUrl : navState.url,
	  status : navState.title,
	  loading : navState.loading,
	  scalesPageToFit : true
	});

	// 动态更新 headerTitle
	if (navState.title) {
		this.props.navigation.setParams({ title: navState.title })
	}
}

...

<WebView
    ref={WEB_VIEW_REF}
    source={{ uri: url }}
    style={styles.webView}
    scalesPageToFit={true}
    startInLoadingState={true}
    onNavigationStateChange={(e) => this.onNavigationStateChange(e)}
/>
<View style={styles.toolbar}>
  {/*退回*/}
  {
	!this.state.backButtonEnabled && (
		<View style={[ styles.toolBtn ]}>
			<SimpleLineIcons name='arrow-left' size={18} style={[ styles.headerIcon, styles.headerIconDisable ]}/>
		</View>  )
    }
    {
        this.state.backButtonEnabled && (
            <TouchableOpacity style={styles.toolBtn} onPress={() => this.goBack()}>
		<SimpleLineIcons name='arrow-left' size={18} style={[ styles.headerIcon ]}/>
	    </TouchableOpacity>  )
    }
...other buttons
</View>
```

## 使用`react-navigation`的`TabNavigator`时Tab之间跳转不能正常返回的情况

如果TabA、TabB同时是`StackNavigator`，当从一个TabA跳到另一个TabB里的某个路由时，返回时常常会处于TabB中而无法正常返回到TabA，一种方法是修改顶部左侧菜单自定义实现返回逻辑，但这种情况下右滑返回手势仍然不能正常返回到TabA。

换种思维，直接将TabA要跳转的页面引入到相应的Tab路由中，代码仍然是一套，只是配了两个路由，路由名字略微不同即可:
```
export const RouteHome = StackNavigator({
  Home : { screen: Views.Home },
  HBrowser: { screen: Views.Browser },
  Scanner : { screen: Views.Scanner }, 
})

export const RouteSetting = StackNavigator({
  Setting : { screen: Views.Home },
  Browser: { screen: Views.Browser },
  ScannerB : { screen: Views.Scanner }, 
})
```
上面的 `HBrowser`、`Browser`用了相同的组件，但路由名不同，在`Home`路由中按需跳转`HBrowser`，在`Setting`中就按需跳转`Browser`，这样兼顾了代码复用与交互的问题。

```
......
<Hyperlink
  onPress={ (url, text) => {
        if (navigation.state && navigation.state.routeName === 'Home') {
            this.props.navigation.navigate('HBrowser', {url: url, text: text})
        } else {
            this.props.navigation.navigate('Browser', {url: url, text: text})
        }
    }}
    linkStyle={styles.url}
>
 <Text style={styles.content} selectable={true} selectable={true}>
  {item.content}
 </Text> 
</Hyperlink>
......
```
**UPDATE:** 经测试，不同StackNavigator里可以包含同名路由，导航时会优先从该Stack里寻找路由，所以上面的路由在取名时不需要刻意规避，简化如下：

```
const BaseRotes = {
  Browser: { screen: Views.Browser },
  Scanner : { screen: Views.Scanner }, 
};

export const RouteHome = StackNavigator({
  Home : { screen: Views.Home },
  ...BaseRotes, 
})

export const RouteSetting = StackNavigator({
  Setting : { screen: Views.Home },
  ...BaseRotes, 
})
```
导航时不再区分路由名：
```
......
<Hyperlink
  onPress={ (url, text) => {
	this.props.navigation.navigate('Browser', {url: url, text: text})
  }}
  linkStyle={styles.url}
>
 <Text style={styles.content} selectable={true} selectable={true}>
  {item.content}
 </Text> 
</Hyperlink>
......
```
