<template>
  <div class="image-editor-wrap">
    <div class="close-btn">
      <button class="save-btn" @click="handleCanvas2Img">保存</button>
    </div>
    <div id="tui-image-editor"></div>
  </div>
</template>

<script>
import "tui-image-editor/dist/tui-image-editor.css";
import "tui-color-picker/dist/tui-color-picker.css";
import ImageEditor from 'tui-image-editor'
// 菜单按钮中文化
const localeCN = {
  Crop: "裁剪",
  Draw: "涂鸦",
  Icon: "添加图标",
  Text: "添加文本",
  Free: "任意线条",
  Straight: "直线",
  Color: "颜色",
  Range: "范围",
}
// 组件自定义样式
const customTheme = {
// image 坐上角度图片
  'common.bi.image': '', // 在这里换上你喜欢的logo图片
  'common.bisize.width': '0px',
  'common.bisize.height': '0px',
  'common.backgroundImage': 'none',
  'common.backgroundColor': '#f3f4f6',
  // 'common.border': '1px solid #444',
 
  // header
  'header.backgroundImage': 'none',
  'header.backgroundColor': '#f3f4f6',

  // load button
  'loadButton.backgroundColor': '#fff',
  'loadButton.border': '1px solid #ddd',
  'loadButton.color': '#222',
  'loadButton.fontFamily': 'NotoSans, sans-serif',
  'loadButton.fontSize': '12px',
  'loadButton.display': 'none', // 可以直接隐藏掉

  // download button
  'downloadButton.backgroundColor': '#fdba3b',
  'downloadButton.border': '1px solid #fdba3b',
  'downloadButton.color': '#fff',
  'downloadButton.fontFamily': 'NotoSans, sans-serif',
  'downloadButton.fontSize': '12px',
  'downloadButton.display': 'none', // 可以直接隐藏掉

  // icons default
  'menu.normalIcon.color': '#8a8a8a',
  'menu.activeIcon.color': '#555555',
  'menu.disabledIcon.color': '#434343',
  'menu.hoverIcon.color': '#e9e9e9',
  'submenu.normalIcon.color': '#8a8a8a',
  'submenu.activeIcon.color': '#e9e9e9',

  'menu.iconSize.width': '24px',
  'menu.iconSize.height': '24px',
  'submenu.iconSize.width': '32px',
  'submenu.iconSize.height': '32px',

  // submenu primary color
  'submenu.backgroundColor': '#1e1e1e',
  'submenu.partition.color': '#858585',

  // submenu labels
  'submenu.normalLabel.color': '#858585',
  'submenu.normalLabel.fontWeight': 'lighter',
  'submenu.activeLabel.color': '#fff',
  'submenu.activeLabel.fontWeight': 'lighter',

  // checkbox style
  'checkbox.border': '1px solid #ccc',
  'checkbox.backgroundColor': '#fff',
 
  // rango style
  'range.pointer.color': '#fff',
  'range.bar.color': '#666',
  'range.subbar.color': '#d1d1d1',
 
  'range.disabledPointer.color': '#414141',
  'range.disabledBar.color': '#282828',
  'range.disabledSubbar.color': '#414141',
 
  'range.value.color': '#fff',
  'range.value.fontWeight': 'lighter',
  'range.value.fontSize': '11px',
  'range.value.border': '1px solid #353535',
  'range.value.backgroundColor': '#151515',
  'range.title.color': '#fff',
  'range.title.fontWeight': 'lighter',
 
  // colorpicker style
  // 'colorpicker.button.border': '1px solid #1e1e1e',
  // 'colorpicker.title.color': '#fff'
}



