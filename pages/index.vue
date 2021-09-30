<template>
  <div class="container">
    <div class="row">
      <div class="col-lg-12">
        <h1>Scheduler</h1>
      </div>
    </div>
    <div class="row">
      <div class="col-lg-12 control-section">
        <Scheduler></Scheduler>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component } from 'vue-property-decorator'

@Component
export default class Main extends Vue {
}
</script>
