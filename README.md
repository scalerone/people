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
``
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

 ```
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
