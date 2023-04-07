## uniapp如何操作DOM
### renderjs
#### 1.说明：
* renderjs是一个运行在视图层的js。它比WXS更加强大。它只支持app-vue和web。  
#### 2.作用：
* 大幅降低逻辑层和视图层的通讯损耗，提供高性能视图交互能力
* 在视图层操作dom，运行 for web 的 js库
#### 3.平台差异说明：
<table border>
	<tr>
		<th>APP</th>
		<th>H5</th>
    <th>微信小程序</th>
   	<th>支付宝小程序</th>  			
   	<th>百度小程序</th>  			
   	<th>字节跳动小程序</th>  			
   	<th>飞书小程序</th>  			
   	<th>QQ小程序</th>  			
	</tr>
	<tr>
    	<td>√(2.5.5+，仅支持vue)</td>
    	<td>√</td>
    	<td>×</td>
    	<td>×</td>
    	<td>×</td>
    	<td>×</td>
    	<td>×</td>
    	<td>×</td>
   </tr>
</table>  

* nvue的视图层是原生的，无法运行js。但提供了bindingx技术来解决通信阻塞。  
* 微信小程序下替代方案是wxs，这是微信提供的一个裁剪版renderjs 。 
* web下不存在逻辑层和视图层的通信阻塞，也可以直接操作dom，所以在web端使用renderjs主要是为了跨端复用代码。如果只开发web端，没有必要使用renderjs。  
####  4.使用方式：
```html
<template>
  <view>
    <!-- test 指的是script标签中module的值，可自定义 -->
    <button @click="test.renderHandleClick" >直接触发renderjs点击</button>
    <button @click="serviceHandleClick">直接触发service点击</button>
    <button 
      @click="test.serviceToRenderjsHandleClick" 
      :msg="msg" 
      :change:msg="test.ToRenderjsHandleClick"
    >通过service触发renderjs点击</button>
    <button @click="test.renderjsToServiceHandleClick" >通过renderjs触发service点击</button>
  </view>
<template>
<script>
	export default {
    data(){
      return{
        msg:null
      }
    },
    methods:{
      serviceHandleClick(){
        // 直接触发service点击
      },
      serviceToRenderjsHandleClick(){
        /**
         * 通过service触发renderjs点击
         * 1.在标签中添加 :msg="msg" :change:msg="test.ToRenderjsHandleClick"
         * msg 是 data中自定义的; ToRenderjsHandleClick 是需要触发renderjs中需要触发的事件名称
         * 2.修改data中定义的 msg 触发renderjs中的事件
         */
        this.msg = Math.random() || {a:'xxx',b:'xxx'}
      },
      ToServiceHandleClick(data){
        // 这是从renderjs触发service中的方法
        // data 为传递过来的值
      },
    }
  }
</script>
<script module="test" lang="renderjs">
	export default {  
		methods: {
            renderHandleClick(event, ownerInstance){
              // 直接触发renderjs点击
              // event是事件对象 获取传入的参数        
              // ownerInstance和this.$ownerInstance是一样的，用来调用service层的方法 
              
            },
            renderjsToServiceHandleClick(event, ownerInstance){
              /**
                * ownerInstance: 第一个参数是方法名，第二个参数是传过去的参数
                * 方式一：ownerInstance.callMethod('ToServiceHandleClick', data) 
                * 方式二：this.$ownerInstance.callMethod('ToServiceHandleClick', data)
                */
              ownerInstance.callMethod('ToServiceHandleClick', event) 
              this.$ownerInstance.callMethod('ToServiceHandleClick', event)
            },
            ToRenderjsHandleClick(newValue, oldValue, ownerVm){
              // newValue 修改后的service中定义的msg值
              // oldValue 修改前的service中定义的msg值
              // ownerVm  用来调用service层的方法 
            }
		}
	}
</script>

```
#### 5.注意事项：
* renderjs 需要在原先的script标签的同级新增一个script，设置lang=renderjs，module=(值任意，相当于命名空间，之后会根据这个名字调用其中的方法)  
* 新script标签内的结构和之前的几乎一致，有几点不同的需要注意：  
 1.生命周期不和uniapp相同，而是和vue相同，onLoad应该写成原生vue的created  
 2.官方文档好像说了renderjs中无法使用uni这个全局变量，具体哪个地方忘了。实测结果是：部分可以。例如uni.upx2px是可以用的，uni.request不可以，所以使用uni全局变量之前先输出看一下有没有  
* 目前仅支持内联使用。
* 不要直接引用大型类库，推荐通过动态创建 script 方式引用。
* 可以使用 vue 组件的生命周期(不支持 beforeDestroy、destroyed、beforeUnmount、unmounted)，不可以使用 App、Page 的生命周期
* 视图层和逻辑层通讯方式与 WXS 一致，另外可以通过 this.$ownerInstance 获取当前组件的 ComponentDescriptor 实例。
* 注意逻辑层给数据时最好一次性给到渲染层，而不是不停从逻辑层向渲染层发消息，那样还是会产生逻辑层和视图层的多次通信，还是会卡
* 观测更新的数据在视图层可以直接访问到。
* APP 端视图层的页面引用资源的路径相对于根目录计算，例如：./static/test.js。
* APP 端可以使用 dom、bom API，不可直接访问逻辑层数据，不可以使用 uni 相关接口（如：uni.request）
* H5 端逻辑层和视图层实际运行在同一个环境中，相当于使用 mixin 方式，可以直接访问逻辑层数据。