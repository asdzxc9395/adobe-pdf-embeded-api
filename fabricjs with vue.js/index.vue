<template>
  <div id="pageDashboard">
    <v-container grid-list-xl fluid>
      <v-btn @click="print">TestChartPrint</v-btn>
      <v-layout row wrap pt-3>
        <v-flex xs6 ref="printTest">
          <v-widget :title="$t('dashboard.graph1.title')" content-bg="white">
            <div slot="widget-content">
              <e-chart
                :path-option="[
                  ['dataset.source', inoutGraph],
                  ['color', [color.lightBlue.base, color.green.lighten1]],
                  ['legend.show', true],
                  ['xAxis.axisLabel.show', true],
                  ['yAxis.axisLabel.show', true],
                  ['grid.left', '0%'],
                  ['grid.bottom', '0%'],
                  ['grid.right', '0%'],
                  ['series[0].type', 'bar'],
                  ['series[1].type', 'bar'],
                  ['series[0].smooth', true],
                  ['series[1].smooth', true],
                  ['series[0].areaStyle', {}],
                  ['series[1].areaStyle', {}],
                  ['series[0].label.show', true],
                  ['series[1].label.show', true],
                  ['series[0].label.position', 'top'],
                  ['series[1].label.position', 'top'],
                ]"
                height="400px"
              >
              </e-chart>
            </div>
          </v-widget>
        </v-flex>
        <v-flex xs6 ref="printTest2">
          <v-widget :title="$t('dashboard.graph2.title')" content-bg="white">
            <div slot="widget-content">
              <e-chart
                :path-option="[
                  ['dataset.source', notinGraph],
                  ['color', [color.cyan.lighten1]],
                  ['legend.show', true],
                  ['xAxis.axisLabel.show', true],
                  ['yAxis.axisLabel.show', true],
                  ['grid.left', '0%'],
                  ['grid.bottom', '0%'],
                  ['grid.right', '0%'],
                  ['series[0].type', 'bar'],
                  ['series[0].smooth', true],
                  ['series[0].areaStyle', {}],
                  ['series[0].label.show', true],
                  ['series[0].label.position', 'top'],
                ]"
                height="400px"
              >
              </e-chart>
            </div>
          </v-widget>
        </v-flex>
      </v-layout>
    </v-container>
    <v-container>
      <canvas :id="'box'"></canvas>
      <v-btn color="primary" dark class="mb-2" id="del">삭제</v-btn>
      <v-btn color="primary" dark class="mb-2" @click="postPdf">전송</v-btn>
      <input type="file" id="pdf-upload" accept="*"> 
    </v-container>
     <v-container>
      <v-data-table
        :headers="headers"
        :items="this.mousePointList"
        item-key="labelNum"
        class="elevation-1"
      >
      <template v-slot:items="props">
        <tr>
        <td class="text-xs-center">{{ props.item.charCode }}</td>
        <td>{{ props.item.X }}</td>
        <td>{{ props.item.Y }}</td>
        <td>
        <input v-model="props.item.spec" placeholder="여기를 수정해보세요" @blur="setSpec(props.item.labelNum + props.item.spec)">
        </td>
        <td>
        <input v-model="props.item.range" placeholder="여기를 수정해보세요"  @blur="setRange(props.item.labelNum + props.item.range)">
        </td> 
        </tr>
      </template>
      </v-data-table>
    </v-container> 

    <dialog-print
      :is-dialog="dialog.isDialog"
      :print-items="dialog.images"
      @close="dialog.isDialog = $event"
      @action="onReload"
    ></dialog-print>
  </div>
</template>

