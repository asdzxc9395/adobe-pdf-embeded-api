<template>
  <div id="pageTable">
    <v-container grid-list-xl fluid>
      <v-layout row wrap>
        <v-flex xs12>
          <v-card>
          <v-toolbar card color="white">
            <!-- <v-subheader style="padding: 0;min-width: 60px"> {{ $t('title.workDate')}}</v-subheader>
             <v-menu
              v-model="isDateFr"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              lazy
              offset-y
              full-width
            >
              <template v-slot:activator="{ on }">
                  <v-text-field
                    v-model="searchForm.dateFr"
                    v-on="on"
                    class="search-field"
                    style="max-width: 110px"
                    filled
                    rounded
                    single-line
                    outline
                    hide-details
                    readonly
                  ></v-text-field>
                </template>
                <v-date-picker v-model="searchForm.dateFr" @input="isDateFr = false"></v-date-picker>
              </v-menu>
              &nbsp;~&nbsp;
              <v-menu
                v-model="isDateTo"
                :close-on-content-click="false"
                :nudge-right="40"
                transition="scale-transition"
                lazy
                offset-y
                full-width
              >
                <template v-slot:activator="{ on }">
                  <v-text-field
                    v-model="searchForm.dateTo"
                    v-on="on"
                    class="search-field"
                    style="max-width: 110px"
                    filled
                    rounded
                    single-line
                    outline
                    hide-details
                    readonly
                  ></v-text-field>
                </template>
                <v-date-picker v-model="searchForm.dateTo" @input="isDateTo = false"></v-date-picker>
              </v-menu>

              <div style="padding: 0 10px"></div> -->

              <v-subheader style="padding: 0 6px;min-width: 50px">JOB-NO</v-subheader>
                <!-- <template v-if="$store.state.authCd == 'VENDOR'"> -->
              <template>
                <v-text-field
                  v-model="vendorCd"
                  class="search-field pr-1"
                  style="max-width: 100px;background: #fafafa"
                  filled
                  rounded
                  single-line
                  outline
                  hide-details
                  readonly
                ></v-text-field>
              <!-- </template> -->
              <!-- <template v-else> -->
                <v-text-field
                  v-model="vendorCd"
                  class="search-field pr-1"
                  style="max-width: 100px"
                  maxlength="8"
                  filled
                  rounded
                  single-line
                  outline
                  hide-details
                ></v-text-field>
              </template>

              <div style="padding: 0 10px"></div>

              <v-subheader style="padding: 0 6px;min-width: 40px">PART-NO</v-subheader>
              <v-text-field
                v-model="vendorCd"
                class="search-field"
                style="max-width: 180px"
                filled
                rounded
                single-line
                outline
                hide-details
              ></v-text-field>

              <v-spacer></v-spacer>
              <input type="file" accept="application/pdf" @change="previewPDF" ref="fileInput"> 
              <v-btn color="primary" dark class="mb-2">조회</v-btn>
              <v-btn color="primary" dark class="mb-2">초기화</v-btn>
              <v-btn color="primary" dark class="mb-2">저장</v-btn>
              <v-btn color="primary" dark class="mb-2">삭제</v-btn>
              <v-btn color="primary" dark class="mb-2">닫기</v-btn>
              <!-- <v-btn color="primary" dark class="mb-2" @click="onConfirm">{{ $t('button.delete') }}</v-btn> -->
          </v-toolbar>
          </v-card>
        </v-flex>
      </v-layout>
          
          <v-card-text class="pt-7">
            <h3 v-if="pdfSelected"></h3>
            <div id="adobe-dc-view" class="pdf-view"></div>
            <div id="annots">
                <v-data-table
                :headers="headers"
                :items="itemsWithIndex"
                item-key="labelNum"
                :rows-per-page-items='[100]'
                class="elevation-1"
              >
              <template v-slot:items="props">
                <tr class="annotationId" @click="select">
                <td class="text-xs-center">{{ props.item.index }}</td>
                <td>
                <input v-model="props.item.spec" placeholder="여기를 수정해보세요" @blur="setSpec(props.item.id + props.item.spec)">
                </td>
                <td>
                <input v-model="props.item.range" placeholder="여기를 수정해보세요"  @blur="setRange(props.item.id + props.item.range)">
                </td> 
                </tr>
              </template>
              </v-data-table>
            </div>
            
          </v-card-text>
        
    </v-container>

  </div>
</template>

