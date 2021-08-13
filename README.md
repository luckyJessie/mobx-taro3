# mobx-taro3
mobx+taro3 demo


import React from 'react';
import ReactDOM from 'react-dom';
import { View } from '@tarojs/components';
import { observable, action, autorun, computed, configure, runInAction, when, reaction }  from 'mobx';
import { observer } from 'mobx-react'

class Store {
  // @observable 定义属性可观察
  // action 装饰器/函数遵循 javascript 中标准的绑定规则。 但是，action.bound 可以用来自动地将动作绑定到目标对象。 注意，与 action 不同的是，(@)action.bound 不需要一个name参数，名称将始终基于动作绑定的属性。
  @observable count = 3
  @observable price = 10

  @action.bound increment () {
    this.count++
  }
  @computed get totalPrice () {
    return this.count * this.price
  }

  @action changeCount () {
    this.count = 10
    this.price = 3
  }

  // @action.bound asyncChangeCount () { // 严格模式下这种方式会报错，应该为下面3种的写法
  //   setTimeout(() => {
  //     this.count = 100
  //   }, 100);
  // }
  // 方法1
  @action.bound asyncChangeCount2 () {
    setTimeout(() => {
      this.changeCount2()
    }, 100);
  }

  @action.bound changeCount2 (value = 20) {
    this.count = value
  }

  // 方法2
  @action.bound asyncChangeCount3 () {
    setTimeout(() => {
      action('changeCount3', () => {
        this.count = 100
      })()
    }, 100);
  }

  // 方法3
  @action.bound asyncChangeCount4 () {
    setTimeout(() => {
      runInAction(() => {
        this.count = 100
      })
    }, 100);
  }

}

// 对oberverable作出数据响应：
// 1.autorun: 默认会执行一次，当被观察的数据被次改变都再执行
const stores = new Store()
autorun(() => {
  console.log(stores)
})
// 2.when: 当count > 10时，只执行一次逻辑
when(() => {
  return stores.count >= 20
}, () => {
  console.log('stores.count', stores.count)
})
// 3. reaction: 只有当被观测的数据被改变时才会执行
reaction(() => {
  // 执行一些业务代码，返回下一个函数需要使用的数据
  return stores.count
}, (data, reaction) => {
  console.log('count', data);
  // 手动解除reaction的监听
  reaction.dispose()
})


// 限制只能通过actions修改值
configure({
  'enforceActions': 'observed'
})

// stores.count = 10
stores.changeCount()

// runInAction
runInAction(() => {
  stores.count = 10
})

// 异步action
stores.asyncChangeCount()


@observer
class App extends React.Component{
     render () {
        const { store1 }= this.props
        return <View>
            {/* 这里@computed只执行了一次 */}
            <View>{store1.totalPrice}</View>
            <View>{store1.totalPrice}</View>
            <View>{store1.totalPrice}</View>
          </View>
    }

}

ReactDOM.render(<App store1={new Store()} />, document.getElementById('root'))