<script>
  import VWidget from '@/components/VWidget'
  import EChart from '@/components/chart/echart'
  import Material from 'vuetify/es5/util/colors'
  import * as DASHBOARD from '../api/dashboard'
  import DialogPrint from '../components/print/chart/DialogPrint'
  import * as PRD from '../api/prd'

  export default {
    name: 'Home',
    layout: 'layout',

    components: {
      VWidget,
      EChart,
      DialogPrint,
    },

    async asyncData() {
      try {
        const {data} = await DASHBOARD.getDashboardList()
        return {
          color: Material,
          inoutItems: data.inouts,
          notinItems: data.notins,
        }
      } catch (error) {
        $nuxt.$emit('on-error', error)
      }
    },

    data: () => ({
      color: Material,
      inoutItems: [],
      notinItems: [],
      dialog: {
        isDialog: false,
        images: {
        output: null,
        output2: null,
        data: '',
        }
      },
      //마우스 커서 이벤트
      // this 내장변수 특성상 배열안에 여러 파라미터가 담긴 객체를 넘기는 경우
      // 객체는 항상 내장변수쪽을 가리키기에
      // 새로운 객체를 선언한다.
      mousePointList: [],
      mousePoint: {
        labelNum: '',
        charCode: '',
        X:'',
        Y:'',
        spec: '',
        range: '',
      },
      //데이터 테이블 
      headers: [
        { text: '마크', align: 'start', value: 'charCode' },
        { text: '좌표X(test)', value: 'X' },
        { text: '좌표Y(test)', value: 'Y' },
        { text: '스펙', value: 'spec' },
        { text: '공차', value: 'range' },
      ],
      num: 0,
      mDown: false,
    }),

    computed: {
      // itemsWithIndex() {
      //   return this.mousePointList.map(
      //     (mousePointList, index) => ({
      //       ...mousePointList,            
      //         index: String.fromCharCode(65 + index) 
      //         // index: index + 1,
      //     }))
      // },
      inoutGraph() {
        if (this.inoutItems.length > 0) {
          return this.inoutItems.map(item => {
            return {
              'month': item.inoutDt,
              '입고': item.inQty,
              '출고': item.outQty
            }
          })
        }
        return [{
          'month': '',
          '입고': 0,
          '출고': 0,
        }]
      },
      notinGraph() {
        if (this.notinItems.length > 0) {
          return this.notinItems.map(item => {
            return {
              'month': item.workDt,
              '상차수량': item.workQty
            }
          })
        }
        return [{
          'month': '',
          '상차수량': 0,
        }]
      },
    },

    mounted() {
      var canvas = new fabric.Canvas('box')

      // pdf => image 변환 시작
      document.querySelector("#pdf-upload").addEventListener("change", function(e) {
          
          var file = e.target.files[0]
          if (file.type != "application/pdf") {
            console.error(file.name, "is not a pdf file.")
            return
          }
          var fileReader = new FileReader();

          fileReader.onload = function() {
            var typedarray = new Uint8Array(this.result);
            var PDFJS = require('pdfjs-dist')
            PDFJS.GlobalWorkerOptions.workerSrc = "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.6.347/pdf.worker.js"
            PDFJS.getDocument(typedarray).promise.then(function(pdf) {
              // you can now use *pdf* here
              console.log("the pdf has ", pdf.numPages, "page(s).")
              pdf.getPage(pdf.numPages).then(function(page) {
                // you can now use *page* here
                var scale = 2.0;
                var viewport = page.getViewport({scale: scale,});
                var canvasEl = document.querySelector("canvas")
                console.log(viewport)
                canvasEl.height = viewport.height;
                canvasEl.width = viewport.width;
                page.render({
                  canvasContext: canvasEl.getContext('2d'),
                  viewport: viewport
                }).promise.then(function() {
                  var bg = canvasEl.toDataURL("image/png");
                  fabric.Image.fromURL(bg, function(img) {
                    img.scaleToHeight(1123);
                    canvas.setHeight(1123);
                    canvas.setWidth(1588);
                    canvas.setBackgroundImage(img);
                    if (canvas.backgroundImage.width < canvas.backgroundImage.height) {
                      img.scaleToHeight(1588)
                      canvas.setHeight(1588);
                      canvas.setWidth(1123)
                    }
                  });
                  canvas.renderAll();
                });
              });

            });
          };
          fileReader.readAsArrayBuffer(file);
        }); 
      // pdf => image 변환 종료
      // 주석 css

      const circle = new fabric.Circle({
      //  // 1)마크 - black,
      //   id: this.num,
      //   radius: 20,
	    //   fill: '#fff',
	    //   opacity: 0.7,
	    //   stroke: 'black',
	    //   strokeWidth: 3,
	    //   strokeUniform: true,
      //   originX: 'center',
      //   originY: 'center'

        // 2)마크 - red
        radius: 20,
        opacity: 0.7,
	      fill: 'red',
        originX: 'center',
        originY: 'center'
      })
      const text = new fabric.Text('A',{
        fontFamily : 'Comic Sans' ,

      //   // 1)마크 - black
      // fontSize: 30,
      // originX: 'center',
      // originY: 'center'

        // 2)마크 - red
        fill: '#fff',
        fontSize: 30,
        originX: 'center',
        originY: 'center'
      })
      canvas.on('mouse:dblclick', e => {
        this.mDown = false
        canvas.add(
        new fabric.Group([circle, text], {
          id: this.num,
          left: e.pointer.x,
          top: e.pointer.y,
          originX: 'center',
          originY: 'center',
          hasControls: false
        })),
        this.mouseEvent(e, canvas),
        this.num++  
      })
      canvas.on('mouse:down', e => {
        this.mDown = true
          // if(canvas.getActiveObject() != undefined) {            
          this.delLabel(canvas, this.mousePointList, canvas.getActiveObject())
            // }
      })
      canvas.on('mouse:up', o=> {
        this.mDown = false;
        if(canvas.getActiveObject() != undefined) {
          const pointer = canvas.getPointer(o.e)
          this.mouseMove(pointer.x, pointer.y, canvas.getActiveObject(), this.mousePointList)
        }
      })
    },
    methods: {
      async mouseEvent(e, canvas) {
        this.mousePoint = new Object()

        this.mousePoint.X = e.pointer.x
        this.mousePoint.Y = e.pointer.y
        this.mousePoint.labelNum = this.num
        
        this.mousePointList.push(this.mousePoint)

        // 주석 text 변경 비동기 처리
        this.mousePointList[this.mousePointList.length - 1].charCode = canvas._objects[this.mousePointList.length - 1]._objects[1].text = String.fromCharCode(65 + this.mousePointList[this.mousePointList.length -1].labelNum) 
      },
      async delLabel(canvas, list, point) {
        document.getElementById('del').onclick = function() {
          for(let i = 0; i < list.length; i++ ) {
            if(canvas.getActiveObject() != null && list[i].labelNum == point.id) {
              list.splice(i, 1)
              canvas.remove(canvas.getActiveObject())
            }
          }
        }
      },
      async mouseMove(x, y, point, list) {
          for(var i = 0; i < list.length; i++) {
            if(list[i].labelNum == point.id) {
              list[i].X = x
              list[i].Y = y
            }
          }
      },
      async postPdf() {
        this.$root.$loading.start()
        try {
          const {message} = await PRD.postPdf(this.mousePointList)
          this.$root.$loading.finish()
        } catch (error) {
          this.$root.$loading.finish()
          this.$nuxt.$emit('on-error', error)
        }
      },
      setSpec(event) {
        if(String(event) != 'NaN') {
        let index = event.charAt(0)  // 번호 추출(배열 index 로 )
        event = event.substring(1)  // index 제거
        for(let i = 0; i < this.mousePointList.length; i++ ) {
            if(this.mousePointList[i].labelNum == index) {
              this.mousePointList[i].spec = event
            }
          }
        }
      },
      setRange(event) {
        if(String(event) != 'NaN') {
        let index = event.charAt(0)  // 번호 추출(배열 index 로 )
        event = event.substring(1)  // index 제거
        for(let i = 0; i < this.mousePointList.length; i++ ) {
            if(this.mousePointList[i].labelNum == index) {
              this.mousePointList[i].range = event
            }
          }
        }
      },
      onReload() {
        this.dialog.isDialog = false
        this.onSearch()
      },
      async print() {

        this.$root.$loading.start()

      try {
      
        const el = this.$refs.printTest;
        const el2 = this.$refs.printTest2;          
      
      const options = {
          type: 'dataURL'
        }
        
        this.dialog.images.output = await this.$html2canvas(el, options);

        this.dialog.images.output2 = await this.$html2canvas(el2, options);
      
      if(document.getElementById('v-widget').offsetWidth >= 750) {
        
        this.dialog.images.data = 'W'
        
        } else {
        
        this.dialog.images.data = 'H'
      }

      this.dialog.isDialog = true

      this.$root.$loading.finish()
        } catch (error) {
          this.$root.$loading.finish()
          this.$nuxt.$emit('on-error', error)
        }
      }
    }
  }
</script>

<style>
#box {
    border: 1px solid #ccc;
  }
</style>
