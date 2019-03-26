<template>
  <div>
    <button @click="print">打印</button>
    <div class="web">
      <webview ref="printWebview" src="../../static/print.html" nodeintegration
               style="width:280px;height:200px"></webview>
    </div>
    <img id="barcode" v-show="false"/>
  </div>

</template>
<script>
  import {ipcRenderer} from 'electron'
  // import createQrCode from '../until/create-qr-code.js'
  import JsBarcode from 'jsbarcode'

  export default {
    name: 'print',
    data () {
      return {
        printList: {}
      }
    },
    mounted () {
      this.barcode()
      // 监听主线程获取到打印机列表后的回调
      ipcRenderer.send('getPrinterList')
      ipcRenderer.on('getPrinterList', (event, data) => {
        // data就是打印机列表
        this.printList = data
      })
    },
    methods: {
      barcode () {
        // JsBarcode生成条形码
        JsBarcode('#barcode')
          .options({font: 'OCR-B'})
          .EAN13('6927999300031', {fontSize: 16, textMargin: 0})
          .blank(20)
          .render()
      },
      printMore () {
        // 批量打印
        for (let i = 0; i < 3; i++) {
          this.print()
        }
      },
      print () {
        // 获取条形码的base64链接
        let qrImage = document.querySelector('#barcode')
        let img = qrImage.src
        // 生成二维码
        // let str = JSON.stringify({'customer_id': 6})
        // let qrImage = createQrCode.png(str)

        // 当vue节点渲染完成后，获取<webview>节点
        const webview = this.$refs.printWebview
        // console.log(webview)
        // 监听<webview>里面的消息，也就是监听print.html里面的ipcRenderer.sendToHost发送的事件，当该事件发送成功后就会进入下面的回调事件中执行打印操作。
        webview.addEventListener('ipc-message', (event) => {
          console.log(event)
          if (event.channel === 'webview-print-do') {
            // window.print()
            // console.log(webview.print)
            // 如果收到<webview>传过来的事件，名为"webview-print-do"，就执行 webview.print打印方法，打印<webview>里面的内容。
            webview.print(
              {
                // 是否是静默打印
                silent: true,
                printBackground: true,
                // 打印机的名称，就是本文一开始获得的打印机列表其中一个
                deviceName: this.printList.name
              },
              (data) => {
                // 这个回调是打印后的回调事件，data为true就是打印成功，为false就打印失败
                console.log('webview success', data)
              }
            )
          }
        })
        // 向<webview>传递webview-print-render事件
        webview.send('webview-print-render', img)
      }
    }
  }
</script>

<style scoped>
  .web {
    position: absolute;
    left: -200px;
    opacity: 0
  }
</style>
