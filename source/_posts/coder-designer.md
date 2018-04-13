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
一种方法是直接在引入组件时将 navigation 作为props传递，另一种是使用 `withNavigation`，后者更优雅。

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

// withNavigation returns a component that wraps MyBackButton and passes in the  
// navigation prop  
export default withNavigation(Component);
```
然后在引入时更直接：
```
<Component ...otherProps />
```

