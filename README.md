# THREE.JS활용 렌더링

해당 저장소는 경천사지10층석탑 을 렌더링 하기 위한

WebGL도구로 ThreeJS를 활용하여 코딩한 내용이다.

ThreeJS사용을 위해서는 해당 세 개의 script를 필요로 한다.

```html
<script src="https://threejs.org/build/three.js"></script>
<script src="https://cdn.rawgit.com/mrdoob/three.js/master/examples/js/loaders/GLTFLoader.js"></script>
<script src="https://threejs.org/examples/js/controls/OrbitControls.js"></script>
```

과연 우리가 시각화된 3D요소를 보려면 어느 요소들이 필요할까?

```javascript
// 배경
const scene = new THREE.Scene();
scene.background = new THREE.Color("white");

// 카메라
const camera = new THREE.PerspectiveCamera(
  50,
  window.innerWidth / window.innerHeight,
  0.1,
  100000
);
camera.position.x = -200;
camera.position.y = 120;
camera.position.z = 0;
camera.lookAt(scene.position);
camera.getEffectiveFOV();

// 모델
let model;

// 태양
const directlight = new THREE.DirectionalLight(0xffffff, 0.5, 3000);
directlight.shadow.camera.near = 10;
directlight.shadow.camera.far = 2000;
const shadowBlur = 50;
directlight.shadow.camera.left = -shadowBlur;
directlight.shadow.camera.right = shadowBlur;
directlight.shadow.camera.top = shadowBlur;
directlight.shadow.camera.bottom = -shadowBlur;
directlight.shadow.mapSize = new THREE.Vector2(1024, 1024);
scene.add(directlight);
directlight.castShadow = true;

//  빛
const spotlight1 = new THREE.SpotLight(0xffffff, 0.5, 500);
spotlight1.position.set(100, 20, 0);
spotlight1.lookAt(0, 20, 0);
scene.add(spotlight1);
const spotlight2 = new THREE.SpotLight(0xffffff, 0.5, 500);
const spotlight3 = new THREE.SpotLight(0xffffff, 0.5, 500);
const spotlight4 = new THREE.SpotLight(0xffffff, 0.5, 500);
spotlight2.position.set(0, 10, 100);
spotlight2.lookAt(0, 20, 0);
scene.add(spotlight2);
spotlight3.position.set(0, 20, -100);
spotlight3.lookAt(0, 20, 0);
scene.add(spotlight3);
spotlight4.position.set(-100, 20, 0);
spotlight4.lookAt(0, 20, 0);
scene.add(spotlight4);
const light = new THREE.AmbientLight(0xffffff, 4);
scene.add(light);

//  출력장치
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 컨트롤러
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enablePan = true;
controls.zoomSpeed = 3;
controls.rotateSpeed = 0.7;
controls.minDistance = 20;
controls.maxDistance = 200;
controls.maxPolarAngle = Math.PI / 2;
controls.enableKeys = true;
controls.keys = {
  LEFT: 65, //left arrow
  UP: 87, // up arrow
  RIGHT: 68, // right arrow
  BOTTOM: 83, // down arrow
};
controls.keyPanSpeed = 15.0;
```

배경, 시선, 태양, 밝기 등등 너무나도 많은 기본 요소들을 필요로 한다.

## 로딩화면

로딩 시 3D 도구 즉 WebGL같은 경우에는 시간이 많이 소요되기 때문에 로딩 바 혹은

사용자에게 시각화 하여 로딩중임을 표시해줄 수 있는 도형이 있다면 매우 좋을 것 이다.

<img src="GitImg\Loading.png">

내가 만든 프로젝트 의 경우에는 해당 정육면체가 회전하며 로딩중 임을 사용자에게 알려준다.

```javascript
// 로딩
const loadingMaterials = [
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading.png"),
    side: THREE.DoubleSide,
  }),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading.png"),
    side: THREE.DoubleSide,
  }),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading2.png"),
    side: THREE.DoubleSide,
  }),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading2.png"),
    side: THREE.DoubleSide,
  }),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading.png"),
    side: THREE.DoubleSide,
  }),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/Loading.png"),
    side: THREE.DoubleSide,
  }),
];
const loading = new THREE.Mesh(
  new THREE.BoxGeometry(80, 80, 80),
  loadingMaterials
);
loading.position.set(0, 0, 0);
loading.rotation.z = 0;
```

즉 로딩 시 여섯 면에 해당하는 정보를 모두 부여하며 해당 객체의 Mesh는 BoxGeometry임을 선언해준다.

## 모델

자 이렇게 로딩까지 구현이 되었다면 로딩이 끝나고 렌더링 할 모델이 있어야 할 것이다.

<img src="GitImg\Model.png">

해당 모델을 불러오기 위해서는 아래와 같은 코드가 필요하며

위 모델은 바닥과 같이 있는것이 아니라 바닥 모델 또한 필요하다.

```javascript
// 바닥
const floor = new THREE.Mesh(
  new THREE.BoxGeometry(100, 1, 100),
  new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load("/floor.png", function (texture) {
      //바닥 x,y축를 기준으로 텍스쳐 10누기
      floor.material.map.repeat.x = 3;
      floor.material.map.repeat.y = 3;
      floor.material.map.wrapS = THREE.RepeatWrapping;
      floor.material.map.wrapT = THREE.RepeatWrapping;
    }),
    side: THREE.DoubleSide,
  })
);
floor.position.set(0, 0, 0);
floor.receiveShadow = true;

// 3D gltf 파일 불러옴
const loader = new THREE.GLTFLoader();
loader.load(
  "/KC_MODEL1.gltf",
  (gltf) => {
    gltf.scene.scale.set(0.05, 0.05, 0.05);
    gltf.scene.rotation.set(-0.013, -0.03, -0.0093);
    gltf.scene.position.set(0, 0.8, 0);
    model = gltf.scene;
    scene.add(gltf.scene);
    scene.add(floor);
    scene.remove(loading);
  },
  (xhr) => {
    scene.add(loading);
    console.log(Math.floor((xhr.loaded / xhr.total) * 100) + "%");
  },
  (err) => {
    console.error(err);
  }
);
```

위와같이 컨트롤 할 수 있는 모델이 생성되었다.

그치만 아쉬운 점은 가만히 있는 모델이기에 둘러보기에 용이하지도 않을 뿐더러

사진인 것 같은 느낌마저 있다.

## 돌려보기

돌리는 것은 자바스크립트의 setInterval함수와 ThreeJs의 명령어들을 조합하여 사용하면 매우 간단한데,

```javascript
// setInterval
const animate = function () {
  requestAnimationFrame(animate);

  if (model) {
    model.rotation.y += 0.001;
    floor.rotation.y += 0.001;
  }

  if (loading) {
    loading.rotation.y += 0.01;
    loading.rotation.x += 0.01;
  }

  controls.update();

  renderer.render(scene, camera);
};

animate();

// 창 조절 시 꽉 찬 화면
window.addEventListener("resize", function () {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});
```

위와 같이 표시해주면 천천히 돌아가며 이제서야 움직이는 3D 형상을 보여준다.
