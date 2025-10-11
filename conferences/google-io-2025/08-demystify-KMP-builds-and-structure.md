## Background

KMP는 개발 업무에 있어 장점을 갖는다.

- 55%의 코드를 공유할 수 있었고,
- 40% 빠른 속도로 개발할 수 있었다.

이를 위해서는 비즈니스 로직을 모든 플랫폼에 따라 공유하면 된다.

## KMPify

Jetpack Compose를 이용하여 여러 플랫폼의 기능을 구현할 수 있다. 플랫폼별로 검증된 정도에 따라 티어 1, 2, 3으로 나눠서 관리하고 있으며, 기능별로는 Stable, Commonized, Alpha 버전으로 관리하고 있다.

Android studio에서 모듈을 생성하거나 프로젝트 생성으로 시작할 수 있다.

## Gradle

Android 프로젝트에서 KMP으로 전환하기 위해서는 gradle을 수정해줄 필요가 있다.

우선 Android library module을 KMP library module로 변경해야한다.

```gradle
// PLUGINS
plugins {
    id("org.jetbrains.kotlin.multiplatform")
    id("com.android.kotlin.multiplatform.library")
}

kotlin { ... }
```

이후에 Android / iOS 용 빌드 target 설정을 해주어야 한다.

```gradle
// PLUGINS
plugins { ... }

kotlin { 
    // TARGETS
    androidLibrary { ... }
    iosX64 { ... }
}
```

다음으로 source set을 지정해야한다. 기본적으로 `<target>Main`, `<target>Test` 에 대한 source set이 생성되며, 사용자 정의가 가능하다.

```gradle
// PLUGINS
plugins { ... }

kotlin { 
    // TARGETS
    androidLibrary { ... }
    iosX64 { ... }
    
    // SOURCE SETS
    sourceSets {
        // custom source set
        androidAndIosMain {
            dependencies { ... }
        }

        // target source set
        androidMain {
            dependsOn(androidAndIosMain)
        }

        // target source set
        iosX64Main {
            dependsOn(androidAndIosMain)
        }
    }
}
```

### Project configuration recommendation

추천하는 실전 예제로 어느 플랫폼에도 종속적이지 않은 common한 source set을 분리하고 공용으로 사용하도록 구조화 하는 것이 좋다. 또한 android의 경우 Jvm 테스트를 위한 `androidHostTest` source set과 integration device 테스트를 위한 `androidDeviceTest` source set을 구분하는 것을 추천한다.

```gradle
plugins { ... }

kotlin {
    androidLibrary { ... }
    iosX64 { ... }
    
    sourceSets {
        // custom source set
        commonMain {
            dependencies { ... }
        }

        // Android source set
        androidMain { ... }
        androidHostTest { ... }
        androidDeviceTest { ... }

        // iOS source set
        iosX64Main { ... }
        iosX64Test { ... }
    }
}
```

common의 의미는 아래와 같다.

- Common은 target이나 platform 의존적이지 않다.
- 모든 곳에서 의존해도 되는 default 이름이다.
- common source set은 정의된 타깃을 위해 컴파일만 된다.


## Publish

library 배포를 여러 형태로 할 수 있다.

- JAR / AAR: JVM / Android target 용 바이너리
- Klib: non-JVM, non-Android target 용 바이너리
- XCFramework: Apple platform 용 바이너리

iOS platform으로 통합하기 위해서는 swift package manager나 CocoaPods를 사용하여 통합해야 한다.
