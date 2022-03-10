# canvas 绘制二维码+背景+头像

创建 canvas 绘图上下文（指定 canvasId）

```javascript
const contex = wx.createCanvasContext('myCanvas')
```

获取二维码、背景、头像的路径

```javascript
let bgPath = '../images/pic2.jpg';

let qrcodePath = '../images/pic2.jpg';

let portraitPath = '../images/aadf.jpg';

```

### 绘制背景

 .drawImage第一个参数：所要绘制的图片资源； 第二个参数：图像的左上角在目标canvas上X轴的位置；第三个参数：图像的左上角位置在目标canvas上的Y轴的位置；第四个参数：在目标画布上绘制图像的宽度，允许对绘制的图像进行缩放；第五个参数：在目标画布上绘制图像的高度，允许对绘制的图像进行缩放

```javascript
contex.drawImage(bgPath, 0, 0, windowWidth, 935)
```

### 绘制头像

头像是圆形的

.save 保存当前的绘图上下文

.restore 恢复之前保存的绘图文

.beginPath 开始创建一个路径  需要调用fill或者stroke才会使用路径进行填充或描边

.arc 画一条弧线 , 创建一个圆可以用 `arc()` 方法指定其实弧度为0，终止弧度为 `2 * Math.PI`；第一个参数：

圆的x坐标，第二个参数：圆的y坐标；第三个参数：圆的半径；第四个参数：起始弧度；第五个参数：终止弧度；第六个参数：可选，指定弧度的方向是顺时针还是逆时针，默认是false，即顺时针

.clip 从原始画布中剪切任意形状和尺寸，一旦剪切了某个区域，则所有之后的绘图都会被限制在被剪切的区域内（不能访问画布上的其他区域）可以在使用 clip() 方法前通过使用 save() 方法对当前画布区域进行保存，并在以后的任意时间对其进行恢复（通过 restore() 方法）。

```javascript
contex.save()
contex.beginPath();
contex.arc(avatarurl_width / 2 + avatarurl_x, avatarurl_heigth / 2 + avatarurl_y, avatarurl_width / 2, 0, Math.PI * 2, false);
contex.clip()
contex.drawImage(portraitPath, avatarurl_x, avatarurl_y, avatarurl_width, avatarurl_heigth);
contex.restore();
```

>注意：canvas绘制图片，可使用drawImage方法，但是绘制网络图片时，在手机端不会显示，需要先将网络图片下载到本地，然后用图片的本地路径绘制

```javascript
 function downLoadImg(netUrl, storageKeyUrl) {
    wx.getImageInfo({
      src: netUrl,    //请求的网络图片路径
      success: function (res) {
        //请求成功后将会生成一个本地路径即res.path,然后将该路径缓存到storageKeyUrl关键字中
        wx.setStorage({
          key: storageKeyUrl,
          data: res.path,
        });
      }
    })
  }
```

### canvas画图画好以后，不能直接长按保存，所以需要把canvas导出为图片，并预览。实现长按保存的功能

### 把canvas的内容导出为图片



把当前画布指定区域的内容导出生成指定大小的图片，并返回文件路径。

>在 `draw` 回调里调用该方法才能保证图片导出成功。

```javascript
wx.canvasToTempFilePath({
  x: 100,
  y: 200,
  width: 50,
  height: 50,
  destWidth: 100,
  destHeight: 100,
  canvasId: 'myCanvas',
  success: function(res) {
    console.log(res.tempFilePath)
  } 
})
```

### 预览图片

```javascript
wx.previewImage({
    current: this.data.url, // 当前显示图片的http链接
    urls: [this.data.url] // 需要预览的图片http链接列表
 })
```



预览图片的方法最好写在绑定的事件里面

