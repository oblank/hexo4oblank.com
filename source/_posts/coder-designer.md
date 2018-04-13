---
title: React Native 入门问题集
date: 2018-04-09 18:17:53
tags:
  - 设计
  - 程序员
categories:
  - 开发
---
记录一些自己入门遇到的问题与解决方法


## 怎么将navigation传递到子级组件？
当需要在子组件中使用`navigation`导航时，一种方法是直接在引入组件时将 navigation 作为props传递，另一种是使用 `withNavigation`，后者更优雅。

- 方法一：引用时传递	
```
<Component navigation={this.props.navigation} ...otherProps />
```
- 方法二：使用 `withNavigation`， [参见官方文档](https://reactnavigation.org/docs/connecting-navigation-prop.html)
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
## iOS真机、模拟器调试兼容网络问题
在RN生成的XCode项目里，默认是通过远程方式调试，只要手机、模拟器处于同一网络即可，但我们的手机的网络极有可能会切换网络（譬如上下班、吃饭、上厕所...等等），而这会导到远程调试的应用无法打开，可以兼容处理一下。

打开XCode项目，修改 `AppDelegate.m`，将原有的
```
jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
```
修该成
```
#if TARGET_IPHONE_SIMULATOR
  //remote debug
 jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
 #else
  // release
 jsCodeLocation = [[NSBundle  mainBundle] URLForResource:@"main"  withExtension:@"jsbundle"];
 #endif
```