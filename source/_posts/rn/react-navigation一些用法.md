---
title: react-navigation一些用法
tags: [react-navigation]
---
**react-navigation 2.14.2 版本**
## 创建底部按钮
创建底部按钮时一开始设置的是静态的，然后导出去的，当初始化成功后，再要改变底部的内容的时候，可以在当前App界面设置参数
如
```js
// App界面
componentDidMount() {
    let {msg, navigation} = this.props
    navigation.setParams({data: msg.data})
    // 1.设置data的参数，可传给 BottomTabs
  }
```
```js
const BottomTabs = createBottomTabNavigator({
    Message: {
        screen: App,
        navigationOptions: ({navigation}) => {
            // 2.可获取传过来的参数
            let msg = navigation.state.params ? navigation.state.params.data : "消息"
            return ({
            title: msg,
            tabBarIcon: ({ focused }) => {
                // focused当前底部组件是否被被聚焦
                return (<Image source={focused ? require('./../../static/background/msg1.png') : require('./../../static/background/msg.png')} style={styles.foIcons}></Image>)
            }
        })}
    }})
export default BottomTabs
```
创建路由
```js
const NavigatorSendMsg = createStackNavigator({
    SendMsg: { screen: SendMsg },
})
const NavigatorEntry = createStackNavigator({
    BottomTabs: { screen: BottomTabs},
    NavigatorSendMsg: {screen: NavigatorSendMsg},  // NavigatorSendMsg为默认的路由名称
}, {
    headerMode: 'none'
});
render(
  <View><NavigatorEntry/></View>
)
```
在界面设置顶部导航栏状态
```js
// 此处与componentDidMount同级
static navigationOptions = ({ navigation }) => {
  // 同理可在componentDidMount里面用props.navigation.setParams设置参数，然后在此处的navigation里面获取
      return {
          headerTitle: <Header
              comCenter={()=>{
                  return <Text>
                  发送消息
                  </Text>}
              }
              boxStyle= {style.fdDox} // 整体样式
          ></Header>,
          headerStyle: {// 顶部栏高度
              height: 85,
          }
      }
  }

```
