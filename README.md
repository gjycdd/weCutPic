### 微信小程序——插件之图片裁剪

> 实现原理利用：利用微信画布提供的接口进行封装<br />
time 2017/9/26

#### <a href='https://github.com/we-plugin/we-cropper'>源码来源 </a>

#### 修改 

> 不传参时出现图片过大的问题

#### 精简

> 仅仅保修基本的图片裁剪功能，专注裁剪

#### 基础
图片的插入有两种方式(图片初始大小建议2M以内)
- 通过初始化时在参数中配置```src```<br />
```
new weCropper({
    // ...
    src: '...',
    // ...
})
```
- 采用惰性载入的方式：点击上传图片按钮后获取图片地址，并通过pushOrign方法载入图片<br />
```
const src = [img path]
this.wecropper.pushOrign(src)
```

#### 安装和使用

> 使用方法：
	
  1. git clone 当前项目
  2. 将项目中的三个文件分别复制到你对应的文件目录下<br />
> ```tips:尽量不要复制到根目录下，保持项目的整洁、可维护性```

  3. 在需要引入的wxml文件中执行如下代码<br />	
	  ```<import src="[your dir]/weCutPic.wxml"/>```<br />
	  并且将```<template is="weCropper" data="{{...cropperOpt, dis}}"/>```使他作为页面最外层结点<br />
> 注意：```[your dir]```是你存放```weCutPic.wxml```文件的路径<br />
	  ```<import src="xxx.wxml"/>```是小程序引入模板的方式[<a href="https://mp.weixin.qq.com/debug/wxadoc/dev/framework/view/wxml/template.html">详情</a>]

  4. 在引入的wxml文件对应wxss文件中执行如下代码<br />
	  ```@import '[your dir]/weCutPic.wxss';```<br />
> 注意：```[your dir]```是你存放```weCutPic.wxss```文件的路径<br />

  5. 以上步骤完成后，在对应的js文件中引入```weCutPic.js```<br />
> es5  ```const weCropper = require('[your dir]/weCutPic.js')```<br />
es6  ```import weCropper from '[your dir]/utils/weCutPic.js'```<br />
注意：```[your dir]```同上文
    
  6. 在js中获取设备信息(长宽)<br />
  	```const device = wx.getSystemInfoSync() // 获取设备信息```<br />
	```const width = device.windowWidth // 获取屏幕可使用窗口宽度```<br />
	```const height = device.windowHeight; // 获取屏幕可使用窗口高度```<br />
> 注意：6步骤需放在···page···外执行，请在完成···1-5···步骤后执行

  7. 为裁切设置参数
> 我们都知道在小程序中每个页面就是一个模块<br />
这些模块都有自己的数据在```page```中的```data```对象内<br />
因此在data中我们设定裁切的参数

```
{
	dis: 'none', //控制裁切的显示和隐藏支持none/block
	cropperOpt: {
      id: 'myCropper', // 画布id
      width,  // 画布宽度
      height, // 画布高度
      scale: 2.5, // 最大缩放倍数
      zoom: 8, // 缩放系数
      cut: {
        x: (width - 300) / 2, // 裁剪框x轴起点
        y: (width - 300) / 2, // 裁剪框y轴期起点
        width: 300, // 裁剪框宽度
        height: 300 // 裁剪框高度
      }
    }
}
```
写好以上数据，我们去```onShow```中初始化，让他每次打开页面都初始化画布

  8. 初始化裁切画布
> 初始化裁切方法1<br />
推荐onLoad时初始化，如有需要可在onShow初始化<br />
若同一个页面只有一个裁剪容器，在其它Page方法中可通过this.wecropper访问实例
```
onLoad: function () {
	const { cropperOpt } = this.data;
	new weCropper(cropperOpt)
	.on('ready', (ctx) => { // 回调函数，canvas初始化后执行
	  console.log(`wecropper is ready for work!`)
	})
	.on('beforeImageLoad', (ctx) => { // 回调函数，图片选择后加载至画布前执行
	  console.log(`before picture loaded, i can do something`)
	  console.log(`current canvas context: ${ctx}`)
	})
	.on('imageLoad', (ctx) => { // 回调函数，图片选择后加载至画布时执行
	  console.log(`picture loaded`)
	  console.log(`current canvas context: ${ctx}`)
	})
}
```
初始化裁切方法2
```
onLoad: function () {
	const cropperOpt = this.data.cropperOpt;
	// 初始化裁切
	new weCropper(cropperOpt)
	.on('ready', (ctx) => { // 回调函数，canvas初始化后执行
	  console.log(`wecropper is ready for work!`)
	})
	.on('beforeImageLoad', (ctx) => { // 回调函数，图片选择后加载至画布前执行
	  console.log(`before picture loaded, i can do something`)
	  console.log(`current canvas context: ${ctx}`)
	})
	.on('imageLoad', (ctx) => { // 回调函数，图片选择后加载至画布时执行
	  console.log(`picture loaded`)
	  console.log(`current canvas context: ${ctx}`)
	})
}
```
若同一个页面由多个裁剪容器，需要主动做如下处理
```
this.A = new weCropper(cropperOptA)
this.B = new weCropper(cropperOptB)
```

  9. 绑定图片移动，缩放功能(必须，若没有则无法进行其他操作)
