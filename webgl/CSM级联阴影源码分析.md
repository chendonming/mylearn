# THREEJS级联阴影源码分析

## 入口

```js
{
  maxFar: params.far, // 阴影最大距离
  cascades: 4, // 层级
  mode: params.mode, // 模式
  parent: scene, // 场景
  shadowMapSize: 1024, // 阴影图大小
  lightDirection: new THREE.Vector3( params.lightX, params.lightY, params.lightZ ).normalize(), // 光的方向
  camera: camera
}
```

**mode** 切割的方法，存在三种
- uniform
- logarithmic
- practical<默认值>

三种模式的切割方法如下:

### uniform
```js
function uniformSplit( amount, near, far, target ) {

  for ( let i = 1; i < amount; i ++ ) {

    target.push( ( near + ( far - near ) * i / amount ) / far );

  }

  target.push( 1 );

}
```

表格如下结果:

https://www.desmos.com/calculator/qtpd4gyfnv?lang=zh-CN

可以看出就是一个线性渐变， 所以uniform切割就是平均切割

### logarithmic
```js
function logarithmicSplit( amount, near, far, target ) {

  for ( let i = 1; i < amount; i ++ ) {

    target.push( ( near * ( far / near ) ** ( i / amount ) ) / far );

  }

  target.push( 1 );

}
```

表格结果如下:

https://www.desmos.com/calculator/zf20krz3hd?lang=zh-CN

可以看出是一个对数函数, 在半数之前几乎没有变化，最后变化剧烈，一直到完全为1.

### practical
```js
function practicalSplit( amount, near, far, lambda, target ) {

  _uniformArray.length = 0;
  _logArray.length = 0;
  logarithmicSplit( amount, near, far, _logArray );
  uniformSplit( amount, near, far, _uniformArray );

  for ( let i = 1; i < amount; i ++ ) {

    target.push( MathUtils.lerp( _uniformArray[ i - 1 ], _logArray[ i - 1 ], lambda ) );

  }

  target.push( 1 );

}
```

使用了平均切割和对数切割 然后取插值， 这里的插值算法如下:
```js
function lerp( x, y, t ) {

	return ( 1 - t ) * x + t * y;

}
```

就是线性插值，所以practical是取前两者的线性插值切割

## 流程

- 创建平行光 this.createLights()
- 更新视锥体 this.updateFrustums()

### updateFrustums 更新视锥体

包含以下操作:

1.切割（uniform logarithmic practical）

只是确定切割方法，将进度值(类似[0, 0.1, 0.4, 1])存放在this.breaks中

2.初始化视锥体mainFrustum

存在一个新的类**CSMFrustum** 通过投影矩阵更新**CSMFrustum**
```js
const camera = this.camera;
camera.updateProjectionMatrix();
this.mainFrustum.setFromProjectionMatrix( camera.projectionMatrix, this.maxFar );
this.mainFrustum.split( this.breaks, this.frustums );
```

**setFromProjectionMatrix**用于获取近平面和远平面上的顶点坐标。

如何获取近平面和远平面的顶点坐标？以下是一个纯净的实现:
```js
setFromProjectionMatrix( projectionMatrix, maxFar ) {

  const isOrthographic = projectionMatrix.elements[ 2 * 4 + 3 ] === 0;

  inverseProjectionMatrix.copy( projectionMatrix ).invert();

  // 3 --- 0  vertices.near/far order
  // |     |
  // 2 --- 1
  // clip space spans from [-1, 1]

  this.vertices.near[ 0 ].set( 1, 1, - 1 );
  this.vertices.near[ 1 ].set( 1, - 1, - 1 );
  this.vertices.near[ 2 ].set( - 1, - 1, - 1 );
  this.vertices.near[ 3 ].set( - 1, 1, - 1 );
  this.vertices.near.forEach( function ( v ) {

    v.applyMatrix4( inverseProjectionMatrix );

  } );

  this.vertices.far[ 0 ].set( 1, 1, 1 );
  this.vertices.far[ 1 ].set( 1, - 1, 1 );
  this.vertices.far[ 2 ].set( - 1, - 1, 1 );
  this.vertices.far[ 3 ].set( - 1, 1, 1 );
  this.vertices.far.forEach( function ( v ) {

    v.applyMatrix4( inverseProjectionMatrix );

    const absZ = Math.abs( v.z );
    if ( isOrthographic ) {

      v.z *= Math.min( maxFar / absZ, 1.0 );

    } else {

      v.multiplyScalar( Math.min( maxFar / absZ, 1.0 ) );

    }

  } );

  return this.vertices;

}
```

**split**用于组装一个frustums数组，frustums数组和this.breaks的进度值保持一致

3.更新阴影边界updateShadowBounds

设置前面灯光的**shadow.camera**的left right top bottom
```js
shadowCam.left = - squaredBBWidth / 2;
shadowCam.right = squaredBBWidth / 2;
shadowCam.top = squaredBBWidth / 2;
shadowCam.bottom = - squaredBBWidth / 2;
```

**squaredBBWidth**的值由远近平面的对角线或者自身平面的宽度决定（ 取最大值为squaredBBWidth ）

4.更新材质updateUniforms

## update
这个是每一帧都需要调用的更新接口, 这个需要和shader一起理解才行，或者说shader才是理解的核心
