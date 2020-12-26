---
layout: post
title: "Firebase 기반 QR코드 생성 프로그램"
date: 2020-12-21
tags: [server]
category : [etc]
comments: true
---

### 기록

##### 12.26
- 에러 1 : `Android SDK is missing required platform API: Required API level 30`
  - 해결 1. Update Android SDK 클릭
    - 디바이스가 연결된 상태임에도, 디바이스를 연결해달라는 메세지 발생.
    - 해결 1.1. 핸드폰 [개발자 모드](https://forum.unity.com/threads/no-android-devices-connected.907934/) 접속, 레퍼런스의 영상 따라서 진행함
      - 에러 : 현재 핸드폰은 API level이 29이고, 프로젝트의 minimum API level은 30으로 되어 있음. 본인 핸드폰이 그리 낡은 기종은 아니므로, 프로젝트 세팅에서 API level을 낮춰줄 예정. (Target API는 30으로, Minimum API는 26으로 설정해줌)
  - 결과 : 해당 에러메세지 사라지고 다른 에러(에러2) 발생

---

- 에러 2 : `Copying assembly from 'Temp/Unity.Subsystem.Registration.dll' to 'Library/PlayerScriptAssemblies/Unity.Subsystem.Registration.dll' failed`
  - 시도 1. 스크립트 폴더를 마우스 오른쪽 버튼으로 클릭 -> Reimport 메뉴 항목을 왼쪽 클릭 (그러나 매번 이렇게 해야한다고 한다.)
    - 결과 : Building Player 등으로 잘 넘어감.
  - 시도 2. `FINALLY. I got a lot of Unity errors and this was the cause. Also solved with Malwarebytes. Looks like miners F*** up with the Unity importer... Thanks a lot, I've been having troubles for weeks.` Security Program때문에 이런 충돌이 발생하는 듯. Coin Miner Malware가 해당 충돌을 일으키기도 한다. (시도 1에서 잘 해결되어 시도 2는 적어두기만 하고 진행하지 않음.)


##### 12.25
- `Could not find key in plist file: [DATABASE_URL]` 에러 발생
- [해결 레퍼런스](https://github.com/firebase/quickstart-unity/issues/892)
- means that the XML generator is attempting to find a URL for your configured Database on the backend. Could you check your GoogleServices.plist file that you have locally to see if there's an entry like this:

  ```
  <key>DATABASE_URL</key>
	<string>https://something-something.firebaseio.com</string>
  ```

- If not, then please attempt to redownload the plist from the Firebase Console and see if it's in there If it's not there then you may need to configure your Database on the console before it appears in the plist.
- => GoogleServices-Info.plist 파일에서 데이터베이스 URL을 설정해준다. 해결되지 않을 경우 Firebase 콘솔에서 다시 다운로드받거나, 콘솔에서 데이터베이스를 다시 확인
- *.plist 파일이 열리지 않아 범용파일리더기를 다운로드 받아 열 예정. 맥의 속성 목록 파일이라고 한다. [다운로드 사이트](https://www.icopybot.com/download.htm)에서 `plist Editor Pro` 다운로드.
- DATABASE_URL key값 자체가 없어서 추가해줌
- 결과 : 해당 에러 메세지 사라짐.

---

- 에러 메세지 : UnityEditor.BuildPlayerWindow+BuildMethodException: Error building Player because scripts have compile errors in the editor
- 시도 1. `Reimport all` 클릭 => 해결되지 않음
- 시도 2. 에러 로그 파일 확인 (위치 : Windows는 `	C:\Users\username\AppData\Local\Unity\Editor\upm.log`에 있음)
  - '[LicensingClient] ERROR Failed to connect to local IPC' 에러 발견
    - `due to a difference in Unity version between what you are using in development and what you have configured on Cloud Build` [레퍼](https://forum.unity.com/threads/error-failed-to-connect-to-local-ipc.817803/)
    - 파이어베이스 사용을 위해서는 Unity 특정 버전이 필요한건가? 그럼 그 전엔 어떻게 빌드?
      - [Firebase Unity SDK 문서](https://firebase.google.com/docs/unity/setup#add-sdks-upm) 다시 확인.
        - `Unity 2018.4 이상 및 .NET 4.x 또는 .NET Standard 2.0을 사용 중인 경우 Unity Package Manager를 사용하여 Firebase SDK 구성요소를 설치할 수 있습니다.`이라는 말 쓰여있다. 현재 Unity 2019.4이므로 `Unity Package Manager`를 사용하여 SDK 구성요소를 설치.
        - [패키지 매니저 매뉴얼](https://docs.unity3d.com/Manual/upm-ui.html) : 현재 Reimport All을 돌리는 중이라, Window > Package Manager에 접근하는건 Reimport 이후 해봐야할 듯.
          - Google의 게임 패키지 레지스트리를 Unity 프로젝트에 추가합니다. 를 위하여 Pacakages/manifest.json에 다음 블록을 추가 (사용법 [관련문서](https://docs.unity3d.com/Manual/upm-scoped.html))

          ```
          "scopedRegistries": [
  {
    "name": "Game Package Registry by Google",
    "url": "https://unityregistry-pa.googleapis.com",
    "scopes": [
      "com.google"
    ]
  }
]
 ```

      - 문서에 따르면 2019 이상이므로 dotnet4/패키지를 가져오면 된다고 한다. 그랬던 것 같은데.. 혹시 아닐수도 있으니 dotnet4 패키지로 다시 가져오자.
        - database, firestore(=>Resolving Android Dependencies가 길게 나옴), auth 가져옴
 - 시도 3. Unity 프로젝트가 다음 요구 사항을 충족하는지 확인합니다. iOS의 경우 — iOS 8 이상 Target API Android의 경우 — API 수준 16(Jelly Bean) 이상 타겟팅 이라는 문구를 [해당 문서](https://firebase.google.com/docs/unity/setup#add-sdks-upm)에서 확인하였으나 Android API 11까지만 가능하여 우선 11로 설정, 다시 빌드

---

- 에러 : `Library\PackageCache\com.unity.textmeshpro@2.0.1\Scripts\Editor\TMP_PackageUtilities.cs(310,17): error CS0433: The type 'Task' exists in both`
- 해결 1 : [공식 질의문서](https://www.google.com/search?q=Library%5CPackageCache%5Ccom.unity.textmeshpro%402.0.1%5CScripts%5CEditor%5CTMP_PackageUtilities.cs(310%2C17)%3A+error+CS0433%3A+The+type+%27Task%27+exists+in+both&rlz=1C1CHZL_koKR759KR759&oq=Library%5CPackageCache%5Ccom.unity.textmeshpro%402.0.1%5CScripts%5CEditor%5CTMP_PackageUtilities.cs(310%2C17)%3A+error+CS0433%3A+The+type+%27Task%27+exists+in+both&aqs=chrome..69i57j69i59j69i58.311j0j7&sourceid=chrome&ie=UTF-8)의 해결책
  - Package Manager에서 TextMeshPro를 업데이트
  - 결과 : 해당 에러 메세지는 사라졌으나 GIFRecorder, IMediaRecorder, HEVCREcorder, JPGRecorde, MP4Recorder, WAVRecorder 등과 관련된 다른 에러 메세지 10가량 발생
  - 관련하여 찾다보니 [해당 게시글](https://forum.unity.com/threads/type-task-exists-in-both-unity-tasks-and-mscorelib.855385/)을 발견, Google Firebase가 Unity.Tasks라는 dll을 포함하며 이에 의하여 Task를 사용하는 다른 녀석들과 문제를 일으키고 있는 듯 하다.
  - 질문 작성자의 해결법 : I was able to fix it. Sorry I have not answered. I uninstalled Firebase from my project and re-installed it using the Package Manager (with .NET set to 4.x). I can now use System.Threading.Tasks just fine with Firebase working as well. One thing to note is (for now) Firebase does not work on Android builds in the Unity 2020 beta. Just food for thought. (파이어베이스를 프로젝트에서 uninstall하고, Package Manager를 사용하여 re-intall한다.)
    - Firebase에서 제안한 External Dependency Manager로 기존의 의존성 매니저를 대체했더니 대략 10시간을 돌려도 끝나지 않는 이상한 의존성 업데이트를 해서, 아침에 확인 후 유니티 강제종료함.
    - 결과 : 이후 다시 빌드하자 에러메세지 사라짐.


##### 12.23.
- 유니티 안드로이드 apk 빌드 [레퍼런스](https://dooding.tistory.com/11)


##### 12.22.
파이어베이스 콘솔로 파이어베이스 프로젝트 생성
유니티-파이어베이스 연동 매뉴얼 따라서 진행중
파이어베이스 sdk 받는데만 20분이 걸린다. 노트북을 사서 그거로 메이플을 하고 싶은 마음이 든다.. 메이플을 못한지 어느덧 4일 정도가 지났다.

##### 12.21.
유니티와 프로젝트를 받아두고, 프로젝트 압축만 2시간을 풀었다. 받고보니 유니티 2019.4.16f1버전 프로젝트가 아니라 `2019.4.1f1` 버전이어서 다시 받는 중.

서버 연결하고 모바일앱은 텍스트에서 큐알이미지로, 키넥트앱은 큐알이미지에서 텍스트로 넘겨주는 코드 필요.

키넥트쪽은 알고보니 `2019.4.4f1`버전이어서 또 받는 중.
일단 유니티는 4.x 버전이면 어느 정도 호환이 가능하다는 말에 버전 4.16f1로 진행.
안드로이드 iOS 필요해서 unity에 받아둠.
