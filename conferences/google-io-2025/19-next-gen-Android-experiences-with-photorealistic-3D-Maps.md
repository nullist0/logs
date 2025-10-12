## Background

Google Map 3D 뷰가 JavaScript API로 공개되었었는데 Android로도 추가되었다.

## Use Map 3D SDK

### Initialize

3D map을 이용하기 위해서 `Map3dView` 클래스를 사용해야한다. XML로 정의할 수 있으며, 초기화 로직이 필요하다. callback을 등록하여 초기화 완료를 들을 수 있다.

```kt
class MainActivity : Activity(), OnMap3dViewReadyCallback {
    private lateinit var map3dView: Map3dView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)

        map3dView = findViewById(R.id.map3dView)
        map3dView.onCreate(savedInstanceState)
        map3dView.getMap3dViewAsync(this)
    }

    override fun onMap3dViewReady(googleMap3d: GoogleMap3D) {
        // This map is ready to be used here
        // googleMap3d으로 Camera 및 Marker 등의 설정이 가능하다. 
    }
}
```

### Locational classes

3D map 지원이 되면서 위치에 고도 (altitude)가 포함된 `LatLngAltitude` 클래스가 제공된다.

```kt
val NEUSCHWANSTEIN_COORS = latLngAltitude {
    latitude = 47.5
    longtitiude = 10.7
    altitude = 988.6
}
```

카메라 위치 클래스에도 기울임 값이 추가된다.

```kt
val DELICATE_ARCH_CAMERA = camera {
    // 카메라 중앙지점과 zoom 거리 설정
    center = NEUSCHWANSTEIN_COORS
    range = 140.0

    // ROLL PITCH YAW 시스템과 동일
    heading = 350.0
    tilt = 60.0
    roll = 0.0
}
```

### Camera

`GoogleMap3D` 클래스를 통해서 카메라 위치를 설정할 수 있다.
```kt
override fun onMap3dViewReady(googleMap3d: GoogleMap3D) {
    googleMap3d.setCamera(DELICATE_ARCH_CAMERA)
}
```

이동 위치 제한 등의 카메라 제한 사항도 설정 가능하다.

```kt
override fun onMap3dViewReady(googleMap3d: GoogleMap3D) {

    val cameraRestriction = cameraRestriction {
        ...
    }
    googleMap3d.setCameraRestriction(cameraResctriction)
}
```

카메라를 원하는 위치로 이동시키는 애니메이션을 실행 시킬 수 있다.

```kt
val flyToArch = flyToOption {
    endCamera = DELICATE_ARCH_CAMERA
    durationInMillis = 5_000 
}
googleMap3d.flyCameraTo(flyToArch)
```

카레라 위치를 고정한채로 주변을 회전하듯이 보여주는 애니메이션도 지원한다.

```kt
val flyAroundArch = flyAroundOption {
    endCamera = DELICATE_ARCH_CAMERA
    durationInMillis = 7_500
    rounds = 2.0 
}
googleMap3d.flyCameraAround(flyAroundArch)
```

### Marker

3D map에서도 마커를 설정할 수 있다. 마커를 설정할때 표시 방법을 설정할 수 있다.

- `AltitudeMode.ABSOLUTE`: 해당 지점에 표기되며, 해수면부터 계산한 altitude 와 동일한 고도에 표시된다.
- `AltitudeMode.CLAMP_TO_GROUND`: altitude가 무시되며 해당 지점의 ground와 동일한 고도에 표시된다.
- `AltitudeMode.RELATIVE_TO_GROUND`: 해당 지점의 ground부터 계산한 altitude 와 동일한 고도에 표시된다.
- `AltitudeMode.RELATIVE_TO_MESH`: 해당 지점의 3D mesh부터 계산한 altitude 와 동일한 고도에 표시된다.

```kt
val markerOptions = markerOptions {
    position = MARKER_COORDIATES
    zIndex = 1
    isExtruded = true
    label = "Relative to mesh"
    isDrawnWhenOccluded = true

    // 3D map에 marker가 어떤 고도로 표현되어야하는지 설정한다.
    altitudeMode = AltitudeMode.RELATIVE_TO_MESH
}

// 주어진 markerOption에 ID가 지정되어있고, 이미 생성되었다면 같은 ID를 갖는 marker를 반환하고 주어진 markerOption에 맞게 업데이트 한다.
// 주어진 markerOption에 ID가 지정되어있고, 생성전이라면 같은 ID를 갖는 새로운 marker를 생성하고 반환한다.
// 주어진 markerOption에 ID가 없다면, ID를 자동생성하여 새로운 marker를 생성하고 반환한다.
val marker = googleMap3d.addMarker(markerOptions)
```

### 3D Model

3D map에 원하는 3D 오브젝트를 추가할 수 있다.

```kt
val modelOption = modelOptions {
    url = "url of glb file"
    position = modelPosition
    altitudeMode = AltitudeMode.ABSOLUTE
    orientation = orientation {
        heading = modelHeading
        tilt = modelTilt
        roll = modelRoll
    }
    scale = vector3D {
        x = modelX
        y = modelY
        z = modelZ
    }
}
```

### Vector graphics

3D map에 벡터 그래픽도 추가할 수 있다. 두가지를 지원한다.

- polygon: 닫힌 path 도형
- polyline: 열린 path 도형

```kt
val polylineCoordinates = ImmutableList.of(
    latLngAltitude { 
        latitude = 41.897
        longitude = -87.622
        altitude = 0.0
    },
)
val polylineOptions = polylineOptions {
    coordinates = polylineCoordinates
    strokeColor = 0xFF87F5FF.toInt()
    strokeWidth = 16.0
    altitudeMode = AltitudeMode. CLAMP_TO_GROUND
    zIndex = 1
    outerColor = 0xFF87F5FF.toInt()
    outerWidth = 1.0
}
val polyline = googleMap3d.addPolyline(polylineOptions)
```
