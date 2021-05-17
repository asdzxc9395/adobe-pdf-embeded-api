### PDF주석

## 원리

-pdf 파일 업로드 
-pdd to image(base64) 변환
-fabricjs를 통한 주석 등록 / 삭제
-axios 전송

설정:
npm install pdfjs-dist (pdf => image변환),(라이브러리 등록방법이 다르니 참고바람)
npm install fabricjs(주석기능)

```html
    <v-container>
      <canvas :id="'box'"></canvas> <!--(2) -->
      <v-btn color="primary" dark class="mb-2" id="del">삭제</v-btn><!--(5) -->
      <v-btn color="primary" dark class="mb-2" @click="postPdf">전송</v-btn><!--(6) -->
      <input type="file" id="pdf-upload" accept="*"> <!--(1) -->
    </v-container>
     <v-container>
      <v-data-table
        :headers="headers"
        :items="this.mousePointList"
        item-key="labelNum"
        class="elevation-1"
      >
      <template v-slot:items="props"><!--(3) -->
        <tr>
        <td class="text-xs-center">{{ props.item.charCode }}</td>
        <td>{{ props.item.X }}</td>
        <td>{{ props.item.Y }}</td>
        <td><!--(4) -->
        <input v-model="props.item.spec" placeholder="여기를 수정해보세요" @blur="setSpec(props.item.labelNum + props.item.spec)">
        </td>
        <td>
        <input v-model="props.item.range" placeholder="여기를 수정해보세요"  @blur="setRange(props.item.labelNum + props.item.range)">
        </td> 
        </tr>
      </template>
      </v-data-table>
    </v-container> 
```


# (1),(2) input file을 통해 pdf파일을 가져온다
-- pdfjs-dist를 통해 해당 pdf 파일을 업로드함과 동시에 image파일로 변환하여 화면에 출력한다.

```javascript
<script>
mounted() {

  var canvas = new fabric.Canvas('box') // canvas 호출

      document.querySelector("#pdf-upload").addEventListener("change", function(e) {
        var file = e.target.files[0]
          if (file.type != "application/pdf") {
            console.error(file.name, "is not a pdf file.")
            return
          }
          var fileReader = new FileReader();
          fileReader.onload = function() {
            var typedarray = new Uint8Array(this.result);
            // 69 ~ 70까지 PDFJS 라이브러리를 등록하는 방법이다

            // 'pdfjs-dist'패키지가 ES 모듈로 작동하지 않는 것으로 보이므로 import pdfjs from 'pdfjs-dist/webpack'정의되지 않은 가져 오기가 발생한다는 것입니다. 그러나 const pdfjs = require('pdfjs-dist/webpack')예상대로 작동합니다.

            // 문서참조 https://mozilla.github.io/pdf.js/examples/
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
                    // 가로, 세로 반응형으로 변환
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
}
</script>
```

# (3) 주석 등록, (4) 주석별 데이터 등록, (5) 주석 삭제
- fabricjs를 통해 이중 canvas를 씌워 주석을 만든다.
- fabricjs의 주석, 테이블 데이터를 각각 따로 분리한다음 전역 index를 주었다.(num)
- 마우스 더블클릭을 통해 등록 이벤트를 주었으며 화면에 보여지는 index는 문자열로 처리하였다.(A = 0, B = 1......)
- 주석을 삭제시, 전역 index를 통해 주석과 table데이터를 동시에 삭제할수 있도록 구현하였다.
- 밑의 데이터 구조를 통해 mousepoint의 데이터를 mousepointlist에 담아 처리하였다.
- 전역 data 구조

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


```html
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
```
```javascript
<script>
mounted() {
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
}
methos() {
  {
    
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
}
  }
<script>
```

# (6) 해당 데이터 전송
- mousepointlist의 데이터들을 axios로 전송하고,
- 해당 pdf의 이미지도 전송을 해야하는데,
- pdf데이터를 등록시 이미지 or pdf로 업로드를 처리해야하는데 이 부분은 추후에 결정해야 할듯 하다.