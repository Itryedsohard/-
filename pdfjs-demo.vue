<!--查看pdf票据-->
<template>
  <div class="main-container">
    <input type="file" ref="fielinput" @change="uploadFile" />
    <h1>PDF翻页显示示例</h1>
    <div class="canvas-container">
      <canvas ref="myCanvas" class="pdf-container">
      </canvas>
    </div>
    <div class="pagination-wrapper">
      <button @click="clickPre">上一页</button>
      <span>第{{pageNo}} / {{pdfPageNumber}}页</span>
      <button @click="clickNext">下一页</button>
    </div>
  </div>
</template>

<script>
  import pdfJS from 'pdfjs-dist'
  export default {
    mounted() {
    },
    data() {
      return {
        pageNo: null,
        pdfPageNumber: null,
        pdfTotalPages: 1,
        renderingPage: false,
        pdfData: null, // PDF的base64
        scale: 1 // 缩放值
      }
    },
    methods: {
      uploadFile () {
        let inputDom = this.$refs.fielinput
        let file = inputDom.files[0]
        let reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = () => {
          let data = atob(reader.result.substring(reader.result.indexOf(',') + 1))
          this.loadPdfData(data)
        }
      },
      loadPdfData (data) {
        // 引入pdf.js的字体
        let CMAP_URL = 'https://unpkg.com/pdfjs-dist@2.0.943/cmaps/'
        //读取base64的pdf流文件
        this.pdfData = pdfJS.getDocument({
          data: data, // PDF base64编码
          cMapUrl: CMAP_URL,
          cMapPacked: true
        })
        this.renderPage(1)
        this.renderScrollPdf()
      },
      // 根据页码渲染相应的PDF
      renderPage (num) {
        this.renderingPage = true
        this.pdfData.promise.then((pdf) => {
          this.pdfPageNumber = pdf.numPages
          pdf.getPage(num)
            .then((page) => {
              // 获取DOM中为预览PDF准备好的canvasDOM对象
              let canvas = this.$refs.myCanvas
              let viewport = page.getViewport(this.scale)
              canvas.height = viewport.height
              canvas.width = viewport.width

              let ctx = canvas.getContext('2d')
              let renderContext = {
                canvasContext: ctx,
                viewport: viewport
              }
              page.render(renderContext).then(() => {
                this.renderingPage = false
                this.pageNo = num
              })
            })
        })
      },
      clickPre () {
        if (!this.renderingPage && this.pageNo && this.pageNo > 1) {
          this.renderPage(this.pageNo - 1)
        }
      },
      clickNext () {
        if (!this.renderingPage && this.pdfPageNumber && this.pageNo && this.pageNo < this.pdfPageNumber) {
          this.renderPage(this.pageNo + 1)
        }
      },
      renderScrollPdf () {
        this.pdfData.promise.then((pdf) => {
          this.pdfTotalPages = pdf.numPages
          this.renderScrollPdfPage(1)
        })
      },
    }
  }
</script>

<style scoped>
  .main-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .canvas-container {
    width: 400px;
    height: 500px;
    border: 1px dashed black;
    position: relative;
    display: flex;
    justify-content: center;
  }
  .scroll-pdf-contanier {
    width: 400px;
    height: 500px;
    border: 1px dashed black;
    position: relative;
    display: flex;
    flex-direction: column;
    overflow-y: scroll;
  }
  .pdf-container {
    width: 100%;
    height: 100%;
  }
  .scroll-pdf-container {
    width: 350px;
  }
  .pagination-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>
