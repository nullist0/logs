## Background

- 휴대폰만 사용하는 사용자보다 태블릿 겸용 유저가 사용량이 3배 더 많았음


Adaptive android를 지원해야하는 이유

1. Android Auto에서도 지원이 가능해져 5억개의 디바이스에서도 지원이 가능해짐
2. Android 16에서 데스크톱 윈도우 모드에서도 지원이 가능함
3. Android XR에서도 지원이 가능함

폼 팩터를 지원하지 않으면 휴대전화 사용자도 상실할 수 있음

## What is Adaptive

Adaptive이란 모든 안드로이드 기기에서 구동 가능하도록 하는 것이다.
1. 모든 폼 팩터에 적용하며, 
2. 여러 디바이스 사용자가 사용 가능하며, 
3. 새로운 폼팩터에도 적용 가능하도록
공학적 기반을 제공하는 것이다.

## How apply Adaptive

- Compose Adaptive Library
- Material Canonical layout

### Without navigation 3 library

Navigation 3를 사용하지 않고 Destination을 통합하고 분할할 수 없기 때문에 화면 분할을 해당 destination에서 구현을 한다. 리스트목록과 실제 내용을 각 화면으로 갖는 경우 `ListDetailPaneScaffold`를 사용할 수 있다. 이와 다른 유스케이스를 갖는 화면의 경우에는 `NavigableSupportingPaneScaffold`을 사용하여 커스텀할 수 있다.

```kt
ListDetailPaneScaffold(
    listPane = { /* Composable */ },
    detailPane = { /* Composable */ },
    // Drag handle / pane expansion
    painExpansionState = rememberPaneExpansionState(scaffoldValue),
    paneExpansionDragHandle = { state ->  ... }
)
```

### With navigation 3

화면 크기에 따라 Composable이 서로 다른 destination을 가질수도 있고, 하나의 destination에서 표시될 수 있다. 이를 navigation 3에서 여러 destination을 하나로 합치고 분할하여 표시하는 구현을 제공한다.

```kt
is Pane.ChatsList -> NavEntry(
    key = backStackKey,
    // destination에서 채팅방 리스트가 어떻게 표시될지 메타데이터를 제공한다.
    metadata = ListDetailSceneStrategy.listPane()
)

is Pane.ChatThread -> NavEntry(
    key = backStackKey,
    // destination에서 채팅방이 어떻게 표시될지 메타데이터를 제공한다.
    metadata = ListDetailSceneStrategy.detailPane()
)
```

## Levitate & Reflow

화면 크기상 표시되고 있던 pane이 화면 전환 이후로도 표시되어야 할때 두가지 개념을 제공한다.

- Levitate: pane을 다이얼로그처럼 표시하는 방법
- Reflow: pane을 화면에 그대로 배치하되 orientation만 변경되어 표시하는 방법

### How - Levitate

```kt
// Centered dialog style
adaptStrategies = ListDetailPaneScaffoldDefaults.adaptStraties(
    extraPaneAdaptStrategy = AdaptStrategy.Levitate(
        // Levitate 사용 조건 (ex. 한 pane만 사용되는 화면인 경우)
        strategy = AdaptiveStrategy.Levitate.Strategy.SinglePaneOnly,
        // 배치 위치
        alignment = Alignment.Center,
        // 다이얼로그 아래로 dimmed color 등의 처리
        scrim = Scrim()
    )
)

// Bottom sheet style
adaptStrategies = ListDetailPaneScaffoldDefaults.adaptStraties(
    extraPaneAdaptStrategy = AdaptStrategy.Levitate(
        strategy = AdaptiveStrategy.Levitate.Strategy.SinglePaneOnly,
        alignment = Alignment.BottomCenter,
        scrim = Scrim()
    ),
    extraPane = { 
        AnimatedPane(
            modifier = Modifier.dragToResize(...)
        )
    }
)
```

### How - Reflow

```kt
adaptStrategies = ListDetailPaneScaffoldDefaults.adaptStraties(
    extraPaneAdaptStrategy = AdaptStrategy.Reflow(
        targetPane = ListDetailPaneScaffoldRole.Detail
    )
)
```

## XR Component Overrides

XR 디바이스에서는 Pane이 각 윈도우로 분리될 수 있다. 이를 설정하기 위해서는 `EnableXrComponentOverrides`를 사용하면 된다.

