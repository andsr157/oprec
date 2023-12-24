<script setup lang="ts">
import { useWindowSize } from "@vueuse/core"
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { useGeneralStore } from "@/stores/general"
import {
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  Color,
  Fog,
  Quaternion,
  Euler,
  AmbientLight,
  DirectionalLight,
} from "three"
const store = useGeneralStore()
const loadingDots = ref("")

function animateDots() {
  setInterval(() => {
    loadingDots.value =
      loadingDots.value === "..." ? "" : loadingDots.value + "."
  }, 500)
}

const experience = ref<HTMLCanvasElement | null>(null)
const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)

const bgColor = new Color("#161618")
const scene = new Scene()

scene.fog = new Fog(bgColor, 0.1, 75)
scene.background = bgColor

const camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(0, 0, 10)
scene.add(camera)

const gltfLoader = new GLTFLoader()
let object: any

onMounted(() => {
  animateDots()
  gltfLoader.load(
    `/oprec/himsi.gltf`,
    (gltf: any) => {
      gltf.scene.scale.set(0.3, 0.3, 0.3)
      gltf.scene.position.set(0, 0, 0)
      scene.add(gltf.scene)
      object = gltf.scene
      setRenderer()
      addLights()
      addControls()
      store.isLoading = false
      loop()
    },
    undefined,
    undefined,
    (error) => {
      console.error("Error loading himsi.gltf", error)
      store.isLoading = false // Set isLoading to false in case of an error
    }
  )
})

let renderer: WebGLRenderer
let controls: OrbitControls

function updateCamera() {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

function updateRenderer() {
  if (renderer) {
    renderer.setSize(width.value, height.value)
    renderer.render(scene, camera)
  }
}

function setRenderer() {
  if (experience.value) {
    renderer = new WebGLRenderer({ canvas: experience.value, alpha: true })
    experience.value.addEventListener("mouseenter", handleMouseEnter)
    experience.value.addEventListener("mouseleave", handleMouseLeave)
  }
}

function addLights() {
  const ambientLight = new AmbientLight(0x404040)
  scene.add(ambientLight)

  const directionalLight = new DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(1, 1, 1).normalize()
  scene.add(directionalLight)
}

function addControls() {
  if (experience.value && camera) {
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.dampingFactor = 0.25
    controls.screenSpacePanning = false
    controls.maxPolarAngle = Math.PI / 2
    controls.enableZoom = false // Disable zoom
  }
}

function loop() {
  if (object) {
    if (!object.userData.hovered) {
      const deltaRotationQuaternion = new Quaternion().setFromEuler(
        new Euler(0, 0.005, 0)
      )
      object.quaternion.multiplyQuaternions(
        deltaRotationQuaternion,
        object.quaternion
      )
    }
  }

  updateCamera()
  updateRenderer()
  controls.update()
  requestAnimationFrame(loop)
}

function handleMouseEnter() {
  if (object) {
    object.userData.hovered = false
  }
}

function handleMouseLeave() {
  if (object) {
    object.userData.hovered = true
  }
}
</script>

<template>
  <div class="w-full h-full">
    <div
      class="flex absolute w-full h-full justify-center items-center"
      v-if="store.isLoading"
    >
      <div
        class="grid grid-cols-4 justify-items-center text-white text-lg"
        style="align-items: center"
      >
        <div class="col-span-3 ml-10">Ready to Join With Us</div>
        <div class="loading-dots text-left w-10 -ml-6 text-lg">
          {{ loadingDots }}
        </div>
      </div>
    </div>
    <canvas ref="experience" class="w-full h-full" />
  </div>
</template>

<style scoped></style>
