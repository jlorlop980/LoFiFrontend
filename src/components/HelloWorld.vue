
<script>

export default {
  data() {
    return {
      playing: false,
      playlists:false,
      lists:[],
      showSongs:false,
      songs:[],
      cancion: new Audio("http://localhost:3001/songs/theOtherSide_BVG.mp3")
    }
  },
  methods: {
    reproduce(){
      if(!this.playing){
        this.cancion.play();
      }else{this.cancion.pause();}
      this.playing=!this.playing;
    },
    tooglePlaylists(){
      this.playlists=!this.playlists;
      this.showSongs=false;

    },
    showSongsP(id){
      let canciones = this.lists.find(list=>list.id==id);
      this.showSongs=!this.showSongs;
      this.songs = canciones.songs;
    },
    changeSong(name){
      this.cancion = new Audio(`http://localhost:3001${name}`);
      this.cancion.play();
      this.playing=true;
    }
  },
  mounted() {
    fetch('http://localhost:3001/api/v1/playlists')
      .then(response => response.json())
      .then(data => {
        this.lists = data.playlists; // assuming the response contains a "playlists" property
      })
      .catch(error => console.error(error));
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

  <div v-if="playlists" class="playlists" >
    <div class="playlist" v-for="list in lists" :key="list.id" v-on:click="showSongsP(list.id)">{{list.name}}</div>
  </div> 
  <div class="songs" v-if="showSongs" >
    <div class="song" v-for="song in songs" :key="song.id" ><p>{{ song.name }}</p> <img class="songPlay" src="../assets/icons/playWheat.svg" v-on:click="changeSong(song.route)"></div>
  </div>

  <button v-on:click="reproduce">
    <img v-if="!playing" src="../assets/icons/play.svg">
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
  display: flex;
  flex-flow: column nowrap;
  gap:1vw;
  position:absolute;
  padding: 1vw;
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
  vertical-align: middle;
  line-height: 5vh;
  color: #F2CC8F;
}

.songs{
  position:absolute;
  left:30vw;
  width: fit-content;
  height: fit-content;
  background: rgba(61, 64, 91, 0.6);
  border-radius: 30px;
  opacity: 0.75;
}

.song{
  width: 15vw;
  height: 5vh;
  background: rgba(61, 64, 91, 0.6);
  border-radius: 30vw;
  color: #F2CC8F;
  display: flex;
  flex-flow: row nowrap;
  gap: 2vw;
  justify-content: center;
  align-items: center;
}

.nav{
  font-family:'Marck Script';
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-around;
}

.songPlay{
  width: 2vw;
  height: 2vw;
}

</style>