## Adaptive Layout on Compose

### Background - Why compose

- Compose에서는 라이브러리로 제공되기에 실행 기기에 의존하지 않는다.
- Compose에 여러 폼팩터나 입력 지원을 위한 컨텍스트 메뉴가 추가되었다.
- Compose는 런타임에서 화면이 구성되므로 XML 대응이 불필요하다.

### Layout decision - Breakpoints

Pane의 갯수를 판정하기 위해서 androidx.window 라이브러리에서 제공한다.

```kt
val windowDpSize = with(LocalDensity.current) {
    LocalWindowInfo.current.containerSize.toSize().toDpSize()
}
val windowSizeClass = WindowSizeClass
    .BREAKPOINTS_V1
    .computeWindowSizeClass(
        windowDpSize.width.value,
        windowDpSize.height.value
    )

// HEIGHT BASED BREAKPOINT
if (
    windowSizeClass.isHeightAtLeastBreakPoint(
        WindowSizeClass.HEIGHT_DP_EXPANDED_LOWER_BOUND
    )
) {
    // 900dp => EXPANDED HEIGHT
} else if(
    windowSizeClass.isHeightAtLeastBreakPoint(
        WindowSizeClass.HEIGHT_DP_MEDIUM_LOWER_BOUND
    )
) {
    // 480dp => MEDIUM HEIGHT
} else {
    // COMPACT HEIGHT
}

// WIDTH BASED BREAKPOINT
if (
    windowSizeClass.isWidthAtLeastBreakPoint(
        WindowSizeClass.WIDTH_DP_EXPANDED_LOWER_BOUND
    )
) {
    // 840dp => EXPANDED WIDTH
} else if(
    windowSizeClass.isWidthAtLeastBreakPoint(
        WindowSizeClass.WIDTH_DP_MEDIUM_LOWER_BOUND
    )
) {
    // 600dp => MEDIUM WIDTH
} else {
    // COMPACT WIDTH
}
```

사용 가이드로는 아래와 같다.
- 세로로는 스크롤이 가능한 경우가 있으므로 가로폭이 대체로 더 중요하다.
- MEDIUM breakpoint에서는 화면 크기가 휴대폰과 유사하므로 하나의 컨텐츠 pane만 표시할 수 있다.
- EXPANDED breakpoint에서는 사용자의 설정에 따라 navigation drawer를 동적으로 표시할 수 있다.
- LARGE breakpoint에서는 화면 크기가 충분히 크므로 naviation drawer와 하나의 컨텐츠 pane을 표시할 수 있다.
- EXTRA LARGE breakpoint에서는 화면 크기가 매우 크므로 3개의 컨텐츠 pane을 표시할 수 있다. `BREAKPOINT_V2`를 사용해야한다.

### Compose Test with Adaptive Layout

`DeviceConfiugrationOverride`를 이용하여 `ComposeTestRule.setContent`에서 표시할 컴포저블의 환경을 변경할 수 있다. 예를 들어 아래의 변수들을 제어할 수 있다.

- `WindowInsets`
- `FontSize`
- `LayoutDirection`
- `ForcedSize`

## Support all form-factors

### Availability

여러 기기를 모두 하나의 앱으로 제공하기 위해서 사용 불가능한 경우를 줄일 필요가 있다. 기능상 반드시 필요한 기능이 아닌 경우 `android:required`를 통해서 설치 불가능한 경우를 방지할 수 있다. 예를 들어 소셜 미디어 앱에서 카메라 기능을 사용할 수도 있지만 필수는 아니기에 수정할 필요가 있다. 

```xml
<use-features
    android:name="android.hardware.camera"
    android:required="false"/>
```

### Input

대부분의 앱에서는 트랙패드나 터치 혹은 마우스만 제공하면 충분하지만, 스타일러스 펜을 지원하는 경우에는 사용성이 더 좋아질 수 있다. Ink API를 통해서 더 나은 사용성을 제공할 수 있다.

### Resizability

여러 폼 팩터와 연결된 디스플레이 지원과 관련하여 앱의 화면 크기가 다양하게 변경될 수 있다. Android 16에서는 600dp 이상의 width를 갖는 기기에서 orientation, aspect ratio, resizability restrictions로 인한 화면 크기 제약을 무시한다.

## Reference

- SocialLite git: https://github.com/android/socialite
