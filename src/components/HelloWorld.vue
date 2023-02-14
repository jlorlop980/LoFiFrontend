
<script>

export default {
  data() {
    return {
      playing: false,
      playlists:false,
    }
  },
  methods: {
    reproduce(){
      this.playing=!this.playing;
    },
    tooglePlaylists(){
      this.playlists=!this.playlists;
    }
  }
}
</script>



<template>
  <ul class="nav">
    <li>
      <h2 v-on:click="tooglePlaylists">playlist</h2>
    </li>

    <li>
      <h2>favourites</h2>
    </li>

    <li>
      <h1>LoFiPlayer</h1>
    </li>

    <li>
      <h2>account</h2>
    </li>

    <li>
      <h2>search</h2>
    </li>
  </ul>

  <div v-if="playlists" class="playlists">
    <div class="playlist">The Spirit Whithin</div>
    <div class="playlist">Between Worlds</div>
    <div class="playlist">Forest Kingdom</div>
    <div class="playlist">The Days I Passed</div>
  </div>

  <button v-on:click="reproduce">
    <img v-if="playing" src="../assets/icons/play.svg">
    <img v-else src="../assets/icons/pause.svg">
  </button>
</template>


<!--CSS-->
<style >
@import url("//fonts.googleapis.com/css?family=Marck+Script");


img{
  width: 5vw;
  height: 5vw;
}

button{
  background-color: #F2CC8F;
  border-radius: 100%;
  position: absolute;
  bottom: 50px;
  left: 50px;
}

ul {
  list-style-type: none;
}

.playlists{
  position:absolute;

  width: fit-content;
  height: fit-content;
  background: rgba(61, 64, 91, 0.6);
  border-radius: 30px;
  opacity: 0.75;
}

.playlist{
  width: 15vw;
  height: 5vh;
  background: rgba(61, 64, 91, 0.6);
  border-radius: 30vw;
  text-align: center;
  color: #F2CC8F;
}


.nav{
  font-family:'Marck Script';
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-around;
}
</style>
