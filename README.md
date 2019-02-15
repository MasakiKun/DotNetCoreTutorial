# 닷넷기반 웹서비스, 리눅스로 배포까지

아는 분이 닷넷코어 이야기를 하시길래, 간단하게 실습해보고 찌끄려봤다.

여기서는 윈도우 운영체제 하에서 웹 프로젝트가 클래스 라이브러리를 참조하는 형태로 웹서비스 생성 후, 빌드 결과물을 리눅스 서버로 배포하는 부분까지 다룬다.

참고로 나는 닷넷 잘 모름. 예전에 닷넷 프레임워크 2.0 나왔을때 대충 해본게 다라서, 뻘소리 다분할 가능성이 있음.

---

## .NET CLI 설치

먼저 닷넷 명령행 도구를 설치해야 한다. [이 페이지](https://dotnet.microsoft.com/download)에서 자신의 환경에 맞는 .NET Core를 다운로드 받아 설치한다.

당연한 이야기지만, 개발하는 측에서는 Runtime이 아닌 SDK를 설치해야 한다. 또한, 리눅스로 배포할 것이므로 닷넷 프레임워크를 설치하는 우를 범하지 않도록 한다.

## 프로젝트 생성

웹 프로젝트가 저장될 폴더 생성 후, 해당 폴더에서 아래 순서대로 진행한다.

### 솔루션 파일을 생성한다.

    dotnet new sln -n Tutorial

new 명령은 프로젝트나 파일을 생성한다. 생성할 수 있는 프로젝트/파일의 템플릿은 ```dotnet new``` 명령을 입력하면 생성 가능한 템플릿의 종류를 확인할 수 있다.

여기서는 ```sln``` 템플릿으로, 비주얼 스튜디오 솔루션 파일 템플릿이다.

-n 옵션은 생성될 프로젝트/파일의 이름을 명시한다(=name). 여기서는 이름을 Tutorial로 명시했으며, Tutorial.sln 파일이 생성된다. 만약 -n 옵션으로 이름을 지정하지 않으면, 디렉토리 이름이 프로젝트/파일명으로 사용된다.

### 하위 프로젝트 생성

    dotnet new mvc -o My.Web
    dotnet new classlib -o My.Lib

여기서도 마찬가지로 new 명령어로 프로젝트를 2개 생성한다.

여기서는 ```mvc``` 템플릿하고 ```classlib``` 템플릿으로, 각각 MVC 기반 웹앱과 클래스 라이브러리 템플릿이다.

(빈 웹 프로젝트의 템플릿명이 ```web```이므로, ```mvc```보다는 ```webmvc```가 더 적절하지 않았나 싶기도 하고)

-o 옵션은 생성될 프로젝트/파일의 디렉토리 명을 명시한다(=output). 여기서는 각각 ```My.Web``` 디렉토리와 ```My.Lib``` 디렉토리를 가리키며, 디렉토리가 존재하지 않는 경우에는 디렉토리를 먼저 생성하고 템플릿이 생성된다. -n 옵션을 지정하지 않았으므로, 디렉토리 이름이 그대로 프로젝트 명이 된다.

### 솔루션 파일에 프로젝트 추가

    dotnet sln Tutorial.sln add My.Web\My.Web.csproj My.Lib\My.Lib.csproj

```sln``` 명령을 사용했는데, 생성한 솔루션 파일에 프로젝트를 추가하거나 삭제할때 사용한다.

여기서는 먼저 만든 웹 프로젝트와 라이브러리 프로젝트를 솔루션 파일에 추가한다.

**솔직히 다 해놓고나서 생각하는건데, 비주얼 스튜디오로 개발할거 아니면 필요없는거 아님-_-???**

여하튼(...) 추가한 뒤에는 ```sln list``` 명령으로 솔루션 파일에 추가된 프로젝트 목록을 볼 수 있고, ```sln remove``` 명령으로 이미 추가한 프로젝트를 제거할 수 있다.

### 프로젝트 참조 설정

    dotnet add My.Web\My.Web.csproj reference My.Lib\My.Lib.csproj

```add``` 명령은 프로젝트에 패키지나 참조를 추가할 때 사용한다.

여기서는 웹 프로젝트에 라이브러리 프로젝트를 참조하도록 설정했다.

참고로 add 명령의 reference 명령이 아닌 package 명령으로 nuget 패키지를 참조하도록 설정할 수 있다. 설치할 패키지는 [NuGet](https://www.nuget.org) 사이트에서 검색하면 된다. 어차피 NuGet 사이트에서 패키지 참조 설정 명령어까지 다 표시되긴 하지만, 굳이 예를 들자면 JSON 처리를 위한 Newtonsoft.Json 패키지를 참조하려면 아래와 같이 하면 된다.

    dotnet add package Newtonsoft.Json

### 배포 패키지 생성

    dotnet publish --configuration Release --output publish

```publish``` 명령으로 배포할 패키지를 생성할 수 있다. ```--configuration``` 옵션으로 빌드 환경을 지정할 수 있고, ```--output``` 옵션으로 패키지가 생성될 경로를 지정할 수 있다. 이쯤에서 짐작하겠지만, 닷넷 CLI의 옵션들은 리눅스의 long/short 옵션 관행...이라고 하나? 그걸 지원한다.

만약 ```--output``` 옵션이 지정되지 않으면 ```bin\{빌드환경}``` 경로로 생성되는데, 개인적으로는 컴파일된 패키지하고 배포할 패키지가 섞이는 느낌이 들어서 좀 찝찝했다. 그래서 ```--output``` 옵션으로 배포 패키지 경로를 따로 지정했다.

다만, 이대로 배포 패키지를 만들고 배포를 해도 구동이 안되는 불상사(...)가 터지는 경우가 있던데, 이건 아래에서 다시 언급한다.

## 리눅스에 실행환경 설정

### 닷넷 CLI 설치

리눅스에는 대부분의 경우, 해당 배포판에서 제공되는 패키지 관리자로 설치할 수 있다. 다만 패키지 저장소의 메인 스트림에는 닷넷이 포함되지 않은 경우가 대다수인건지, 저장소에 추가를 직접 해 주어야 한다. ~~그나저나 젠투는 왜 없죠... 왜 때문이죠...~~ ~~왜긴 왜야, 변태 리눅스니까! =3==3~~ ~~영원히 고통받는 변태 리눅스~~

[이 문서](https://dotnet.microsoft.com/download/linux-package-manager/ubuntu18-04/sdk-current)를 보고 사용중인 리눅스 배포판에 맞게 패키지 관리자에 저장소를 추가하고 설치한다.

다만, 문서에 보면 ```dotnet-sdk-x.y``` 패키지를 설치하도록 되어있는데, 사실 개발이 아니라 서비스 구동만을 위해서라면 굳이 SDK 패키지를 설치할 필요는 없다. ```dotnet-runtime-x.y``` 패키지를 설치하는 것으로 충분한다.

### 배포 패키지 배포

위 설명대로 따라왔다면 ```My.Web``` 디렉토리(웹 프로젝트)와 ```My.Lib``` 디렉토리(라이브러리 프로젝트)에 각각 publish 디렉토리가 생겼을 것이다.

여기서 ```My.Lib``` 프로젝트는 ```My.Web``` 프로젝트에게 호출되기만 하는 가련한(...) 프로젝트일 뿐이니, 메인 프로젝트는 ```My.Web``` 프로젝트이다.

(실제로는 ```My.Web``` 프로젝트에 Main 메서드가 포함된 public 클래스가 있으므로, ```My.Web``` 프로젝트가 전체 프로젝트의 진입점이 되는거겠지)

```My.Web``` 프로젝트의 publish 디렉토리의 전체 파일을 리눅스 웹서버의 적당한 경로로 업로드한다.

업로드하는 방법은 생략한다.

### 서비스 실행

    dotnet My.Web.dll

여기서는 프로그램의 진입점이 존재하는 프로젝트를 닷넷 CLI로 실행한다.

## 추가정보

### 실행환경 자체포함 배포셋 만들기

개발환경이랑 배포환경의 닷넷 런타임의 버전이 달라지면 실행이 안된다. 옵션은 ```dotnet --info``` 옵션으로 확인할 수 있다.

내 경우, 로컬 개발환경은 Windows 10 + .NET Core 2.1.5 였고, 리눅스는 Ubuntu Server 18.04.2 + .NET Core 2.1.8 이었는데, 실행시에 해당 버전에 맞는 AspDotNetCore가 존재하지 않는다면서 에러메시지만 뿜고 실행되지 않았다.

닷넷은 배포 패키지 안에 실행환경이 포함된 자체포함 어플리케이션(Self-contained Application)을 만들 수 있다. 자체포함 어플리케이션으로 만들면, 배포할 서버에 닷넷코어 실행환경을 굳이 설치하지 않아도 되고, 어플리케이션이 구동될 실행환경을 제어할 수 있다는 장점이 있다. 마치 자바 9 이후에 추가된 [jlink](https://github.com/MasakiKun/jlink-example) 같은 느낌이다.

개인적으로는 앞으로는 간편하게 배포하고 실행할 수 있는 self-contained 어플리케이션이 배포와 운영에 더 유리하지 않을까 생각해본다. 뭐, 필드에서 어떻게 받아들여질지는 모르겠지만.

self-contained 어플리케이션을 만들기 위해서는 publish 옵션에 ```--runtime``` 옵션과 배포할 서버의 운영환경을 주면 된다. 2019년 2월 15일 현재, 닷넷코어에서 지원하는 런타임셋은 [.NET Core RID 카탈로그](https://docs.microsoft.com/ko-kr/dotnet/core/rid-catalog) 페이지를 참고한다.

서론이 길었는데, 요는 publish 명령에 ```--runtime``` 옵션을 주면 된다는 것이다.

    dotnet publish --configuration Release --output publish --runtime linux-x64

위 명령어는 배포 패키지에 64비트 리눅스 실행환경을 포함해서 배포셋을 만든다.

서버에서는 배포셋을 배포 후, 진입점 프로젝트의 이름을 가진 파일에 실행권한을 준 뒤, 그 파일을 그대로 실행하면 서버가 실행된다.

    sudo chmod +x My.Web
    ./My.Web

윈도우용 배포셋의 경우에는, 진입점 프로젝트의 이름을 가진 exe 파일을 실행한다.

    My.Web.exe

