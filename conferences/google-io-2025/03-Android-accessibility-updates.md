

## Visions 

### Talkback
- Talkback 기능에서 이미지를 Gemini 로 읽을 수 있게 변경되었다.
- Talkback 유저가 더 자세한 사항을 위해 Gemini에게 질문가능 하도록 변경되었다.
- Talkback 을 이용하여 화면 검색이 가능하도록 변경되었다.

### Text
- 저시력자를 위해 대비를 늘려주는 Outline Text 기능이 추가되었다.

### Pixel specific feature
- Pixel Magnifier라는 기능을 통해서 이미지를 확대하여 텍스트를 더 용이하게 읽고 이미지 내의 검색이 가능하다.
- Guided Frame이라는 기능을 통해서 사진촬영시 가이드를 Talkback으로 받을 수 있다.

## Accessilbity

- Live Caption - Expressive Caption
- Live Transcribe
- Hearing Aids

## Dexterity

물리적 키보드 접근성이 강화되어 기능이 추가되었다.
- mouse 키: 키보드로 마우스 커서 접근가능
- sticky 키: 여러 키를 눌러야하는 단축키를 하나의 키로 접근가능
- slow 키: 키보드 입력 시간을 느리게 제어하여 길게 눌러도 하나의 입력으로 제어
- bounce 키: 빠른 같은 키 입력을 무시하는 제어 기능

## API updates

Android 16에서 화면내의 요소를 더 자세히 표시하기 위해 API가 추가되었다.

- `AccessibilityNodeInfo` class
- `AccessilbiityEvent` class

### Supplemental description

하위 요소의 contentDescription을 override하지 않는 방법으로 semantic을 제어하기 위한 기능이다. `View`의 XML에서 `android:supplementalDescription`을 추가하거나 `setSupplementalDescription`함수를 통해서 제어할 수 있다.

### New states - Expanded & Collapsed State

화면 요소에 대한 expanded / collapsed 상태가 추가되어 제어할 수 있다. 화면 요소가 변하는 경우에는 접근성이 변경되었음을 notify해야한다.

```kt
class Container: LinearLayout {
    override fun onInitializeAccessbilityNodeInfo(info: AccessbilityNodeInfo?) {
        super.onInitializeAccessbilityNodeInfo(info)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODE.BAKLAVA) {
            info?.expandedState = mExpanded ?
                AccessbilityNodeInfo.EXPANDED_STATE_EXPANDED :
                AccessbilityNodeInfo.EXPANDED_STATE_COLLAPSED
        }
    }

    fun setExpanded(expanded: Boolean) {
        mExpanded = expanded
        if (accessbilityManager.isEnabled()) {
            val event = AccessbilityEvent(AccessbilityEvent.TYPE_WINDOW_CONTENT_CHANGED)
            event.setContentChangeTypes(AccessbilityEvent.CONTENT_CHANGE_TYPE_EXPADNED)
            sendAccessibilityEventUnchecked(event)
        }
    }
}
```

### Partially checked control

하위 체크박스도 모두 체크되어야 체크되는 일부 체크상태에 대한 접근성 상태도 추가되었다.

```kt
class TriStateCheckBox: View {
    override fun onInitializeAccessbilityNodeInfo(info: AccessbilityNodeInfo?) {
        super.onInitializeAccessbilityNodeInfo(info)
        info?.setCheckable(true)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODE.BAKLAVA) {
            val checkedState = when (mCheckedState) {
                FALSE -> AccessbilityNodeInfo.CHECKED_STATE_FALSE
                TRUE -> AccessbilityNodeInfo.CHECKED_STATE_TRUE
                PARTIAL -> AccessbilityNodeInfo.CHECKED_STATE_PARTIAL
            }
            info?.setChecked(checkedState)
        } else {
            info?.setChecked(mCheckedState == CHECKED_STATE_TRUE)
        }
    }

    fun setCheckedStatee(checkedState: Int) {
        mCheckedState = checkedState
        if (accessbilityManager.isEnabled()) {
            val event = AccessbilityEvent(AccessbilityEvent.TYPE_WINDOW_CONTENT_CHANGED)
            event.setContentChangeTypes(AccessbilityEvent.CONTENT_CHANGE_TYPE_CHECKED)
            sendAccessibilityEventUnchecked(event)
        }
    }
}
```

### Required input fields

입력이 필수인 입력란의 접근성 제어도 추가되었다.

```kt
class RequiredEditText: View {
    override fun onInitializeAccessbilityNodeInfo(info: AccessbilityNodeInfo?) {
        super.onInitializeAccessbilityNodeInfo(info)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODE.BAKLAVA) {
            info?.fieldRequired = true
        }
    }
}
```

### Indeterminate ranges

화면에서 로딩 중이지만 수치가 없는 로딩의 경우에는 접근성 제어가 없었지만 추가되었다.


```kt
class LoadingIndicator: View {
    override fun onInitializeAccessbilityNodeInfo(info: AccessbilityNodeInfo?) {
        super.onInitializeAccessbilityNodeInfo(info)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODE.BAKLAVA) {
            info?.setRangeInfo(AccessbilityyNodeInfo.RangeInfo.INDETERMIATE)
        }
    }
}
```

## Best practices

### Suppoorting Talkback image description

`Compose`나 `ImageView`를 사용하는 대부분의 경우에는 자동으로 지원되지만 커스텀 뷰의 경우에는 변경이 필요하다.

```kt
class SomeView: View {
    override fun getAccessbilityClassName(): CharSequence {
        return "android.widget.ImageView"
    }
}
```

### Deprecation of Announcement API

사용자에게 사용성을 저해하는 접근성을 제시할 수 있는 두개의 API를 deprecate하였다.

- `TYPE_ANNOUNCEMENT`에 대한 `AccessilibityEvent` 송출
- `View.announceForAccessibility` 함수

대신 시멘틱에 더 적합한 접근성 기능을 사용해야 한다. 예시로는 아래와 같다.

- Label control
- Window & Pane titles
- Live Regiens

### Accessibility Check for Compose UI Tests

Compose 1.8에서 `AndroidComposeTestRule` 을 통해서 접근성을 테스트할 수 있게 되었다. 사용자에게 적절한 접근성을 제공하지 못하는 경우 exception을 발생시켜 자동으로 확인할 수 있다.

```kt
class AccessibilityTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<ComponentActivity>()

    @Test
    fun missingContentDescription() {
        composeTestRule.setContent {
            Box(
                modifier = Modifier
                    .size(50.dp)
                    .clickable { }
                    .semantics {
                        contentDescription = ""
                    }
            )
        }

        composeTestRule.enableAccessbilityChecks()
        composeTestRule.onRoot().tryPerformAccessibilityChecks()
    }

    
    @Test
    fun missingContentDescription_usingCustomAccessibilityValidator() {
        composeTestRule.setContent {
            Box(
                modifier = Modifier
                    .size(50.dp)
                    .clickable { }
                    .semantics {
                        contentDescription = ""
                    }
            )
        }

        val accessbililtyValidator =
            AccessobilityValidator().also {
                it.setThrowExceptionFor(AccessibilityCheckResultType.WARNING)
            }
        composeTestRule.enableAccessbilityChecks(accessbililtyValidator)
        composeTestRule.onRoot().tryPerformAccessibilityChecks()
    }
}

```