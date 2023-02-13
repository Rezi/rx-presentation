<template>
  <div class="relative h-full grid grid-cols-12 slidev-layout">
     <div class="col-span-7">
     
       
          <slot />
       
      </div> 
      <div class="col-span-5">
        <div class="pl-6">
        <slot name="right"/>
      </div>
      </div>
  </div>
</template>

<script setup lang="ts"></script>

