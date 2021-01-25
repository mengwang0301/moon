<template>
  <div class="about">
    <div id="container" style="width: 100%; height: 100%"></div>
  </div>
</template>

<script>
  import * as THREE from 'three'
  import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
  import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader'
  import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader'
  import {DRACOLoader} from 'three/examples/jsm/loaders/DRACOLoader'

  export default {

    data() {
      return {
        earthworm: null,
        xOffset: 0,
        zOffset: 0,
        earthwormPosArr: []
      }
    },
    mounted() {
      this.initScene()
      this.initInput()
      this.render()
    },

    methods: {
      initScene() {
        this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 100000);
        this.camera.position.set(280, 130, 0);		//相机位置
        this.scene = new THREE.Scene();

        let axes = new THREE.AxesHelper(100000)
        this.scene.add(axes)

        this.renderer = new THREE.WebGLRenderer({antialias: true, alpha: true})
        this.renderer.setSize(window.innerWidth, window.innerHeight)
        this.renderer.inputEncoding = true;
        this.renderer.outputEncoding = THREE.sRGBEncoding;
        document.getElementById('container').appendChild(this.renderer.domElement)// body元素中插入canvas对象
        // 执行渲染操作
        this.renderer.render(this.scene, this.camera)
        this.controls = new OrbitControls(this.camera, document.getElementById('container'))// 创建控件对象

        let textureCube = new RGBELoader().setDataType(THREE.UnsignedByteType);
        textureCube.load(`${process.env.BASE_URL}textures/015.hdr`, texture => {
          let pmremGenerator = new THREE.PMREMGenerator(this.renderer);
          pmremGenerator.compileEquirectangularShader();
          let cubeRenderTarget = pmremGenerator.fromEquirectangular(texture);
          textureCube = cubeRenderTarget.texture;
          texture.dispose();
        });

        // this.controls = new OrbitControls(this.camera);
        this.controls.autoRotate = false;
        this.controls.target.set(0, 80, 0)

        window.addEventListener('resize', this.onWindowResize, false);


        let loader = new GLTFLoader();
        let dracoLoader = new DRACOLoader().setDecoderPath(`${process.env.BASE_URL}js/gltf/`);
        loader.setDRACOLoader(dracoLoader);

        // let thismodel, mains = [];
        let thismodel1 = [];
        // let earthworm

        loader.load(`${process.env.BASE_URL}models/jiantou.gltf`, gltf => {
          // let children = []
          // const group = new THREE.Group()
          // for (let i = 0; i < 5; i++) {
          //   const child = gltf.scene.children[0];
          //   child.position.set((5-i)*10, 0, 0)
          //   this.earthwormPosArr.push(child.position)
          //   // children.push(child)
          //   group.add(child)
          // }
          this.earthworm = gltf.scene;
          // this.earthworm = gltf.scene.children[0];

          // group.children = children
          // console.log(group)
          // this.earthworm = group
          // console.log(gltf.scene)

          this.earthworm.scale.set(1000, 1000, 1000);
          this.earthworm.rotation.set(Math.PI, 0, 0)
          // this.earthworm.position.set(15, 0, 0);





          this.earthworm.traverse(child => {
            // this.earthwormPosArr.push(child.position)
            if (child.isMesh) {
              child.material.envMap = textureCube;
              child.material.envMapIntensity = 2;
              child.material.side = THREE.DoubleSide;
            }
          });
          // console.log(this.earthworm)
          this.scene.add(this.earthworm);
        }, undefined, err=> {
          console.error(err)
        });



        loader.load(`${process.env.BASE_URL}models/gltf/fangzi.gltf`, gltf => {
          thismodel1 = gltf.scene;

          thismodel1.scale.set(100, 100, 100);
          thismodel1.rotation.set(0, 0, 0)
          thismodel1.position.set(0, 0, 0);
          thismodel1.traverse(child => {
            if (child.isMesh) {
              child.material.lightMap = child.material.aoMap;
              child.material.lightMapIntensity = 1;
              child.material.aoMapIntensity = 0.3;
              child.material.envMap = textureCube;
              child.material.envMapIntensity = 2;
              child.material.side = THREE.DoubleSide;
            }
          });
          this.scene.add(thismodel1);
        });
      },

      render() {
        requestAnimationFrame(this.render);
        this.controls.update();
        this.renderer.clear();
        this.updateQiuyin()
        this.renderer.render(this.scene, this.camera);
      },

      initInput() {

        window.addEventListener('keydown', event => {

          switch (event.keyCode) {

            // A
            case 65:
              console.log('AAAAAAAAAAAAAAAAAA')
              this.xOffset = -1
              break;

            // S
            case 83:
              console.log('sssssssssssssssssssssss')
              this.zOffset = 1
              break;

            // d
            case 68:
              console.log('dddddddddddddddddd')
              this.xOffset = 1
              break;

            // w
            case 87:
              console.log('wwwwwwwwwwwwwwwwwwwwwwwww')
              this.zOffset = -1
              break;

          }

        }, false);

        window.addEventListener('keyup', ()=> {

          console.log('结束------')
          this.xOffset = 0
          this.zOffset = 0
        }, false);

      },

      updateQiuyin() {
        if (this.earthworm) {
          // this.earthworm.position.x += this.xOffset
          // this.earthworm.position.z += this.zOffset
          // console.log(this.xOffset)
          // this.earthworm.position.x += this.xOffset
          // this.earthworm.position.z += this.zOffset
          // // 先取出数组中第一个元素
          // if (this.xOffset ===0 && this.zOffset === 0) {
          //   return
          // }
          const pos = new THREE.Vector3(this.earthwormPosArr[0].x += this.xOffset/1000, this.earthwormPosArr[0].y, this.earthwormPosArr[0].z += this.zOffset/1000)

          this.earthwormPosArr.splice(0,0, pos)
          // console.log(this.earthwormPosArr.length)
          this.earthwormPosArr.splice(this.earthwormPosArr.length-1,1)
          // console.log(this.earthwormPosArr.length)
          // console.log('----------')
          // console.log(this.earthwormPosArr)
          this.earthwormPosArr.forEach((v,i)=>{
            const obj = this.earthworm.children[i]
            // obj.scale.set(1000, 1000, 1000);
            obj.position = v
          })
        }
      },

      onWindowResize(event) {
        this.camera.aspect = window.innerWidth / window.innerHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(window.innerWidth, window.innerHeight);
      }
    }
  }
</script>
