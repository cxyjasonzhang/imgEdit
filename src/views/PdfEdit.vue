<template>
  <div>
    <div class="view-box" :style="`width:${pdfSize.width}px;height:${pdfSize.height}px;user-select:none`">
      <iframe :src="`${pdfContent}#scrollbars=0&toolbar=0&statusbar=0`" ref="pdfViewer" v-show="pdfContent !== null" width="100%" height="100%"></iframe>
      <div class="board" @dblclick.stop="activeEdit($event)" @click.stop='closeEdit' @mousedown="moveDownPosition" @mousemove='movePosition' @mouseup='mouseupPosition'>
        <!-- 文字显示 -->
        <div v-for="(d,index) in contentList" :data-id='index' :key='index' class="divbox" :style="`position:absolute;z-index:3;left:${d.left-6}px;top:${d.top-8}px;color:red`" @dblclick.stop="activeClick(d)">
          <i class="el-icon-circle-close closediv" size='32' @click.stop='removediv(d,index)'></i>
          <div v-html='d.inputValue'></div>
        </div>
        <!-- 标注框显示 -->
        <svg class="svgLine" id='svgLine'>
          <rect :width='d.width' :height='d.height' :x='d.x' :y='d.y' class="svgrect" @dblclick.stop="removeLine(d,index)" v-for='(d,index) in lineList' :key='index'></rect>
        </svg>
      </div>
      <!-- 绘制线条 -->
      <!-- 弹出写文字的框 -->
      <div v-if="inputShow" :style="inputPosition" class="inputBox">
        <textarea   :rows="10" v-model="inputContent.inputValue" style='width:300px;' />
        <!-- <editor v-model="inputContent.inputValue" @blur="closeEdit" style='width:300px;height:200px'/> -->
        <div class="sureBtn">
          <el-button size="mini" @click="closeEdit">取消</el-button>
          <el-button type="success" size="mini" @click="submitEdit">确定</el-button>
        </div>
      </div>
      <!-- 换页 -->
      <el-pagination background layout="prev, pager, next" :total="total" @current-change="d => (page=d) && modifyPdf(d)" :current-page="page" :page-size="1">
      </el-pagination>
      <el-button @click='saveAs'>保存</el-button>
    </div>
  </div>
</template>
 
<script>
import { PDFDocument, rgb } from "pdf-lib";
// import Editor from '@tinymce/tinymce-vue';
import fontkit from '@pdf-lib/fontkit'
import $ from 'jquery'
const pdfUrl = "https://pdf-lib.js.org/assets/us_constitution.pdf";
import url from '../../public/pdf/sxjs.pdf'
import stsong from '@/assets/chinese.stsong.ttf'

