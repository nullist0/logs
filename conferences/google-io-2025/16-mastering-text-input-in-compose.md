## State-base TextFields

value & callback을 구현된 `TextField`가 State 기반의 Composable API가 추가되었다. Key features는 아래와 같다.

- value & callback이 아닌 새로운 State hoisting 패러다임 적용
- `onValueChange` callback을 `InputTransformation`으로 변경
- `VisualTransformation`을 `OutputTransformation`으로 변경
- single/multi line 설정 인자를 받도록 변경

### Sample

```kt
val usernameState = rememberTextFieldState()
TextField(
    state = usernameState,
    inputTransformation = InputTransformation { ... },
    outputTransformation = OutputTransformation { ... },
    lineLimits = TextFieldLineLimits.SingleLine,
    placeholder = { Text("Username") }
)
```

## SecureTextField

비밀번호와 같은 입력을 받기 위한 `TextField`가 추가되었다. key feature는 아래와 같다.

- 비밀번호 숨기기가 기본적으로 활성화
- 잘라내기, 복사 기능 방지
- 입력 값 조건 추가 기능

### Sample

`SecureTextField`는 single line이므로 설정하지 않아도 된다.

```kt
val passwordState = rememberTextFieldState()
SecureTextField(
    state = passwordState,
    placeholder = { Text("Password") }
)
```

8글자 이상 입력과 같이 값에 제한을 두는 경우 인자로 추가할 수 있다.


```kt
val passwordState = rememberTextFieldState()
SecureTextField(
    state = passwordState,
    placeholder = { Text("Password") },
    supportingText = {
        Text("Password must be at least 8 characters")
    },
    isError = passwordState.text.length in 1..7
)
```

## Autofill

`TextField`의 content type에 따라 유저 기기에 저장된 값으로 자동 채우기를 수행하도록 할 수 있다. `Modifier.semantics` 으로 `contentType` 설정할 수 있다.

### Sample - Sign up

```kt
// Username 에 대한 Autofill 지원
TextField(
    modifier = Modifier.semantics {
        contentType = ContentType.NewUsername
    }
)

// Password 에 대한 Autofill 지원
SecureTextField(
    modifier = Modifier.semantics {
        contentType = ContentType.NewPassword
    }
)

// Autofill을 수행하도록 commit한다.
val autofillManager = LocalAutofillManager.current
Button({ autofillManager?.commit() }) {
    Text("Sign up")
}
```

### Sample - Log in

가입시와 달리 패스워드 매니저에 의해 저장된 값을 사용하기 위해서는 `New` prefix가 없는 `ContentType`을 사용해야한다.

```kt
// Username 에 대한 Autofill 지원
TextField(
    modifier = Modifier.semantics {
        contentType = ContentType.Username
    }
)

// Password 에 대한 Autofill 지원
SecureTextField(
    modifier = Modifier.semantics {
        contentType = ContentType.Password
    }
)
```

### Sample - 2FA

SMS OTP와 같은 2FA도 동일한 방식으로 제공할 수 있다.

```kt
// OTP Code를 auto fill할 수 있도록 지원
TextField(
    modifier = Modifier.semantics {
        contentType = ContentType.SmsOtpCode
    }
)
```

## Obfuscation

`SecureTextField` 를 사용할때 비밀번호 전체 혹은 일부를 보여줄 수 있도록 인자 `textObfuscationMode`가 추가되었다. 3가지 설정이 있으며,

- `Visible`: 모든 비밀번호가 보임
- `Hidden`: 모든 비밀번호가 보이지 않음
- `RevealLastTyped`: 마지막으로 입력된 비밀번호만 보임

### Sample

```kt
var passwordVisible by remember { 
    mutableStateOf(false) 
}
SecureTextField(
    trailingIcon = {
        Icon(
            if (passwordVisible) {
                Icons.Filled.VisibilityOff
            } else {
                Icons.Filled.Visbility
            },
            modifier = Modifier.clickable {
                passwoardVisible = !passwordVisible
            }
        )
    },
    textObfuscationMode = if (passwordVisible) {
        Visible
    } else {
        RevealLastTyped
    }
)
```

## Custom layout

`TextField`는 Material design에 맞게 설계되어 커스텀한 레이아웃을 제공하기 위해서는 `BasicTextField`의 `decorator` 인자를 사용할 수 있다. `innerTextField`를 이용하여 복사 붙여넣기 등의 기존의 인터렉션을 사용할 수 있으나, 인터렉션이 필요 없는 경우 임의로 텍스트를 표시할 수 있다.

### Sample

```kt
BasicTextField(
    decorator = {
        // ...
        innerTextField()
        // ...
    }
)
```

## Modifier.contentReceiver

텍스트 입력 말고도 리치 컨텐츠를 지원할 수 있다. 예를 들어 이미지 복사 붙여넣기, 키보드의 GIF 입력, 이미지 드래그 앤 드롭 기능을 지원할 수 있다. 이때 `Modifier.contentReceiver`를 사용해야한다.

`Modifier` 기능이므로 `TextField`가 아닌 다른 Composable에도 지원이 가능하다. 이 경우 키보드이 GIF 입력이나 이미지 복사 붙여넣기는 불가하나, 드래그 앤 드롭 기능이 지원된다.

### Sample

```kt
TextField(
    modifier = Modifier.contentRececiver { transferableContent ->
        // transferableContent로부터 Uri를 획득하여 상태로 구성할 수 있다.
        chatViewModel.handleContent(transferableContent)
    }
)
```

## Auto-size

`TextField`의 parent constraint에 맞게 텍스트 크기를 줄이고 늘리는 기능을 제공할 수 있다.

### Sample

```kt
TextField(
    autoSize = TextAutoSize.StepBased()
)
```

## Formatting input / output

키보드 입력과 화면에 표시되는 텍스트를 `InputTransformation`과 `OutputTransformation`으로 변경할 수 있다. 텍스트 변경시 각 인자의 적용은 아래와 같이 적용된다.

1. 텍스트 변경이 발생한다.
2. `InputTransformation`으로 값을 변경하여 `TextFieldState`에 반영한다.
3. `TextFieldState` 값을 이용하여 `OutputTransformation`으로 변경한다.
4. 3번의 결과를 `TextField`에 반영한다.

### Sample

미국 전화번호 형태로 포매팅하는 예시를 보자.

- 입력은 숫자만 있어야 한다.
- 번호수는 10개이다.
- 화면에는 `(123)1234-5678`의 형태로 표시되어야 한다.

```kt
TextField(
    inputTransformation = InputTransformation.maxLength(10).then {
        if (!TextUtils.isDigitsOnly(asCharSequence())) {
            revertAllChanges()
        }
    },
    outputTransformation = PhoneNumberOutputTransformation()
)

class PhoneNumberOutputTransformation: OutputTransformation {
    override fun TextFieldBuffer.transfromOutput() {
        if (length > 0) insert (0, "(")
        if (length > 4) insert (0, ")")
        if (length > 8) insert (0, "-")
    }
}
```
