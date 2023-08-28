<template>
    <div class="main">
      <div class="title">
        <h1>Meteo dal mondo</h1>
        <p>Clicca in un punto nella mappa per scoprire il meteo nei prossimi giorni</p>
      </div>
      <map-view></map-view>
    </div>
</template>
<script>
import MapView from '../components/MapView.vue'

export default {
  name: "Home",
  components: {
    MapView,
  },
};
</script>

<style lang="scss">
.main {
  .title{
    position: absolute;
    right: 0;
    width: 80%;
    height: 100px;
    z-index: 3;
    background-color: rgba(0, 0, 0, 0.3);
    color: white;
    h1 {
      margin: 10px 0;
    }
    p {
      font-size: 20px;
    }
  }
}
</style>