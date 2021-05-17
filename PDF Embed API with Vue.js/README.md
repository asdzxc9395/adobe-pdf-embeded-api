# PDF Embed API with Vue.js



## Vue 관련 Adobe 기본 설정


``` 
const ADOBE_KEY = '';

const app = new Vue({
  el:'#app',
  data:{
    pdfAPIReady:false,
    adobeDCView:null,
    pdfSelected:false
  }, 
  created() {
    //credit: https://community.adobe.com/t5/document-services-apis/adobe-dc-view-sdk-ready/m-p/11648022#M948
    if(window.AdobeDC) this.pdfAPIReady = true;
  }, 
  methods: {
    previewPDF() {
      let files = this.$refs.fileInput.files;
      if(files.length === 0) return;
      this.pdfSelected = true;
      let reader = new FileReader();
      let viewer = this.adobeDCView;
      console.log(`going to view ${files[0].name}`);
      reader.onloadend = function(e) {
        let filePromise = Promise.resolve(e.target.result);
        viewer.previewFile({
          content: { promise: filePromise }, 
          metaData: { fileName: files[0].name }
        });
      };
      reader.readAsArrayBuffer(files[0]);
 
    }
  },
  watch: {
    pdfAPIReady(val) {
      // should only be called when true, but be sure
      if(val) {
        this.adobeDCView = new AdobeDC.View({
          clientId: ADOBE_KEY, 
          divId: "pdf-view"
        });
      }
    }
  }
})

document.addEventListener("adobe_dc_view_sdk.ready", () => { app.pdfAPIReady = true; });
```

- 공식 API문서에 들어가면 (https://www.adobe.com/devnet-docs/dcsdk_io/viewSDK/index.html#getting-credentials) 커스터마이징이 가능하나, 공식 문서에 나오지 않은 자체적인 이슈들만 따로 정리해봐야겠다. 나중에 다른 프로젝트에 쓰일수도 있으니...


1. PDF 주석 관련 sidebar를 잠굴시, load되는 동안 잠깐 빤작이는 이슈 처리

```
var previewFilePromise = viewer.previewFile({
		  	  content: {promise : filePromise },
		  	  metaData:{fileName: files[0].name, id: String(files[0].name)}
		      },{
            enableAnnotationAPIs: true,
            includePDFAnnotations: true,
            annotationUIConfig : { 
              showCommentsPanel : false,
              downloadWithAnnotations: false }
          });
})
```

2. PDF와 그에 대한 주석하나하나마다 primary key값이 존재하는데 파일을 전송시 두가지 방법이 있다.
- 주석이 담긴 PDF 파일 전송
- PDF, 주석데이터 따로 전송(sampleImage에 나와있음)

```
var list_of_annotations = "json형식"
// pdf 오픈시 주석 로드
 annotationManager.addAnnotations(list_of_annotations)
  .then(() => console.log("Success"))
  .catch(error => console.log(error));

```

3. 보통 PDF API를 자체적으로 주석을 수정, 변경할 수 있으나, 사용용도에 따라 주석데이터를 양식에 맞게 커스터마이징함(SampleImage 참고)
- 데이터 테이블 <===> PDF Annotation primary key으로 연결
- 비동기로 삭제, 등록, 수정 가능