export default {
  data() {
    return {
      instance: null
    }
  },
  mounted() {
    this.instance = new ImageEditor(
      document.querySelector('#tui-image-editor'),
      {
        includeUI: {
          loadImage: {
            path: "https://ts1.cn.mm.bing.net/th/id/R-C.6b5df1bfe0e4778a44dba0753cd169c8?rik=QRQIMqvjWRCO5Q&riu=http%3a%2f%2fpic39.nipic.com%2f20140321%2f8857347_232251363165_2.jpg&ehk=7oAaMo6LCHJc%2bqpQ0IPvcH7v69jGRQhb2vDz%2fOd5720%3d&risl=&pid=ImgRaw&r=0",
            name: "image"
          },
          menu: ["crop", "draw", "icon", "text"],
          initMenu: "draw",
          locale: localeCN,
          // 自定义主题
          theme: customTheme,
          // 菜单位置 下面
          menuBarPosition: "bottom",
          // 设置画布的最大宽高，能自动等比例缩放图片到指定的宽高内
          cssMaxWidth: 850,
          cssMaxHeight: 580
        },
      }
    )
    // 清除自定义样式造成的一条边框线
      document.getElementsByClassName('tui-image-editor-main')[0].style.top = 0
      // 设置图片编辑其余距离底部90px（就不会被底部展开的工具栏遮挡住了）===>无效
      // document.getElementsByClassName('tui-image-editor-wrap')[0].style.bottom = 90
 
      //! 修改图片编辑器的顶部导航栏
      // 这个获取到的是tui-image-editor组件默认自带的顶部菜单栏目
      // console.log('顶部工具栏dom')
      // console.log(document.querySelector('.tui-image-editor-help-menu'))
      // 将顶部菜单栏整个隐藏掉
      // document.querySelector('.tui-image-editor-help-menu').style.display = 'none'
 
      // 你也可以指定那个菜单隐藏，留几个有用的菜单
      // document.querySelector('[tooltip-content="Undo"]').style.display = 'none'// 上一步
      // document.querySelector('[tooltip-content="Redo"]').style.display = 'none' // 下一步
      // document.querySelector('[tooltip-content="Reset"]').style.display = 'none' // 完全重新编辑
      document.querySelector('[tooltip-content="ZoomIn"]').style.display = 'none' // 放大
      document.querySelector('[tooltip-content="ZoomOut"]').style.display = 'none' // 缩小
      document.querySelector('[tooltip-content="Hand"]').style.display = 'none' // 拖动界面
      document.querySelector('[tooltip-content="History"]').style.display = 'none'
      document.querySelector('[tooltip-content="Delete"]').style.display = 'none' // 删除选中编辑内容
      document.querySelector('[tooltip-content="DeleteAll"]').style.display = 'none' // 清空
      // 隐藏分割线
      document.querySelectorAll('.tui-image-editor-icpartition').forEach(item => {
        item.style.display = 'none'
      })
  },
  methods: {
    /** 保存编辑后图片 */
    handleCanvas2Img () {
      // 调用组件官方方法，获取整个编辑后图片的base64数据
      const base64String = this.instance.toDataURL()
      // 给图片左下角插入文字（传入当前获取到的编辑后的图片base64数据）
      // this.addUserNameText(base64String)
      // 要延时调用，否则会被锁死，因为异步方法会在没有这个dom时触发它
      setTimeout(() => {
        //! 将编辑后的base64图片码传给上传组件（父组件）
        // ?this.instance.toDataURL()==》重新调用组件官方方法，获取加入左下角文字后的base64数据
        // sessionStorage.setItem('editedPicBase64', this.instance.toDataURL())
        const a = document.createElement('a')
        const filename = 'picName.png'
        a.href = base64String  // picSrc  是图片 base64 码，可以直接给 img 的 src 属性，展示图片
        a.download = filename
        document.body.appendChild(a)
        a.click()
        setTimeout(() => {
          document.body.removeChild(a)
        }, 1000)
        // sessionStorage.setItem('editedPicBase64', base64String)
      }, 700)
    },
    // 对图片插入左下角用户名和日期文字===》当点击保存按钮时触发
    addUserNameText (base64String) {
      const fileUrl = this.dataURLtoFile(base64String)
      // ?new Date().getTime()==>获取当前时间戳，并通过changeDate方法转换成好看的格式，存在data中，方便绘制图片的时候印上去
      const drawADate = this.changeDate(new Date().getTime())
      // console.log('转化base64为文件类型：==》', fileUrl)
      //! tui-image-editor组件的官方方法，用于获取当前图片的宽高
      this.instance.loadImageFromFile(fileUrl).then(result => {
        // TODO：改成获取userName
        this.instance.addText('张三' + drawADate, {
          styles: {
            fill: '#2196F3',
            fontSize: 22,
            fontWeight: 'bold'
          },
          position: {
            x: 10,
            y: result.newHeight - 30
          }
        }).then(objectProps => {
          console.log('左下角文字加上了')
        })
      })
    },
    dataURLtoFile (dataurl) {
      var arr = dataurl.split(',')
      var mime = arr[0].match(/:(.*?);/)[1]
      var bstr = atob(arr[1])
      var n = bstr.length
      var u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new File([u8arr], { type: mime })
    },
     // 转换时间格式
    changeDate (originVal) {
      const dt = new Date(originVal)
      // 年
      const y = dt.getFullYear()
      // dt.getMonth() + 1 就是获取日期+1变成1-12月，
      // ! 通过.padStart(2, '0')，当字符串长度不足两位时，用'0'来填充在首位至2位
      // ! 因为是针对字符串的，所以在dt.getMonth() + 1 + '' 转为字符串
      // 月
      const m = (dt.getMonth() + 1 + '').padStart(2, '0')
      // 日
      const d = (dt.getDate() + '').padStart(2, '0')
      // 时
      const hh = (dt.getHours() + '').padStart(2, '0')
      // 分
      const mm = (dt.getMinutes() + '').padStart(2, '0')
      // 秒
      const ss = (dt.getSeconds() + '').padStart(2, '0')
      // 返回过滤后的时间格式  yyyy-mm-dd hh:mm:ss
      return `${y}-${m}-${d} ${hh}:${mm}:${ss}`
    }
  }
}
</script>


<style>
.image-editor-wrap {
  width: 100%;
  height: 80vh;
  background: #f9f9f9;
}

.image-editor-wrap  canvas {
  border: 1px solid #b3b3b3;
}

.save-btn {
  position: absolute;
  top: -30px;
  left: 10px;
  z-index: 999;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  background-color: skyblue;
  cursor: pointer;
}


/* 设置弹窗中的图床距离顶部距离 */
.el-dialog__header {
  margin-bottom: 25px;
}
/* 强制压缩菜单的高度 ，减少占用屏幕的空间*/
.tui-image-editor-container .tui-image-editor-submenu {
  height: auto !important;
}
.tui-image-editor-container.bottom .tui-image-editor-submenu > div {
  padding: 0 !important;
}
.tui-image-editor-container .tui-image-editor-help-menu.top {
  background-color: white;
}
/* 顶部工具栏定位 */
.tui-image-editor-container .tui-image-editor-header {
  top: -55px;
}
.tui-image-editor-container .tui-image-editor-help-menu.top {
  top: -50px;
}
/* 取消超出部分隐藏，否则因为顶部工具栏已经超出去了，会显示不出来
.tui-image-editor-container {
  overflow: auto;
} */
/* 顶部工具栏定位 */
.tui-image-editor-container {
  overflow: visible;
}
</style>