<script>
  import * as PRD from '../../../api/prd'

  export default {
    name: "DialogGraph",
    layout: 'layout',
    
    data: () => ({
    vendorCd:'',
		pdfAPIReady:false,
    adobeDCView:null,
    pdfSelected:false,
    annotationManager:null,
    annotationId: new Object(),
    mousePointList: [],
    mousePoint: {
      id: '',
      spec: '',
      range: '',
    },
    //데이터 테이블 
    headers: [
        { text: '마크', align: 'start', value: 'index' },
        { text: '스펙', value: 'spec' },
        { text: '공차', value: 'range' },
      ],
	  }),
    head() {
      return {
        script: [
          {
            src: "https://documentcloud.adobe.com/view-sdk/main.js"
          }
        ]
      }
    },
    created() {
    //credit: https://community.adobe.com/t5/document-services-apis/adobe-dc-view-sdk-ready/m-p/11648022#M948
    if (global.AdobeDC) this.pdfAPIReady = true 
    },
    computed: {
      itemsWithIndex() {
        return this.mousePointList.map(
          (mousePointList, index) => ({
            ...mousePointList,            
              index: String.fromCharCode(65 + index) 
              // index: index + 1,
          }))
      },
    },
    mounted() {
      this.annotationId = document.getElementsByClassName('annotationId')
    },
    methods: {
    previewPDF() {


        let files = this.$refs.fileInput.files;
        if(files.length === 0) return;

        this.pdfSelected = true;

        let reader = new FileReader();
        let viewer = this.adobeDCView = new AdobeDC.View({
            clientId:  "31a75aa0327d418b826597045c34061e", //localhost:3000
            divId: "adobe-dc-view",
            locale: "ko-KR"
          });
        let saveOptions = {
          //  autoSaveFrequency: <Number, default=0>,
          //  enableFocusPolling: <Boolean, default=false>,
          // showDisabledSaveButton: true,
          showSaveButton: true
        }
        var annotationManager;

        console.log(`going to view ${files[0].name}`);

        reader.onloadend = function(e) {
          let filePromise = Promise.resolve(e.target.result)

          var previewFilePromise = viewer.previewFile({
		  	  content: {promise : filePromise },
		  	  metaData:{fileName: files[0].name, id: String(files[0].name)}
		      },{
            // 기본값 false: true는 API를 활성화 합니다. 클라이언트의 애플리케이션은 주석과 PDF 버퍼를 브라우저에서 렌더링하는 PDF EMBED API에 전달합니다.
            enableAnnotationAPIs: true,
            // 기본값 false: true인경우, 기존에 지원되는 주석을 수정할 수 있으며, 지원되지 않는 주석은 읽기전용입니다.
            // 또한 주석을 추가하거나 업데이트 해도 원본 PDF 파일에 영향이 가지 않습니다.
            includePDFAnnotations: true,
            annotationUIConfig : { 
              // 오른쪽 주석창 on / off
              showCommentsPanel : false,
              // 다운로드시 주석 저장
              downloadWithAnnotations: false }
          });

          // 저장함수
          viewer.registerCallback(
            AdobeDC.View.Enum.CallbackType.SAVE_API,
               function(metaData, content, options) {
                //  var base64String = window.btoa(String.fromCharCode(...new Uint8Array(content)));
                  /* Add your custom save implementation here...and based on that resolve or reject response in given format */
                  return new Promise((resolve, reject) => {
                     resolve({
                        code: AdobeDC.View.Enum.ApiResponseCode.SUCCESS,
                        data: {
                           /* Updated file metadata after successful save operation */
                           metaData: metaData,
                           content: content                      
                           }
                     });
                  }).then(
                    annotationManager.getAnnotations()
                                          .then(result => regPDF(content, result))
                                          .catch(error => console.log(error))
                  )
               },
            saveOptions
          );
          
          previewFilePromise.then(function (adobeViewer) {
            adobeViewer.getAnnotationManager().then(function (annotManager) {
              annotationManager = annotManager
              function addCommentText(annotation) {
                  annotation.bodyValue = '';
                  annotationManager.updateAnnotation(annotation)
                      .then(function() {
                        addAnnotation(annotation.id, annotationManager)
                      })
                      .catch(function (error) {
                          console.log(error)
                      });
              }
              // pdf 오픈시  기존 주석정보 다 가져올수 있게
              // annotationManager.getAnnotations()
              //                           .then (result => console.log(JSON.stringify(result)))
              //                           .catch(error => console.log(error))

              // pdf 오픈시 기존에 있던 주석 다 제거
              annotationManager.removeAnnotationsFromPDF()
                                        .then(setAnnotation())
                                        .catch(error => console.log(error));
              
              // pdf 오픈시 주석 로드
              // annotationManager.addAnnotations(list_of_annotations)
                              // .then(() => console.log("Success"))
                              // .catch(error => console.log(error));

              /* API to register events listener */
              annotationManager.registerEventListener(
                  function (event) {
                      if (event.type === "ANNOTATION_ADDED") {
                          addCommentText(event.data)
                      }
                      if (event.type === "ANNOTATION_DELETED" && document.getElementById(event.data.id)) {
                          deleteAnnotation(event.data.id)
                      }
                      if (event.type === "ANNOTATION_SELECTED" && document.getElementById(event.data.id)) {
                          // document.getElementById(event.data.id).style.color = "red";
                          document.getElementById(event.data.id).style.backgroundColor = "#BDBDBD";
                      }
                      if (event.type === "ANNOTATION_UNSELECTED" && document.getElementById(event.data.id)) {
                          document.getElementById(event.data.id).style.color = "black";
                          document.getElementById(event.data.id).style.backgroundColor = "white";
                      }
                      // console.log(event);
                  }
              );
            })
          })
          
          
        }
        reader.readAsArrayBuffer(files[0]); 

        var addAnnotation = ((id, a)=> {
          this.annotationManager = a
          this.addData(id)
        })
        var deleteAnnotation = (id => {
          this.deleteData(id)
        })
        var setAnnotation = (result =>  {
          this.setAnnotation(result)
        })
        var regPDF = ((content, result) => {
          this.regPDF(content, result)
        })
      },
      select(e) {
        var id
        if(e.path[2].id == '') {
          id = e.path[1].id
        } else {
          id = e.path[2].id
        }
        this.annotationManager.selectAnnotation(id).then(function () {}).catch(function () {});
      },
      async addData(id) {
        this.mousePoint = new Object()
        this.mousePoint.id = id
        this.mousePointList.push(this.mousePoint)
      },
      deleteData(id) {
        // document.getElementById(id).remove()
          for(var i = 0; i < this.mousePointList.length; i++) {
            if(this.mousePointList[i].id == id) {
              this.mousePointList.splice(i , 1)
              break
            }
          }  
      },
      regPDF(content, result) {
        console.log(this.mousePointList)
        // console.log(document.getElementsByClassName('annotationId'))
        console.log(content)
        console.log(JSON.stringify(result))
      },
      setAnnotation() {
        if(this.annotationId.length != 0) {
          var paragraphs = Array.prototype.slice.call(this.annotationId, 0);
          
          for (var paragraph of paragraphs) {
            paragraph.remove();
          }
        }
        this.mousePointList = []
      },
      setSpec(event) {
        if(String(event) != 'NaN') {
        let index = event.substring(0,35)  // 번호 추출(배열 index 로 )
        event = event.substring(35)  // index 제거
        for(let i = 0; i < this.mousePointList.length; i++ ) {
            if(this.mousePointList[i].id == index && event != 'undefined') {
              this.mousePointList[i].spec = event
            }
          }
        }
      },
      setRange(event) {
        if(String(event) != 'NaN') {
        let index = event.substring(0,35)  // 번호 추출(배열 index 로 )
        event = event.substring(35)  // index 제거
        for(let i = 0; i < this.mousePointList.length; i++ ) {
            if(this.mousePointList[i].id == index  && event != 'undefined') {
              this.mousePointList[i].range = event
            }
          }
        }
      },
  },
  updated() {
    try {
        if(this.mousePointList.length != 0) {
            for (var i = 0; i < this.mousePointList.length; i++) {
              if(this.annotationId[i].id != this.mousePointList[i].id) {
                this.annotationId[i].id = this.mousePointList[i].id
              }
            }
          } 
        }
    catch (error) {
      this.$nuxt.$emit('on-error', error)
        }
  }
}
</script>
<style>
/* [v-cloak] {display: none} */

.pt-7 {
    margin: 0;
    display: flex;
    height: 100vh;
}
.pdf-view {
  width: calc(80% - 400px);
  margin: 0;
  float: left;
}
#annots {
  width: 400px;
  margin: 0;
  float: left;
}
</style>

<style lang="stylus" scoped>
  .flex.sm1
    flex-basis unset

  .flex.sm2
    max-width 150px

  .v-input.search-field /deep/ .v-input__control .v-input__slot
    border 1px solid rgba(0, 0, 0, 0.54) !important
    min-height 32px !important

  .v-input.search-field /deep/ .v-input__control .v-input__slot input
    margin 0 !important

  .v-text-field /deep/ .v-input__control .v-input__slot
    padding 0 6px

  .v-btn
    min-width unset

  .grid-list-xs .v-text-field
    padding 0
    margin 0

  .grid-list-xl /deep/ .v-table__overflow
    overflow-x scroll

  .pt-7 {
      margin: 0;
      display: flex;
      height: 100vh;
  }
  .pdf-view {
    width: calc(100% - 500px);
    margin: 0;
    float: left;
  }
  #annots {
    width: 500px;
    margin: 0;
    float: left;
  }
</style>