export default {
  name: "pdf_edit",
  // components: { Editor },
  data() {
    return {
      pdfSize: {
        width: 0,
        height: 0,
      },
      inputPosition: {
        position: "absolute",
        top: "0px",
        left: "0px",
        zIndex: 4
      },
      inputShow: false,
      inputContent: {
        inputValue: null,
        contentsList: [],
        top: 0,
        left: 0,
        show: 1
      },
      contentListAll: [],
      contentList: [],
      pdfUrl,
      pdfContent: null,
      total: 0,
      pdfDoc: {},
      page: 1,
      height: 0,
      node: null,
      pdfTempDoc: null,
      lineListAll: [],
      lineList: [],
      ubuntuFont: null
    };
  },
  mounted() {
    // var url = this.pdfUrl + '?' + Math.random()//去除缓存
    console.log(url, 'sasas');
    const _this = this;
    async function getPdfContent() {//加载pdf，并分页
      const arrayBuffer = await fetch(url).then((res) => res.arrayBuffer());
      // const arrayBuffer = url.arrayBuffer();
      _this.pdfDoc = await PDFDocument.load(arrayBuffer);
      // _this.pdfDoc = await PDFDocument.load(url);
      _this.total = _this.pdfDoc.getPages().length//总页数
      _this.modifyPdf(1);//显示第一页
      _this.drawLineDone();//添加画线事件
    }
    getPdfContent()
  },
  methods: {
    //文字拖动事件
    moveDownPosition(position) {
      let index = $(position.target).parents('.divbox').data('id')
      this.node = this.contentListAll[index]
    },
    movePosition(position) {
      if (this.node) {
        if ((this.node.left < 0 && position.movementX < 0) || (this.node.top < 0 && position.movementY < 0) || (this.node.left > this.pdfSize.width && position.movementX > 0) || (this.node.top > this.pdfSize.height && position.movementY > 0)) {
        } else {
          this.node.left = this.node.left * 1 + position.movementX * 1
          this.node.top = this.node.top * 1 + position.movementY * 1
        }
      }
    },
    mouseupPosition(position) {
      this.node = null
    },
    //添加画线事件
    drawLineDone() {
      var _this = this
      $('#svgLine').mousedown(function (position) {
        let that = this
        let x1 = position.offsetX
        let y1 = position.offsetY
        let str = {
          page: _this.page,
          x: x1 * 1,
          y: y1 * 1,
          width: 0,
          height: 0,
        }
        _this.lineList.push(str)
        $(this).mousemove(function (e) {
          let x = e.offsetX
          let y = e.offsetY
          let width = x - x1 * 1
          let height = y - y1 * 1
          if (width > 0) {
            str.width = width
          } else {
            str.width = -width
            str.x = x
            str.y = y
          }
          if (height > 0) {
            str.height = height
          } else {
            str.height = -height
            str.x = x
            str.y = y
          }
        })
        $(this).mouseup(function (e) {
          $(that).unbind('mousemove')
          $(that).unbind('mouseup')
          _this.lineListAll.push(str)
        })
      })
    },
    //删除文字
    removediv(d, index) {
      this.$confirm('确定要删除该标记吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.contentList.splice(index, 1)
        let temp = ''
        this.contentListAll.map((val, index) => {
          if (val == d) {
            temp = index
            return
          }
        })
        this.contentListAll.splice(temp, 1)
      })
    },
    //删除画线
    removeLine(d, index) {
      this.$confirm('确定要删除该标记吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.lineList.splice(index, 1)
        let temp = ''
        this.lineListAll.map((val, index) => {
          if (val == d) {
            temp = index
            return
          }
        })
        this.lineListAll.splice(temp, 1)
      })
    },
    //显示pdf和各未保存的标记
    async modifyPdf(p) {//p是显示第几页
      this.contentList = []
      this.lineList = []
      this.closeEdit()
      for (let i = 0; i < this.contentListAll.length; i++) {//该页所存在的文字
        if (this.contentListAll[i].page == p) {
          this.contentList.push(JSON.parse(JSON.stringify(this.contentListAll[i])))
        }
      }
      for (let i = 0; i < this.lineListAll.length; i++) {//该页所存在的线
        if (this.lineListAll[i].page == p) {
          this.lineList.push(JSON.parse(JSON.stringify(this.lineListAll[i])))
        }
      }
      const _this = this;
      const page = _this.pdfDoc.getPage(p * 1 - 1);
      const { width, height } = page.getSize();
      this.height = height
      _this.pdfSize.width = `${width + 2}`;
      _this.pdfSize.height = `${height + 2}`;
      const pdfTempDoc = await PDFDocument.create();//pdf的显示
      const copiedPages = await pdfTempDoc.copyPages(_this.pdfDoc, [p * 1 - 1]);
      pdfTempDoc.addPage(copiedPages[0]);
      if (window.navigator && window.navigator.msSaveOrOpenBlob) {
        const blob = new Blob([await pdfTempDoc.save()], { type: 'application/pdf' });
      } else {
        const pdfUrl = URL.createObjectURL(
          new Blob([await pdfTempDoc.save()], { type: 'application/pdf' }),
        );
        _this.pdfContent = pdfUrl
      }
    },
    //保存pdf
    async saveAs() {
      const _this = this;
      const pages = this.pdfDoc.getPages()
      // const url = require('@/assets/chinese.stsong.ttf')

      const fontBytes = await fetch(stsong).then((res) => res.arrayBuffer())//添加字体包，没有字体包不显示中文
      this.pdfDoc.registerFontkit(fontkit)
      this.ubuntuFont = await this.pdfDoc.embedFont(fontBytes, { subset: true })//不加subset:true,pdf会变得很大
      //把所有的文字和线框的标记都画到pdf上去
      for (let k = 0; k < this.page; k++) {
        const firstPage = pages[k]
        for (let i = 0; i < this.contentListAll.length; i++) {
          let content = this.contentListAll[i]
          if (content.page - 1 == k) {
            // for (let j = 0; j < content.contentsList.length; j++) {
              let text = content.inputValue
              firstPage.drawText(text, {
                x: content.left * 1,
                y: this.height * 1 - content.top * 1,
                size: 14,
                font: this.ubuntuFont,
                color: rgb(0.95, 0.1, 0.1),
              });
            }
          // }
        }
       
        for (let i = 0; i < this.lineListAll.length; i++) {
          let content = this.lineListAll[i]
          if (content.page - 1 == k) {
            const firstPage = pages[k]
            firstPage.drawRectangle({
              x: content.x*1,
              y: this.height * 1 - content.y*1,
              width: content.width*1,
              height:-content.height*1,
              borderWidth: 2,
              borderColor: rgb(0.75, 0.2, 0.2),
              opacity: 0,
              borderOpacity: 1,
            })
          }
        }
      }
      //把pdf转化成base64
      let pdfContent = await this.pdfDoc.saveAsBase64({
        dataUri: true,
      });
      //此处调接口，把base64返给后台
      this.contentListAll = []
      this.contentList = []
      this.lineListAll = []
      this.lineList = []
      this.modifyPdf(this.page)
    },
    //双击打开写字板
    activeEdit(e) {
      const { offsetX, offsetY } = e;
      // console.log(offsetX, offsetY);
      this.inputPosition.top = `${offsetY}px`;
      this.inputPosition.left = `${offsetX}px`;
      this.inputShow = true;
      this.inputContent.top = `${offsetY}`;
      this.inputContent.left = `${offsetX}`;
      this.inputContent.page = this.page;
      this.inputContent.show = 0
    },
    //保存文字
    submitEdit() {
      if (this.inputContent.show == 0) {
        console.log()
        var inputContent = JSON.parse(JSON.stringify(this.inputContent))
        this.contentListAll.push(inputContent)
        this.contentList.push(inputContent)
      }
      this.closeEdit();
    },
    //关闭写字板
    closeEdit() {
      this.inputContent = JSON.parse(JSON.stringify(this.inputContent))
      this.inputContent.inputValue = null;
      this.inputShow = false;
    },
    //双击修改文字标记
    activeClick(d) {
      this.inputContent = d
      this.inputPosition.left = `${d.left}px`;
      this.inputPosition.top = `${d.top}px`
      this.inputContent.show = 1
      this.inputShow = true;
    },
  },
};
</script>

<style scoped>
.view-box {
  position: relative;
  border: 1px solid #ccc;
  width: 100%;
  height: 600px;
  margin: 10px auto;
}
.pdf-input {
  width: 100px;
  line-height: 20px;
  border: 1px solid #ccc;
  background: #eee;
  z-index: 3;
}
.inputBox {
  background: #fff;
  padding: 5px;
  border-radius: 5px;
}
.sureBtn {
  text-align: right;
  margin-top: 10px;
}
.board {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}
.svgrect {
  stroke: rgb(237, 10, 10);
  stroke-width: 2;
  position: relation;
  fill-opacity: 0;
}
.svgLine {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
  z-index: 1;
}
.divbox {
  cursor: move;
  border: 1px solid red;
  z-index: 3;
  padding: 5px;
  white-space: nowrap;
}
.movediv {
  position: absolute;
  top: -8px;
  left: -8px;
}
.closediv {
  position: absolute;
  top: -8px;
  right: -8px;
  cursor: pointer;
  background: #f64404;
  color: #fff;
  border-radius: 50%;
}
</style>
