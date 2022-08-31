---
layout: post
title: React Native Splash Screen
author: vgihan
description: React Native로 Splash Screen 적용하기
tags: [ReactNative, SplashScreen]
categories: [React Native]
published: true
extensions:
  preset: gfm
---

앱을 시작하기 전에 화면에 등장하는 스플래시 이미지를 React Native 환경에서 적용해보자. react-native-splash-screen 라이브러리를 활용하여 구현할 수 있다. iOS에서 실행을 확인할 수 있는 방법이 없어, 안드로이드 환경에서 구현하는 방법만 살펴본다.

### Installing

react-native-splash-screen을 설치한다.

```bash
yarn add react-native-splash-screen
```

### Android

아래 순서로 프로젝트 내부 빌드 파일들을 수정한다.

**settings.gradle**

```java
include ':react-native-splash-screen'
project(':react-native-splash-screen').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-splash-screen/android')
```

**android/app/build.gradle**

compile-time dependency를 추가한다.

```java
...
dependencies {
    ...
    implementation project(':react-native-splash-screen')
}
```

**MainApplication.java**

패키지를 추가해준다. 아래와 같이 `SplashScreenReactPackage` 를 추가하는데, 패키지가 이미 존재한다는 문제가 발생하면 getPackages 함수 내부의 수정 내용을 없앤다. import 만 해주면 자동으로 패키지를 추가해주기 때문에 중복 문제가 발생한다.

```java
// react-native-splash-screen >= 0.3.1
import org.devio.rn.splashscreen.SplashScreenReactPackage;
// react-native-splash-screen < 0.3.1
import com.cboy.rn.splashscreen.SplashScreenReactPackage;

public class MainApplication extends Application implements ReactApplication {

    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        public boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
            new SplashScreenReactPackage()  //here
            );
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }
}
```

**MainActivity.java**

MainActivity의 onCreate 함수 내부에 `SplashScreen.show(this);` 라인을 추가하여 스플래시 화면이 출력되도록 한다.

```java
import android.os.Bundle; // here
import com.facebook.react.ReactActivity;
// react-native-splash-screen >= 0.3.1
import org.devio.rn.splashscreen.SplashScreen; // here
// react-native-splash-screen < 0.3.1
import com.cboy.rn.splashscreen.SplashScreen; // here

public class MainActivity extends ReactActivity {
   @Override
    protected void onCreate(Bundle savedInstanceState) {
        SplashScreen.show(this);  // here
        super.onCreate(savedInstanceState);
    }
    // ...other code
}
```

**app/src/main/res/layout/launch_screen.xml**

layout 폴더가 없으면, layout 폴더를 생성하여 추가한 후 아래와 같은 xml 파일을 작성한다. 가장 중요한 부분은, `android:src="@drawable/launch_screen"` 이곳인데, 이미지의 확장자명을 제외한 파일 이름을 @drawable/ 뒤에 입력해야한다.

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageView android:layout_width="match_parent" android:layout_height="match_parent" android:src="@drawable/launch_screen" android:scaleType="centerCrop" />
</RelativeLayout>
```

**drawble**

이후 res/drawable/ 디렉토리 내부에 launch_screen.png 와 같은 이미지를 복사한다.

**App.tsx**

splash 이미지를 숨길 타이밍을 정한다.

```java
useEffect(() => {
    setTimeout(() => {
      SplashScreen.hide();
    }, 1000);
  }, []);
```
