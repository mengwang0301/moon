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

        // let axes = new THREE.AxesHelper(100000)
        // this.scene.add(axes)

        let container = document.getElementById('container')
        let width = container.clientWidth// 窗口宽度
        let height = container.clientHeight// 窗口高度
        // 创建相机对象
        this.camera = new THREE.PerspectiveCamera(30, width / height, 1, 50000)
        this.camera.position.set(0, 0, 2000)// 设置相机位置
        this.camera.lookAt(new THREE.Vector3(size * column / 2, 0, size * row / 2))// 设置相机方向(指向的场景对象)

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
        // this.scene.add(land)
        for (let i = 0; i < row; i++) {
          for (let j = 0; j < column; j++) {
            let mesh = new THREE.Mesh(new THREE.PlaneBufferGeometry(size+1, size+1), groundMaterial)
            const height = data[column * i + j]
            mesh.position.x = (size * j)
            mesh.position.y = height - 2200
            mesh.position.z = (size * i)
            mesh.rotation.x = -Math.PI / 2
            // mesh.receiveShadow = true
            // console.log('x= '+ size*j+'   y = ' + size*i )
            this.scene.add(mesh)
          }
        }
      }
    }
  }
</script>
