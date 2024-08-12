<template>
  <ion-app>
    <ion-router-outlet />
  </ion-app>
</template>

<script setup lang="ts">
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import { defineCustomElements } from '@ionic/pwa-elements/loader';

defineCustomElements(window);
</script>