```
touchStart(e) {
	this.wecropper.touchStart(e)
},
touchMove(e) {
	this.wecropper.touchMove(e)
},
touchEnd(e) {
	this.wecropper.touchEnd(e)
},
```

  10. 为底部按钮绑定对应事件
```
// 注册一个选择图片函数
upImg(){
    this.setData({
      dis: 'block'
    })
    wx.chooseImage({ // 图片选择API，详情惨开微信小程序官方文档
      count: 1, 
      success:(res) => {
        var tempFilePaths = res.tempFilePaths;
        if (res.tempFiles[0].size / 1024 > 600) {
          this.setData({
            dis: 'none'
          })
          wx.showToast({
            title: '请选择2M以内的图片！',
          })
          return;
        }
        const src = tempFilePaths[0];
        this.wecropper.pushOrign(src);        
      },
      fail: () => {
        wx.showToast({
          title: '已取消选择',
        })
      }
    })
 }

// 注册重新选择、取消以及确定裁切的事件

	// 重新选择
reChooseImg() {
    this.upImg();
}
	// 取消
cancelChoose() {
    this.setData({
      dis: 'none'
    })
    wx.showToast({
      title: '已取消上传',
    })
}
	//确认裁切
checkChoose() {
	this.wecropper.getCropperImage((avatar) => {
	  if (avatar) {
	    //  获取到裁剪后的图片
	    wx.uploadFile({ //图片上传API，详情见微信小程序官网
	      url: 'https://',
	      filePath: avatar, 
	      name: 'file',
	      success: (res) => {
	        //do something
	        wx.showToast({
	          title: '上传成功',
	        })
	      }
	    })
	  } else {
	    wx.showToast({
	      title: '获取图片失败，请稍后重试',
	    })
	  }
	})
}
```

#### 更多
##### 总参数

  | 参数名      | 类型    |  是否必填  |  参数值    |  备注   |
  | :--------:  | :-----: | :--------: |  :------:  | :-----: |
  | dis         | String  |  true      | none/block |控制截取工具的显示与隐藏|
  | cropperOpt  | Object  |  true      | 见cropperOpt详情| 用于初始化画布 |

##### cropperOpt

  | 参数名      | 类型    |  是否必填  |  参数值    |  备注   |
  | :--------:  | :-----: | :--------: |  :------:  | :-----: |
  | id          | String  |  false     | 默认值cropper | 画布的独有ID |
  | width       | Number  |  false     | 默认值375  | 画布的宽 |
  | height      | Number  |  false     | 默认值375  | 画布的高 |
  | scale       | Number  |  false     | 默认值2.5  | 画布的最大缩放倍数 |
  | zoom        | Number  |  false     | 默认值5    | 画布的缩放系数 |
  | cut         | Object  |  false     | 默认位于左上角| 裁切框的属性|

##### cut(存在)

  | 参数名      | 类型    |  是否必填  |  参数值    |  备注   |
  | :--------:  | :-----: | :--------: |  :------:  | :-----: |
  | x           | Number  |  true      | 用户自定义 | 裁切框左上角距画布左上角横坐标 |
  | y           | Number  |  true      | 用户自定义 | 裁切框左上角距画布左上角纵坐标 |
  | width       | Number  |  true      | 用户自定义 | 裁剪框宽度 |
  | height      | Number  |  true      | 用户自定义 | 裁剪框宽高 |

#### BUGs&&TODOS

##### BUGs
- [ ] 取消选择，不能清空画布

##### TODOS
- [ ] 修复取消选择BUG
- [ ] 可定制的底部按钮(现行---直接修改源码)







