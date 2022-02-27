# 蛋糕售卖前后台管理系统
后台代码: https://github.com/Masker7/online-mall-cake   (最开始没有采用前后端分离的模式，所以目前代码commit记录全部销毁了)
后台服务暂时关闭可以参考下面的截图和代码

## 代码结构：

## 核心代码部分:
### 无感刷新：
```js
function refreshToken() {
	return axios({
		method: 'post',
		url: 'http://192.168.137.1:13400/omc/api/common/update/accessToken',
		data: {
			refreshToken: getRefreshToken()
		}
	})
}
request.interceptors.request.use(config => {
	nprogress.start()
	if (getAccessToken()) {
		config.headers.authentication = getAccessToken()
	}
	// 请求拦截器
	return config
})
// 是否正在刷新的标记
let isRefreshing = false
// 当token正在刷新的时候，将请求放在请求队列中,token刷新完毕，执行这些请求
let requests = []

request.interceptors.response.use(
	res => {
		if (res.data.code === 400) {
			let config = res.config
			if (!isRefreshing) {
				isRefreshing = true
				return refreshToken()
					.then(res => {
						// 刷新token成功，将最新的token更新到header中，同时保存在localStorage中
						let token = res.data.response.access_token
						setAccessToken(token)
						// 重置一下配置
						config.headers['authentication'] = token
						requests.forEach(cb => cb(token))
						//清空队列
						requests = []
						// 重试当前请求并返回promise
						return request(config)
					})
					.catch(err => {
						console.error('refreshtoken error =>', err)
						//刷新token失败,跳转到首页重新登录吧
						// window.location.href = '/login'
						return Promise.reject(err)
					})
					.finally(() => {
						isRefreshing = false
					})
			} else {
				return new Promise(resolve => {
					requests.push(token => {
						// config.baseURL = ''
						config.headers['authentication'] = token
						resolve(request(config)) // 请求 1 携带自生配重新进入requests请求队列，等待token刷新
					})
				})
			}
		}
		nprogress.done()
		return res
	},
	err => {
		return Promise.reject('fail' + err)
	}
)
```
### canvas图片体积压缩核心部分
详情可以参考这篇我写的文章： https://juejin.cn/post/7069269252867358733
```js

export const compressPic = (file, encoderOtp = 0.3, minSize = 300 * 1024) => {
  return new Promise(resolve => {
    // 1. 通过FileReader读取文件
    const reader = new FileReader()
    let res = reader.readAsDataURL(file)
    reader.onload = (event) => {
      // 2. 读取完毕之后获取图片的base64(上文伏笔),并创建新图片
      const { result: src } = event.target
      console.log(src.length)
      if (src.length < minSize) {
        return resolve(file)
      }
      const image = new Image()
      image.src = src
      image.onload = () => {
        // 3.图片加载完之后通过canvas压缩图片
        const canvas = document.createElement('canvas')
        canvas.width = image.width
        canvas.height = image.height
        // 3.1 绘制canvas
        const ctx = canvas.getContext('2d')
        ctx.drawImage(image, 0, 0, image.width, image.height)
        // 3.2 返回图片url地址,并且进行压缩
        const canvasURL = canvas.toDataURL(file.type, encoderOtp)
        const buffer = atob(canvasURL.split(',')[1])
        // 3.3 bufferArray 无符号位字节数组 相当于在内存中开辟length长度的字节空间
        let length = buffer.length
        const bufferArray = new Uint8Array(length)
        // 3.4 给新开辟的bufferArray赋值
        while (length--) {
          bufferArray[length] = buffer.charCodeAt(length)
        }
        // 3.5将压缩后的文件通过resolve返回出去
        const resultFile = new File([bufferArray], file.name, { type: file.type })
        console.log(resultFile)
        resolve(resultFile)
      }
    }
  })
}


```

## 界面展示
#### 首页
![image](https://user-images.githubusercontent.com/75125132/155880131-85b0666a-2680-4569-8486-8ea0c340112a.png)
#### 商品分类
![QQ录屏20220227191500](https://user-images.githubusercontent.com/75125132/155880206-f1323ece-0957-43b8-a424-ccdc31a9b1b0.gif)
#### 热销
![image](https://user-images.githubusercontent.com/75125132/155880231-e5a1e4a4-6d9b-409a-963a-a10decae19da.png)
#### 新品
![image](https://user-images.githubusercontent.com/75125132/155880238-863630e0-af2a-4080-a7f4-44220d6bf87a.png)
#### 登录注册
![QQ录屏20220227191741](https://user-images.githubusercontent.com/75125132/155880307-48116908-8aaa-4cbb-99e7-c374e8744d56.gif)
#### 个人主页
![image](https://user-images.githubusercontent.com/75125132/155880395-48a8582a-4820-40a8-86dd-2689174982f8.png)
#### 详情
![image](https://user-images.githubusercontent.com/75125132/155887031-cb8e9c8c-a6b5-4225-a619-af0b772ab643.png)
