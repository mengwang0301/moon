<template>
  <div class="home">
    <!--    <img alt="Vue logo" src="../assets/logo.png">-->
    <!--    <HelloWorld msg="Welcome to Your Vue.js App"/>-->
    <div id="container" style="width: 100%; height: 600px"></div>
  </div>
</template>

<script>
  // @ is an alias to /src
  import HelloWorld from '@/components/HelloWorld.vue'
  import data from '@/assets/data/map_1km.json'
  import * as THREE from 'three'
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'

  const size = 50
  const column = 50
  const row = 50
  export default {
    name: 'Home',

    components: {
      HelloWorld
    },

    mounted() {
      console.log(data)
      this.initSkyBox()
      this.drawLand()
      this.animateModel()
    },

    methods: {
      animateModel() {
        requestAnimationFrame(this.animateModel)
        this.renderer.render(this.scene, this.camera)
      },
      initSkyBox() {
        this.scene = new THREE.Scene()
        this.scene.background = new THREE.Color(0xcce0ff)


        let ambient = new THREE.AmbientLight(0xcccccc)
        this.scene.add(ambient)
        // let directionalLight = new THREE.DirectionalLight( 0xffffff, 0.5 );
        // this.scene.add( directionalLight );

        // let axes = new THREE.AxesHelper(100000)
        // this.scene.add(axes)

        let container = document.getElementById('container')
        let width = container.clientWidth// 窗口宽度
        let height = container.clientHeight// 窗口高度
        // 创建相机对象
        this.camera = new THREE.PerspectiveCamera(30, width / height, 1, 50000)
        this.camera.position.set(2000, 2000, 2000)// 设置相机位置
        this.camera.lookAt(0,0,0)// 设置相机方向(指向的场景对象)

        /**
         * 创建渲染器对象
         */
        this.renderer = new THREE.WebGLRenderer({antialias: true})
        this.renderer.setSize(width, height)
        this.renderer.setClearColor(0xb9d3ff, 1)// 设置背景颜色
        document.getElementById('container').appendChild(this.renderer.domElement)// body元素中插入canvas对象
        // 执行渲染操作
        this.renderer.render(this.scene, this.camera)
        this.controls = new OrbitControls(this.camera, document.getElementById('container'))// 创建控件对象
        // this.controls.maxPolarAngle = Math.PI * 0.5;
        // this.controls.minAzimuthAngle = THREE.MathUtils.degToRad(30)
        // this.controls.minDistance = 1000;
        // this.controls.maxDistance = 1.4 * sqWidth;
      },
      drawLand() {
        const groundMaterial = new THREE.MeshBasicMaterial({
          color: 0x666666,
        })
        const land = new THREE.Mesh(new THREE.PlaneBufferGeometry(size * column, size * row), groundMaterial)
        land.position.x = size * column / 2
        land.position.z = size * row / 2
        land.rotation.x = -Math.PI / 2

        let maxHeight = 0
        let minHeight = 10000
        data.forEach(v=>{
          maxHeight = Math.max(maxHeight, v)
          minHeight = Math.min(minHeight, v)
        })

        let vertices = []
        for (let i = 0; i < row; i++) {
          for (let j = 0; j < column; j++) {
            const height = data[column * i + j]
            const newHeight = (height - (maxHeight - minHeight) / 2) * 2 + (maxHeight-minHeight)
            vertices.push(new THREE.Vector3(size * j, newHeight, size * i))
          }
        }

        // 立方体
        let cubeGeometry = new THREE.Geometry()
        cubeGeometry.vertices = vertices

        let faces=[]
        for (let i = 0; i < row - 1; i++) {
          for (let j = 0; j < column - 1; j++) {
            let face1 = new THREE.Face3(row*i+j, row*(i+1)+j, row*i + j + 1)
            const y1 = vertices[face1.a].y
            const y2 = vertices[face1.b].y
            const y3 = vertices[face1.c].y
            face1.vertexColors = [
              new THREE.Color(y1 - Math.floor(y1), y1 - Math.floor(y1), y1 - Math.floor(y1)),
              new THREE.Color(y2 - Math.floor(y2), y2 - Math.floor(y2), y2 - Math.floor(y2)),
              new THREE.Color(y3 - Math.floor(y3), y3 - Math.floor(y3), y3 - Math.floor(y3)),
            ]

            let face2 = new THREE.Face3(row*i + j + 1, row*(i+1)+j, row*(i+1)+j+1)
            const y4 = vertices[face2.a].y
            const y5 = vertices[face2.b].y
            const y6 = vertices[face2.c].y
            face2.vertexColors = [
              new THREE.Color(y4 - Math.floor(y4), y4 - Math.floor(y4), y4 - Math.floor(y4)),
              new THREE.Color(y5 - Math.floor(y5), y5 - Math.floor(y5), y5 - Math.floor(y5)),
              new THREE.Color(y6 - Math.floor(y6), y6 - Math.floor(y6), y6 - Math.floor(y6)),
            ];
            faces.push(face1)
            faces.push(face2)
          }
        }
        cubeGeometry.faces = faces;
        const cube = new THREE.Mesh(cubeGeometry,new THREE.MeshBasicMaterial({
          vertexColors : THREE.VertexColors,
          side: THREE.DoubleSide
        }));
        this.scene.add(cube);
      }
    }
  }
</script>
