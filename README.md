# people
Threejs实现键盘控制人物行走跳动/Capsule碰撞体(胶囊体)/碰撞检测(Octree八叉树)/游戏第三人称/镜头跟随人物移动
学习博客 https://blog.csdn.net/baidu_29701003/article/details/129975094
1，功能介绍
Threejs实现键盘控制人物行走和跳动、使用八叉树Octree和胶囊体Capsule进行碰撞检测、人物头顶显示名称标签、镜头跟随人物移动并且镜头围绕人物旋转，类似游戏中第三人称。如下效果图


![](https://img-blog.csdnimg.cn/b4a0cf6f12ba47e49c10a307e76458ac.gif)



 2，功能实现
Octree是一种基于八叉树的数据结构，使用Octree进行碰撞检测可以提高性能，特别是在场景中存在大量物体的情况下。

Capsule（胶囊体）是一种3D几何体，它由两个球体和一个圆柱体组成。通常被用作游戏和虚拟现实应用中的角色或物体的碰撞体积。 

创建一个名为worldOctree的Octree对象，并将gltf.scene中的所有网格节点添加到Octree中。

Octree是一种空间划分数据结构，可用于更有效地进行3D碰撞检测。Octree通过将3D空间分解成八个相等的子空间，然后递归地将每个子空间再分解成八个子空间，以此类推。在这个过程中，Octree会将所有网格对象分配到合适的节点中。这样做的好处是，在进行碰撞检测时，可以避免对整个场景中的所有网格进行遍历，而只需遍历与当前视野和碰撞体相关的那些网格。这样，可以大大减少碰撞检测的计算量，提高性能。
```
const worldOctree = new Octree();
worldOctree.fromGraphNode(gltf.scene);
```
创建Capsule.js实例来表示一个胶囊体，然后将该实例添加到Octree中，以便进行空间分区和碰撞检测。例如：
```
const playerCollider = new Capsule(position1, position2, radius);
 
const result = worldOctree.capsuleIntersect(playerCollider);
```
其中，position1和position2是线段的两个端点的位置，radius是球的半径。创建Capsule.js实例后，可以使用该实例来创建一个包含几何体信息的playerCollider对象，并使用worldOctree.capsuleIntersect()方法将该对象添加到Octree中。

```
const result = worldOctree.capsuleIntersect(playerCollider);
```

它首先调用了 worldOctree 中的 capsuleIntersect 方法，将玩家的碰撞体 playerCollider 作为参数传入，得到碰撞检测的结果 result。 如果 result 存在，说明发生了碰撞，需要将玩家位置进行调整

```js
function playerCollisions() {
 
	const result = worldOctree.capsuleIntersect(playerCollider);
 
	playerOnFloor = false;
 
	if (result) {
 
		playerOnFloor = result.normal.y > 0;
 
		if (!playerOnFloor) {
 
			playerVelocity.addScaledVector(result.normal, -result.normal.dot(playerVelocity));
 
		}
 
		playerCollider.translate(result.normal.multiplyScalar(result.depth));
 
	}
 
}

```

这里使用了 result.normal.multiplyScalar(result.depth) 的方法，其中 result.normal 表示碰撞的法向量，result.depth 表示碰撞的深度。将这两个值相乘，可以得到需要进行调整的距离向量，将其作为参数传入 playerCollider 的 translate 方法中，就可以将玩家位置进行调整，避免穿模现象的发生。


# Threejs实现鼠标点击人物行走/镜头跟随人物移动/鼠标点击动画/游戏第三人称/行走动作
https://zuoben.blog.csdn.net/article/details/128498888

1，功能介绍
Threejs获取鼠标点击位置、实现鼠标点击人物行走、人物头顶显示名称标签、镜头跟随人物移动并且镜头围绕人物旋转，类似游戏中第三人称、鼠标点击位置有动画效果，如下效果图

![](https://img-blog.csdnimg.cn/2878bcb8f0f3455fbc722cdee0f423fe.gif)

2，功能实现
获取鼠标点击位置，使用Threejs射线Raycaster和数学库中Plane平面，官方文档地址：three.js docs，平面（Plane）是在三维空间中无限延伸的二维平面，原理就是创建一个射线与平面相交，取相交位置的点就是鼠标点击的位置

```
// 拾取对象
function pickupObjects(event) {
  // 点击屏幕创建光线投射用于进行鼠标拾取
  var raycaster = new THREE.Raycaster();
 
  // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
  var vector = new THREE.Vector2((event.clientX / window.innerWidth) * 2 - 1, -(event.clientY / window
						.innerHeight) * 2 + 1);
 
  var fxl = new THREE.Vector3(0, 1, 0);
  // 创建平面 设置平面法向量Vector3 和 原点到平面距离
  var groundplane = new THREE.Plane(fxl, 1);
  raycaster.setFromCamera(vector, camera);
  var ray = raycaster.ray;
  let intersects = ray.intersectPlane(groundplane);
  console.log(intersects)
}
 
document.addEventListener('click', pickupObjects);
```
这里注意平面法向量Vector3(0, 1, 0) 代表平面的正方向是沿Y轴向上的，距离1代表平面到原点的距离，符号代表和法向量的方向相反，所以这个平面在Y轴正方向。

引入人物模型，使用模型加载器THREE.GLTFLoader加载人物模型，人物头顶显示名称标签并播放人物站立动画，注意播放动画需要requestAnimationFrame实时更新动画

```
// 加载人物模型，模型glb格式
let gloader = new THREE.GLTFLoader()
gloader.load("model/people.glb", result => {
	peopleObj = result.scene;
	peopleAnimations = result.animations;
 
	// 主人物名字，并设置相对位置
	let spriteText = getTextCanvas("左本博客");
	spriteText.position.set(0, 2, 0);
	peopleObj.add(spriteText);
 
	// 组合对象添加到场景中
	scene.add(peopleObj);
 
	// 默认播放人物站立动画
	mixer = new THREE.AnimationMixer(peopleObj);
	mixerArr.push(mixer)
	activeAction = mixer.clipAction(peopleAnimations[0]);
	activeAction.play();
})
```
人物名称标签，创建Canvas写入文本，使用THREE.CanvasTexture从Canvas元素中创建纹理贴图作为材质THREE.SpriteMaterial，创建一个物体精灵（Sprite）

```
// 创建文字精灵
var getTextCanvas = function(text) {
	let option = {
			fontFamily: 'Arial',
			fontSize: 30,
			fontWeight: 'bold',
			color: '#ffffff',
			actualFontSize: 0.08,
		},
		canvas, context, textWidth, texture, materialObj, spriteObj;
	canvas = document.createElement('canvas');
	context = canvas.getContext('2d');
	// 先设置字体大小后获取文本宽度
	context.font = option.fontWeight + ' ' + option.fontSize + 'px ' + option.fontFamily;
	textWidth = context.measureText(text).width;
 
	canvas.width = textWidth;
	canvas.height = option.fontSize;
 
	context.textAlign = "center";
	context.textBaseline = "middle";
	context.fillStyle = option.color;
	context.font = option.fontWeight + ' ' + option.fontSize + 'px ' + option.fontFamily;
	context.fillText(text, textWidth / 2, option.fontSize / 1.8);
 
	texture = new THREE.CanvasTexture(canvas);
	materialObj = new THREE.SpriteMaterial({
		map: texture
	});
	spriteObj = new THREE.Sprite(materialObj);
	spriteObj.scale.set(textWidth / option.fontSize * option.actualFontSize, option.actualFontSize, option
		.actualFontSize);
 
	return spriteObj;
}
```
鼠标点击位置动画，使用图片加载器THREE.ImageLoader加载如下透明图片，创建canvas作为THREE.MeshBasicMaterial材质贴图并设置透明和两面渲染，设置定时顺序更新材质

![Alt text](https://img-blog.csdnimg.cn/fd681e3cf0174687865f5a77f01b09fb.png)


// 鼠标点击位置出现动画效果
function clickEffect() {
	var textures = [];
	var index = 0;
	var clickEffectMesh;
	
	// 加载图片
	var loader = new THREE.ImageLoader();
	loader.load("img/spotd.png", function(image) {
		let canvas, context, geometry, material, plane;
		let w = image.width;
		let num = image.height / image.width;
 
		for (let i = 0; i < num; i++) {
			textures[i] = new THREE.Texture();
		}
 
		for (let i = 0; i < num; i++) {
			canvas = document.createElement('canvas');
			context = canvas.getContext('2d');
			canvas.height = w;
			canvas.width = w;
			context.drawImage(image, 0, w * i, w, w, 0, 0, w, w);
			textures[i].image = canvas;
			textures[i].needsUpdate = true;
		}
		
		// 创建平面几何体，添加图片材质并设置透明和两面渲染
		geometry = new THREE.PlaneGeometry(1, 1);
		material = new THREE.MeshBasicMaterial({
			map: textures[0],
			transparent: true,
			side: THREE.DoubleSide,
		});
 
		clickEffectMesh = new THREE.Mesh(geometry, material);
		clickEffectMesh.rotateX(Math.PI / 2);
		clickEffectMesh.visible = false;
 
		scene.add(clickEffectMesh);
	});
	
	// 播放动画，设置定时顺序更新材质
	this.effect = function(x, y, z) {
		clickEffectMesh.visible = true;
		clickEffectMesh.position.set(x, y, z);
		let id = setInterval(function() {
			if (index == 10) {
				index = 0;
				clickEffectMesh.visible = false;
				clearInterval(id);
			}
			clickEffectMesh.material.map = textures[index];
			index++;
		}, 20)
	}
}
 鼠标点击人物行走，使用TWEEN.Tween实现人物行走并播放人物行走动画，设置人物朝向行走方向，更新控制器目标中心位置及相机与人物保持相对距离
 
```
// 人物当前位置
let originPositon = new THREE.Vector3(peopleObj.position.x, 0, peopleObj.position.z);
// 任务目标位置
let targetPositon = new THREE.Vector3(intersects.x, 0, intersects.z);
clickEffectObj.effect(intersects.x, 0, intersects.z);
// 移动距离
let distance = originPositon.distanceTo(targetPositon);
 
peopleObj.lookAt(intersects.x, 0, intersects.z);
 
// 在传入的时间间隔内，逐渐将此动作的权重（weight）由1降至0
// 判断当前人物是否空闲状态，如果空前则改成步行状态
if (activeAction.getClip() != peopleAnimations[1]) {
	activeAction.fadeOut(0.2);
	activeAction = mixer.clipAction(peopleAnimations[1]);
	activeAction
		.reset()
		.setEffectiveTimeScale(1)
		.setEffectiveWeight(2)
		.fadeIn(0.2)
		.play();
}
 
if (tween) {
	TWEEN.remove(tween);
}
tween = new TWEEN.Tween(originPositon)
	.to(targetPositon, distance * 800)
	.easing(TWEEN.Easing.Linear.None)
	.onComplete(function() {
		activeAction.fadeOut(0.2);
		activeAction = mixer.clipAction(peopleAnimations[0]);
		activeAction
			.reset()
			.setEffectiveTimeScale(1)
			.setEffectiveWeight(2)
			.fadeIn(0.2)
			.play();
	})
	.onUpdate(function() {
		peopleObj.position.set(this.x, 0.04, this.z);
 
		let pos = peopleObj.position.clone();
		pos.y = 1;
		controls.target = pos;
		controls.update();
	})
	.start();
```