## Background

XR 지원으로 앱의 차별성을 지원할 수 있다. XR 디바이스인 프로젝트 무한 개발을 통해서 릴리즈될 예정이다.

### Spaces

유저는 두가지 종류의 space를 이용하여 앱을 사용하게 된다.
- Home space
- Full space

Home space는 XR 공간에서 2D 스타일로 앱을 띄운 것이며, 여러 앱을 동시에 사용할 수 있다. Full space에서는 XR 공간에 하나의 앱을 3D 스타일로 띄우며, 3D content를 사용할 수 있게 되지만, 하나의 앱만 사용할 수 있다. 각 space간의 변경은 아래와 같은 API 호출을 통해 이뤄진다.

```kt
session.scene.spatialEnviornment.requestFullSpaceMode()
session.scene.spatialEnviornment.requestHomeSpaceMode()
```

## Virtual Enviornment

Full Space 상태에서는 사용자가 위치한 환경을 바꿀 수 있도록 Virtual Environment를 제어할 수 있다. 이를 이해하기 위해서는 두가지 개념이 필요하다.

- Skybox: Skybox는 유저 주변에 보이는 돔형태의 이미지이다.
- Geometry: Geometry는 유저 주변의 깊이감과 지형이다.

설정하기 위해서는 아래의 예제 코드를 사용할 수 있다.

```kt
// Virtual environment 설정
val exr = ExrImage.create(session, "environment.zip").await()
val geometry = GltfModel.create(session, "someModel.glb").await()
sesson.scene.spatialEnvironment.setSpatialEnvironmentPreference(
    SpatialEnvironment.SpetialEnvironmentPreference(exr, geomentry)
)

// Virtual environment 해제
session.scene.spatialEnvironment.setPassthroughOpacityPreference(1.0f)
```

## Videos

XR 환경에서는 영상을 3D 형태로 보여줄 수 있다. 180도 Video와 360도 Video 기능을 제공한다. Stereoscopic content를 제공하여 영상의 깊이감을 제공할수도 있다. 양 눈에 보이는 이미지의 각도를 조절하여 깊이감 착시를 일으킬 수 있다. 이는 프레임에 같이 저장된 이미지이며, HEVC 코덱에서 지원한다.

## SceneCore

Video를 유저에게 보여주기 위해서 SceneCore 라이브러리를 사용할 수 있다. SceneCore를 통해 생성한 모든 요소는 Entity라고 부른다. 유저에게 영상을 보여주는 Entity는 `SurfaceEntity` 클래스이며, 아래 팩토리 메서드로 생성할 수 있다.

```kt
// Entity 생성
val surfaceEntity = SurfaceEntity.create(
    session, // XR session class
    SurfaceEntity.StereoMode.MONO, // Stereoscopic or Monoscopic
    headPose,
    Vr360Sphere(1.0f)
)

// Entity 실행
val player = ExoPlayer.Builder(context).build().apply {
    setMediaItem(MediaItem.fromUri(uri))
    setVideoSurface(surfaceEntity.getSurface())
    prepare()
}
```

### Details

#### Poses

`Pose` 클래스는 Entity의 위치와 회전각도를 표현한다. `Quaternion`은 `Quaternion.fromEulerAngles` 함수를 통해서 생성할 수 있다.

```kt
class Pose(translation: Vector3, rotation: Quaternion)
```

#### Entities

Entity들은 View tree와 동일하게 parent-child 관계를 가질 수 있다. 회전과 위치는 parent 내에서 상대적으로 표현된다. Entity에 유저가 할 수 있는 행위를 등록할 수 있는데 이를 component라고 부른다. `Entity.addComoponent`를 호출하여 추가할 수 있으며, component에는 아래와 같은 예시가 존재한다.

- MovableComponent: Entity를 이동시킬 수 있는 행위를 제공한다.
- ResizableComponent: Entity 크기를 변형할 수 있는 행위를 제공한다.
- InteractableComponent: Entity와 상호작용할 수 있는 범용적인 행위를 제공한다. 클릭, 드래그 등 이벤트를 직접 제어할 수 있다.

#### 3D model

3D model을 XR 공간에 추가할 수 있다. 모델은 glTF 파일 포맷을 지원해야한다.

가장 간단하게 3D 모델을 보이는 방법은 아래와 같다. 화면 중앙에 표시된다.

```kt
val intent = Intent(Intent.ACTION_VIEW).apply {
    data = Uri.parse(modelUrl)
    setClassName("com.google.vr.sceneviewerxr", "com.google.vr.sceneviewerxr.SceneViewerXrActivity")
    setPackage("com.google.vr.sceneviewerxr")
}
startActivity(intent)
```

혹은 glTF 리소스를 직접 로드할 수 있다. 위치나 회전 혹은 애니메이션을 제어할 수 있다.

```kt
// glTF 파일을 직접 불러온다.
val tigerModel = GltfModel.create(session, modelFileName).await()
val tigerGltfEntity = GltfModelEntity.create(session, tigerModel).apply {
    // 위치나 회전, 크기를 제어할 수 있다.
    setScale(0.5f)
    setPose(Pose(Vector3(0f, 0f, -0.5f)))
}
// Animation을 실행할 수 있다.
tigerGltfEntity.startAnimation(
    loop = true,
    animationName = "Roar"
)
```

#### With ARCore

ARCore의 여러 기능과 같이 사용할 수 있는데, 여기서는 anchor와의 연동을 설명한다. anchor는 실제 세상에서의 위치를 뜻한다. anchor를 이용하여 3D model을 더 현실적인 위치에 표시할 수 있다.

```kt
// Anchor를 찾을 위치를 생성한다.
val pose = session.scene..transformPoseTo(
    paintingEntity.getPose(),
    session.core.perceptionSpace
)

// Anchor를 생성하고, 성공시 local storage에 저장한다.
when (val result = Anchor.create(session, pose)) {
    is AnchorCreateSuccess -> {
        // local storage에서 가져올 수 있는 UUID를 발급한다.
        val uuid = result.anchor.persist()
    }
    else -> { /* Anchor creation failed */ }
}

// UUID를 이용하여 Anchor를 가져오고 등록한다.
when (val result = Anchor.load(session, uuid)) {
    is AnchorCreateSuccess -> {
        val pose = session.scene.perceptionSpace.transformPoseTo(
            result.anchor.state.value.pose,
            session.core.activitySpace
        )
        paintingEntity.setPose(pose)
    }
    else -> { /* Anchor couldn't be found */ }
}
```

Hand tracking도 ARCore 기능을 이용하여 계산하여 처리할 수 있다. 손을 분석하여 얻는 26개의 벡터를 이용하여 손의 상태를 계산하고 이에 따른 처리를 추가할 수 있다.
