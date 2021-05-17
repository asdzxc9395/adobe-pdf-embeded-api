# Adobe Document Services PDF Embd API 시작

Adobe Document Services PDF Embd API를 사용하면 웹 사이트에 PDF 파일을 표시하고 이 기능을 활용하여 사용자에게 혜택을 제공할 수 있습니다.

## 전제조건

PDF Embd API로 작업하려면 먼저 응용프로그램을 등록하고 클라이언트 ID(API Key)를 가져와야 합니다. [Request Access](https://www.adobe.com/go/dcsdks_credentials)에서 사용자 자신의 클라이언트 ID를 가져옵니다.

## 지원되는 플랫폼

1. Windows - Chrome, Firefox, Edge, IE11+
2. Mac - Chrome, Firefox, Safari
3. 안드로이드 - 크롬
4. iOS - Chrome, Safari

## 샘플 실행

다음 하위 절에서는 검체를 실행하는 방법을 설명합니다.

## PDF 미리 보기

PDF 뷰어는 사용자가 문서를 읽고 작업할 수 있도록 전체 브라우저 창을 채웁니다.
이 기능을 사용하려면 ```전체 창 임베드 모드``` 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

### 크기 컨테이너 임베드 모드

PDF 뷰어는 슬라이드 쇼 및 가로 모드로 크기가 지정된 용기에 포함되어 있습니다.
이 기능을 사용하려면 ```Size Container Embedding Mode``` 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

## 인라인 임베드 모드

PDF 뷰어는 앱/웹 페이지의 컨텍스트 내에 포함되어 있습니다.
이 기능을 사용하려면 "In-Line Embedd Mode" 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

### 라이트박스 임베드 모드

PDF 뷰어는 웹 페이지 상단에 표시되고 전체 브라우저 창을 채워서 집중된 보기를 제공합니다. 콘텐츠 웹 사이트, 콘텐츠 포털 및 전자 메일에 가장 적합합니다.
이 기능을 사용하려면 "Lightbox Embedding Mode" 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

### 로컬 PDF 파일 미리 보기

이 샘플은 PDF Embd API를 사용하여 로컬 PDF 파일을 렌더링하는 방법을 보여줍니다. PDF Embd API를 사용하여 PDF 파일 렌더링을 시작하기 위해 컴퓨터에서 PDF 파일을 선택하라는 메시지가 표시됩니다.
실제로 보려면 ```More Sample/Work with Local File``` 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

### 주석 도구를 사용한 PDF 미리 보기

PDF 뷰어는 PDF 파일을 주석 도구와 함께 렌더링합니다.
실제로 보려면 ```More Sample/PDF Annotations``` 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

주석을 추가한 후 사용자가 PDF 파일을 저장하는 경우 기본적으로 파일은 로컬에서 다운로드됩니다. 이 샘플은 이 경우 약간의 지연을 추가하는 것 외에는 아무 조치도 취하지 않는 동작을 변경하는 방법을 보여 줍니다.

### 주석 API가 포함된 PDF 미리 보기

PDF 뷰어는 PDF 파일을 주석 도구 및 API와 함께 렌더링합니다.
이 샘플은 주석 API를 사용하여 주석을 프로그래밍 방식으로 추가, 가져오기, 내보내기, 업데이트 및 삭제하는 방법을 보여줍니다.
실제로 보려면 ```More Samples/PDF Annotations APIs``` 폴더를 컴퓨터에 복사한 후 지원되는 브라우저에서 사용해 보십시오.

### 이벤트 포함 PDF 미리 보기

이 샘플은 PDF 뷰어에서 이벤트 수신을 시작하는 방법을 보여줍니다.
실제로 보려면 ```More Sample/Capture PDF Embedd API Events``` 폴더에 있는 파일을 컴퓨터에 복사하고 지원되는 브라우저에서 index.html을 여십시오.

### 검색 엔진 색인 처리

이러한 샘플은 PDF Embd API를 사용하여 표시되는 PDF 파일의 인덱싱을 활성화하는 방법에 대한 다양한 구현을 제공합니다.
이 기능을 사용하려면 "More Sample/Handle Search Engine Indexing" 폴더를 컴퓨터에 복사한 후 지원되는 브라우저에서 사용해 보십시오.

## 뷰어 APIs

PDF Embd API는 PDF와의 사용자 인터페이스 및 사용자 상호 작용을 사용자 정의할 수 있는 뷰어 API를 제공합니다.
이 샘플은 이러한 API를 사용하여 런타임에 프로그래밍 방식으로 UI 사용자 지정을 수행하는 방법을 보여 줍니다.
실제로 보려면 ```More Sample/Viewer APIs``` 폴더에 있는 파일을 컴퓨터에 복사한 후 지원되는 브라우저에서 사용해 보십시오.

## JavaScript 프레임워크 샘플

### 반응 샘플

이 샘플은 PDF 뷰어를 React 응용 프로그램에 통합하는 방법을 보여줍니다.
실제로 확인하려면 "More Sample/Ract Sample" 폴더를 컴퓨터에 복사하고 React Sample [README](More%20 Sample/React%20 Sample/README.md)의 지침을 따르십시오.

### 각도 샘플

이러한 샘플은 PDF 뷰어를 Angular 응용 프로그램에 통합하는 방법을 보여줍니다.
실제로 확인하려면 "More sample/Angular sample" 폴더를 컴퓨터에 복사하고 Angular sample [README](More%20 sample/Angular%20 sample/README.md)의 지침을 따르십시오.

## 설명서

자세한 내용은 https://www.adobe.com/go/dcviewsdk_docs을 참조하십시오.

## 라이선스

이 프로젝트는 MIT 라이센스에 따라 라이센스가 부여됩니다. 자세한 내용은 [License](LICENSE.md)를 참조하십시오
These samples demonstrate how to integrate the PDF viewer in an Angular application.
To see it in action, copy ```More Samples/Angular Samples``` folder to your computer, and follow the instructions in Angular Samples [README](More%20Samples/Angular%20Samples/README.md).
