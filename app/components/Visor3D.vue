<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const contenedor3D = ref(null)

let escena, camara, renderizador, controles, animacionId

onMounted(() => {
  // 1. ESCENA
  escena = new THREE.Scene()
  escena.background = new THREE.Color(0x1a1a2e) // Fondo oscuro más bonito para personaje

  // 2. CÁMARA
  camara = new THREE.PerspectiveCamera(
    45,
    contenedor3D.value.clientWidth / contenedor3D.value.clientHeight,
    0.1,
    1000
  )
  camara.position.set(0, 1.5, 4) // Posición buena para ver personaje

  // 3. RENDERIZADOR
  renderizador = new THREE.WebGLRenderer({ 
    antialias: true,
    alpha: false 
  })
  renderizador.setSize(contenedor3D.value.clientWidth, contenedor3D.value.clientHeight)
  renderizador.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderizador.shadowMap.enabled = true
  contenedor3D.value.appendChild(renderizador.domElement)

  // 4. LUCES (importante para que Yi se vea bien)
  const ambient = new THREE.AmbientLight(0xffffff, 0.7)
  escena.add(ambient)

  const directional = new THREE.DirectionalLight(0xffffff, 1.4)
  directional.position.set(5, 10, 7)
  directional.castShadow = true
  escena.add(directional)

  // Luz de relleno
  const fillLight = new THREE.DirectionalLight(0x88aaff, 0.6)
  fillLight.position.set(-5, 5, -7)
  escena.add(fillLight)

  // 5. CARGAR MODELO GLB (YI)
  const loader = new GLTFLoader()
  loader.load(
    '/models/yi.glb',
    (gltf) => {
      const modelo = gltf.scene
      
      // Ajustes recomendados (puedes modificar estos valores)
      modelo.scale.set(1, 1, 1)        // Ajusta tamaño
      modelo.position.set(0, -0.8, 0)        // Bajar un poco si está flotando
      modelo.rotation.y = Math.PI / 4        // Rotación inicial (opcional)

      escena.add(modelo)
      console.log('✅ Modelo Yi de Nine Sols cargado correctamente')
    },
    (progress) => {
      console.log(`Cargando: ${(progress.loaded / progress.total * 100).toFixed(1)}%`)
    },
    (error) => {
      console.error('❌ Error al cargar el modelo:', error)
    }
  )

  // 6. CONTROLES
  controles = new OrbitControls(camara, renderizador.domElement)
  controles.enableDamping = true
  controles.dampingFactor = 0.08
  controles.enableZoom = true
  controles.enablePan = true
  controles.minDistance = 1.5
  controles.maxDistance = 30

  // 7. ANIMACIÓN
  const animar = () => {
    animacionId = requestAnimationFrame(animar)
    controles.update()
    renderizador.render(escena, camara)
  }
  animar()

  // Resize
  const handleResize = () => {
    if (!camara || !renderizador || !contenedor3D.value) return
    camara.aspect = contenedor3D.value.clientWidth / contenedor3D.value.clientHeight
    camara.updateProjectionMatrix()
    renderizador.setSize(contenedor3D.value.clientWidth, contenedor3D.value.clientHeight)
  }
  window.addEventListener('resize', handleResize)
})

// Limpieza
onBeforeUnmount(() => {
  cancelAnimationFrame(animacionId)
  if (renderizador) {
    renderizador.dispose()
    renderizador.domElement.remove()
  }
})
</script>

<template>
  <div class="contenedor-visor">
    <div ref="contenedor3D" class="canvas-3d"></div>
  </div>
</template>

<style scoped>
.contenedor-visor {
  width: 100%;
  display: flex;
  justify-content: center;
  padding: 2rem 0;
}

.canvas-3d {
  width: 900px;
  height: 600px;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 15px 35px rgba(0,0,0,0.3);
}
</style>