
<script>
import axios from "axios";

export default {
  data() {
    return {
      playing: false,
      playlists:false,
      currentPlaylist:1,
      currentPlayingPlaylist:1,
      CPP:[],
      lists:[],
      showSongs:false,
      songs:[],
      newPlaylistName: "",
      showModal:false,
      cancion: new Audio(),
      currentPlayingSong:{},
      finalizada:false,
      currentTime:0,
      totalTime:0,
      likes:[],
      volume:0,
      hover:"oculto"
    }
  },

  methods: {
    openModal() {
    this.showModal = true;
  },
  closeModal() {
    this.showModal = false;
  },
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
      this.currentPlaylist=id;
      let canciones = this.lists.find(list=>list.id==id);
      this.showSongs=!this.showSongs;
      this.songs = canciones.songs;
    },
    changeSong(actual){
      console.log(actual.id)
      this.currentPlayingPlaylist=this.currentPlaylist;
      this.CPP=this.lists.find(list=>list.id==this.currentPlayingPlaylist).songs;
      this.currentPlayingSong=actual;
      this.cancion.pause();
      this.cancion = new Audio(`http://localhost:3001${actual.route}`);
      this.addEvents()
      this.playing=false;
      this.reproduce();
      this.cancion.addEventListener('ended', () => {
      console.log('Song has ended');
      this.nextSong()
      });
    },
    resetSong(){
      this.cancion.currentTime=0;
    },
    nextSong(){
      console.log("terminó");
      let indice= this.CPP.findIndex(elemento=>elemento.id==this.currentPlayingSong.id)
      if(indice+1==this.CPP.length){
        this.changeSong(this.CPP[0])
      }
      else{
        this.changeSong(this.CPP[indice+1])
      }
    },
    createPlaylist() {
      axios.post('http://localhost:3001/api/v1/playlists', {
        nombre: this.newPlaylistName,
        userId: 1, // Reemplaza esto con el ID de usuario apropiado
        songs: []
      })
      .then(response => {
        this.showModal = false;
        this.newPlaylistName = "";
        this.lists.push(response.data);
      })
      .catch(error => console.error(error));
    },
    likeSong(){
      axios.post('http://localhost:3001/api/v1/favs', {
        userId: 1, // Reemplaza esto con el ID de usuario apropiado
        cancion: this.currentPlayingSong
      })
      .then(response => {
        this.likes.push(this.currentPlayingSong)
      })
      .catch(error => console.error(error));
      console.log(this.currentPlayingSong)
    },
    addEvents(){

        this.cancion.addEventListener("timeupdate", () => {
        this.currentTime = this.cancion.currentTime;
      });
        this.cancion.addEventListener("loadedmetadata", () => {
        this.totalTime = this.cancion.duration;
      });
    },
    formatTime(time) {
      const minutes = Math.floor(time / 60);
      const seconds = Math.floor(time % 60);
      return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
    },
    updateCurrentTime(event) {
      this.cancion.currentTime = event.target.value;
    },
    updateVol(event){
      this.cancion.volume=this.volume;
      console.log(this.cancion.volume)
    }
  },
  
  mounted() {
    axios.get('http://localhost:3001/api/v1/playlists')
    .then(response => {
      this.lists = response.data.playlists; // assuming the response contains a "playlists" property
      this.songs=this.lists[0].songs;
      this.cancion = new Audio(`http://localhost:3001/${this.songs[0].route}`)
      this.CPP= this.songs;
      this.currentPlayingSong=this.songs[0];
      this.currentPlayingSongName=this.songs[0].name;
      this.cancion.addEventListener('ended', () => {
      console.log('Song has ended');
      this.nextSong()
      });
      this.currentTime=this.cancion.currentTime;
      this.volume=this.cancion.volume;
      console.log(this.cancion.duration)
      this.totalTime=this.cancion.duration;
      this.addEvents()
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


    <div class="add-playlist">
      <img class="addPl" src="../assets/icons/addWheat.svg" v-if="!showModal" v-on:click="openModal">
      <div v-if="showModal" class="modal">
        <div class="modal-content">
          <h2>Create a new playlist</h2>
          <input v-model="newPlaylistName" placeholder="Playlist name" type="text">
          <div class="modal-buttons">
            <button v-on:click="closeModal">Cancel</button>
            <button v-on:click="createPlaylist">Create</button>
          </div>
        </div>
      </div>
    </div>
  </div> 
    
  <div class="songs" v-if="showSongs" >
    <div class="song" v-for="song in songs" :key="song.id" ><p>{{ song.name }} - <span class="autor">{{ song.artist }}</span></p> <img class="songPlay" src="../assets/icons/playWheat.svg" v-on:click="changeSong(song)"></div>
    
  </div>

  <div class="player">
    <div class="player-data">
      <p class="data-name">{{ currentPlayingSong.name }}</p>
      <p class="data-artist">{{ currentPlayingSong.artist }}</p>
    </div>
    
    <div class="player-timeRelated">
      
      <input class="sliderTime" type="range" min="0" :max="totalTime" step="0.01" v-model="currentTime" @input="updateCurrentTime">
      <p>{{ formatTime(currentTime) }} / {{ formatTime(totalTime)}}</p>
      <div class="volControl">
        <input v-show="hover" class="volume" :class="hover" type="range" min="0" max="1" step="0.01" v-model="volume" @input="updateVol" @mouseenter="hover=true" @mouseleave="hover=false">
        <img v-if="volume==1" class="volume-icon" src="../assets/icons/maxVol.svg" @mouseenter="hover=true" @mouseleave="hover=false">
        <img v-if="volume>=0.5 && volume<1" class="volume-icon" src="../assets/icons/mediumVol.svg" @mouseenter="hover=true" @mouseleave="hover=false">
        <img v-if="volume<=0.5 && volume>0" class="volume-icon" src="../assets/icons/minVol.svg" @mouseenter="hover=true" @mouseleave="hover=false">
        <img v-if="volume==0" class="volume-icon" src="../assets/icons/mute.svg" @mouseenter="hover=true" @mouseleave="hover=false">
        
      </div>
      
    
      
    </div>
    <div class="playerControls">
      <img v-on:click="resetSong" class="playerControl-icon" src="../assets/icons/repeatWheat.svg">  
      <img v-on:click="showSongsP(currentPlayingPlaylist)" class="playerControl-icon" src="../assets/icons/CurrentPLWheat.svg">

      <p class="playerControls-text">previous</p>

      <button class="playbutton" v-on:click="reproduce">
      <img v-if="!playing" src="../assets/icons/play.svg">
      <img v-else src="../assets/icons/pause.svg">
      </button>

      <p class="playerControls-text" v-on:click="nextSong">next</p>

      <img v-on:click="likeSong" class="playerControl-icon" src="../assets/icons/likeWheat.svg">
      <img class="playerControl-icon" src="../assets/icons/addWheat.svg">
    </div>
  </div>

</template>


<!--CSS-->
<style >
@import url("../assets/css/index.css")
</style>
