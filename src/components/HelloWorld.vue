
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
      currentPlayingSong:0
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
    changeSong(name,id){
      console.log(id)
      this.currentPlayingPlaylist=this.currentPlaylist;
      this.CPP=this.lists.find(list=>list.id==this.currentPlayingPlaylist).songs;
      this.currentPlayingSong=id;
      this.cancion.pause();
      this.cancion = new Audio(`http://localhost:3001${name}`);
      this.playing=false;
      this.reproduce();
      
    },
    resetSong(){
      this.cancion.currentTime=0;
    },
    nextSong(){
      let indice= this.CPP.findIndex(elemento=>elemento.id==this.currentPlayingSong)
      if(indice+1==this.CPP.length){
        this.changeSong(this.CPP[0].route,this.CPP[0].id)
      }
      else{
        this.changeSong(this.CPP[indice+1].route,this.CPP[indice+1].id)
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
    }
  },
  mounted() {
    axios.get('http://localhost:3001/api/v1/playlists')
    .then(response => {
      this.lists = response.data.playlists; // assuming the response contains a "playlists" property
      this.songs=this.lists[0].songs;
      this.cancion = new Audio(`http://localhost:3001/${this.songs[0].route}`)
      this.CPP= this.songs;
      this.currentPlayingSong=this.songs[0].id;
    })
    .catch(error => console.error(error));

    

    //cuando acabé al canción
    this.cancion.addEventListener('ended', () => {
    console.log('Song has ended');
    // Add your logic here to move to the next song or stop playing
  });
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
    <div class="song" v-for="song in songs" :key="song.id" ><p>{{ song.name }}</p> <img class="songPlay" src="../assets/icons/playWheat.svg" v-on:click="changeSong(song.route,song.id)"></div>
    
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

    <img class="playerControl-icon" src="../assets/icons/likeWheat.svg">
    <img class="playerControl-icon" src="../assets/icons/addWheat.svg">
  </div>

</template>


<!--CSS-->
<style >
@import url("//fonts.googleapis.com/css?family=Marck+Script");


img{
  width: 5vw;
  height: 5vw;
}

.playbutton{
  background-color: #F2CC8F;
  border-radius: 100%;
  cursor: pointer;
  border-color: #0000;
}

.playerControl-icon{
  max-width: 2rem;
  max-height: 2rem;
  cursor: pointer;
}

.playerControls-text{
  font-family:'Marck Script';
  color:#F4F1DE;
  font-size: 1.5rem;
  text-shadow: 2px 2px 4px #000000;
}

ul {
  list-style-type: none;
}

.addPl{
  width: 2.5rem;
  height: 2.5rem;
}

.playlists{
  display: flex;
  flex-flow: column nowrap;
  gap:1vw;
  position:absolute;
  padding: 1vw;
  width: fit-content;
  height: fit-content;
  background: rgba(61, 64, 91, 0.7);
  border-radius: 30px;
  justify-content: center;
  align-items: center;
}

.playlist{
  width: 15vw;
  height: 5vh;
  background: rgba(61, 64, 91, 0.9);
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

.playerControls{
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-evenly;
  align-items: center;
  position: absolute;
  bottom: 10vh;
  gap: 5vw;
  left: 20vw;
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

.modal-buttons{
  position: absolute;
  top:50vh;
  display:flex;
  justify-content: center;
  flex-flow:row nowrap;
}

</style>
