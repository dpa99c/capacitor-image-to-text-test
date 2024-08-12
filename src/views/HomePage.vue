<script setup lang="ts">
import {ref} from 'vue';
import {IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonIcon} from '@ionic/vue';
import {camera, image, refresh} from 'ionicons/icons';

import { Capacitor } from '@capacitor/core';
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';
import { Ocr, TextDetections } from '@capacitor-community/image-to-text';

const imagePath = ref<string>('');
const detectedText = ref<string>('');

const onPressTakePhoto = async () => {
  const photo = await Camera.getPhoto({
    quality: 90,
    allowEditing: false,
    resultType: CameraResultType.Uri,
    source: CameraSource.Camera,
  });

  imagePath.value = photo.path as string;
  await detectText();
}

const onPressPickImage = async () => {
  const image = await Camera.pickImages({
    limit: 1,
  })

  imagePath.value = image.photos[0].path as string;
  await detectText();
}

const detectText = async () => {
  detectedText.value = '';

  try{
    const data: TextDetections = await Ocr.detectText({ filename: imagePath.value });

    if(data.textDetections && data.textDetections.length > 0){
      console.dir(data.textDetections);
      for (const detection of data.textDetections) {
        detectedText.value += `${detection.text}<br>`;
      }
    }else{
      detectedText.value = 'No text detected';
    }
  }catch (e){
    detectedText.value = 'Error detecting text: ' + e.message;
  }
}

const onPressReset = () => {
  imagePath.value = '';
  detectedText.value = '';
}


</script>


<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Image-to-Text</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title>Image-to-Text</ion-title>
        </ion-toolbar>
      </ion-header>

      <div id="container">

        <ion-button v-if="!imagePath" @click="onPressTakePhoto">
          <span>Take Photo</span>
          <ion-icon slot="end" :icon="camera"></ion-icon>
        </ion-button>

        <ion-button v-if="!imagePath" @click="onPressPickImage">
          <span>Pick Image</span>
          <ion-icon slot="end" :icon="image"></ion-icon>
        </ion-button>

        <ion-button v-if="imagePath" @click="onPressReset">
          <span>Reset</span>
          <ion-icon slot="end" :icon="refresh"></ion-icon>
        </ion-button>

        <img v-if="imagePath" :src="Capacitor.convertFileSrc(imagePath)" />

        <div v-if="detectedText">
          <h2>Detected Text:</h2>
          <p v-html="detectedText"></p>
        </div>

      </div>
    </ion-content>
  </ion-page>
</template>


<style scoped>
#container {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 0.5rem;
}

ion-button{
  width: 50%;
  margin: 0.5rem auto;
}


img{
  margin-top: 20px;
  max-width: 100%;
  max-height: 300px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
</style>
