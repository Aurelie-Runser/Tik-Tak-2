<template>
    <div class="canvas_container" ref="canvasContainer">
        <canvas ref="canvas"/>
    </div>
</template>

<style>
.canvas_container{
    width: 100%;
}
</style>
  
<script setup>
import { ref, toRefs, onMounted, onUpdated, onBeforeUnmount } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ColladaLoader } from "three/examples/jsm/loaders/ColladaLoader.js";
import * as THREE from "three";
import { TextureLoader } from "three/src/loaders/TextureLoader.js";

const props = defineProps({
    boitier_texture : String,
    boitier_forme : String,
    bracelet_texture : String,
    pierre_couleur : String,
    main_color : String,
    fond_nom : String
})

const proprietes = toRefs(props)
  
const canvasContainer = ref(null);
const canvas = ref(null);
let controls, scene, camera, renderer, animationId = null;
let clock = new THREE.Clock();
let aiguilleHeures,
    aiguilleMinutes,
    aiguilleSecondes,
    boitierForme,
    iPierre,
    iBracelet,
    iFermoir,
    iBouton;
  
const initScene = () => {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(
        75,
        16 / 9,
        0.1,
        1000
    );
    const width = canvasContainer.value.clientWidth;
    const height = canvasContainer.value.clientWidth/2;
  
    renderer = new THREE.WebGLRenderer({ canvas: canvas.value });
    // renderer.setSize(1300, 700);
    renderer.setSize(width, height);
    renderer.setClearColor(0x222222, 1);
    controls = new OrbitControls(camera, renderer.domElement);

    const textureLoaderScene = new THREE.TextureLoader();
    scene.background = textureLoaderScene.load(`/images/fond_${proprietes.fond_nom.value}.jpg`);
  
    var loader = new ColladaLoader();
    loader.load("/models/montre.dae", onLoaded, onProgress, onError);
};
  
const updateClockHands = () => {
    const now = new Date();
    const hours = now.getHours() % 12;
    const minutes = now.getMinutes();
    const seconds = now.getSeconds();
  
    const hoursRotation = (hours + minutes / 60) * (Math.PI / 6);
    const minutesRotation = (minutes + seconds / 60) * (Math.PI / 30);
    const secondsRotation = seconds * (Math.PI / 30);
  
    if (aiguilleHeures) aiguilleHeures.rotation.z = -hoursRotation;
    if (aiguilleMinutes) aiguilleMinutes.rotation.z = -minutesRotation;
    if (aiguilleSecondes) aiguilleSecondes.rotation.z = -secondsRotation;
};
  
const animate = () => {
    let dt = clock.getDelta();
    updateClockHands();
    animationId = requestAnimationFrame(animate);
    renderer.render(scene, camera);
};
  
function onLoaded(collada) {
    let objects = collada.scene;
  
    aiguilleHeures = objects.getObjectByName("aiguille_heures");
    aiguilleHeures.material = new THREE.MeshBasicMaterial({
        color: proprietes.main_color.value,
    });

    aiguilleMinutes = objects.getObjectByName("aiguille_minutes");
    aiguilleMinutes.material = new THREE.MeshBasicMaterial({
        color: proprietes.main_color.value,
    });

    aiguilleSecondes = objects.getObjectByName("aiguille_secondes");
    aiguilleSecondes.material = new THREE.MeshBasicMaterial({
        color: proprietes.main_color.value,
    });
  
    if (proprietes.boitier_forme.value == "boitier_rond"){
        boitierForme = objects.getObjectByName("boitier_rond");
    } else if (proprietes.boitier_forme.value == "boitier_carre"){
        boitierForme= objects.getObjectByName("boitier_carre");
    }
    
    const textureLoaderBoitier = new TextureLoader();
    const textureBoitier = textureLoaderBoitier.load(`/images/background_${proprietes.boitier_texture.value}.png`);
    
    boitierForme.material[0] = new THREE.MeshBasicMaterial({
        color: proprietes.main_color.value,
    });

    boitierForme.material[1] = new THREE.MeshBasicMaterial({
        map: textureBoitier,
    });
  
    iBouton = objects.getObjectByName("bouton");
    iBouton.material = new THREE.MeshBasicMaterial({
        color: "#000",
    });
  
    iPierre = objects.getObjectByName("pierre");
    
    iPierre.material = new THREE.MeshBasicMaterial({
        color: proprietes.pierre_couleur.value,
    });
    
    if (proprietes.pierre_couleur.value == "none"){
        iPierre.material.opacity =  0,
        iPierre.material.transparent = true
    };
  
    let iPierre2 = iPierre.clone();
    iPierre2.position.y -= 38;
  
    let iPierre3 = iPierre.clone();
    iPierre3.position.x -= 18.5;
    iPierre3.position.y -= 18.75;
  
    let iPierre4 = iPierre.clone();
    iPierre4.position.x += 18.5;
    iPierre4.position.y -= 18.75;
  
    iBracelet = objects.getObjectByName("bracelet");
    const textureLoader = new TextureLoader();
    const texture = textureLoader.load(`/images/texture-${proprietes.bracelet_texture.value}.jpg`);
    iBracelet.material = new THREE.MeshBasicMaterial({ map: texture });
  
    iFermoir = objects.getObjectByName("fermoir");
    iFermoir.material = new THREE.MeshBasicMaterial({
        color: proprietes.main_color.value,
    });
  
    scene.add(
        aiguilleHeures,
        aiguilleMinutes,
        aiguilleSecondes,
        boitierForme,
        iBouton,
        iPierre,
        iPierre2,
        iPierre3,
        iPierre4,
        iBracelet,
        iFermoir
    );
  
    controls.target.set(0, 0, 0);
    camera.position.set(0, 0, 100);
    controls.update();
}
  
var onProgress = function (data) {
    if (data.lengthComputable) {
        var percentComplete = (data.loaded / data.total) * 100;
        // console.log(Math.round(percentComplete, 2) + "% downloaded");
    }
};
  
var onError = function (data) {
    console.error(data);
};
  
onMounted(() => {
    initScene();
    animate();
});

onUpdated(() => {
    initScene();
})
  
onBeforeUnmount(() => {
    cancelAnimationFrame(animationId);
});

</script>