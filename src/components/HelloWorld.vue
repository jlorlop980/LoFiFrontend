
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
      volume:0,
      hover:"oculto",
      userId:2,
      logged:true,
      showAdds:false,
      favorites:[],
      showFavs:false,
      emails:[],
      accountMenu:false
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
      this.showFavs=false
      this.playlists=!this.playlists;
      this.showSongs=false;

    },
    
    toogleFavs(){
      if(!this.logged){
        alert("primero tienes que hacer el login")
        return 1
      }
      this.showFavs=!this.showFavs
      this.playlists=false;
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
      this.cancion = new Audio(`https://lofibackend-production.up.railway.app${actual.route}`);
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
    
    previousSong(){
      let indice= this.CPP.findIndex(elemento=>elemento.id==this.currentPlayingSong.id)
      if(indice==0){
        this.changeSong(this.CPP[this.CPP.length-1])
      }
      else{
        this.changeSong(this.CPP[indice-1])
      }
    },

    createPlaylist() {
      axios.post('https://lofibackend-production.up.railway.app/api/v1/playlists', {
        nombre: this.newPlaylistName,
        userId: this.userId, // Reemplaza esto con el ID de usuario apropiado
        songs: []
      })
      .then(response => {
        this.showModal = false;
        this.newPlaylistName = "";
        this.lists.push(response.data);
      })
      .catch(error => console.error(error));
    },

    addSongToPlaylist(id){
      if(!this.logged){
        alert("primero tienes que hacer el login")
        return 1
      }
      axios.put(`https://lofibackend-production.up.railway.app/api/v1/playlists/nsong/${id}`, this.currentPlayingSong)
      .then(response => {
        console.log('Playlist updated successfully');
        const actList=response.data;
        const index=this.lists.findIndex(lista=>lista.id==actList.id);
        this.lists[index]=actList;
        alert(`añadida ${this.currentPlayingSong.name} a la playlist ${actList.name}`)
        this.showAdds=false;
      })
      .catch(error => {
        console.error('Error updating playlist', error);
      });
    },

    likeSong(){
      if(!this.logged){
        alert("primero tienes que hacer el login")
        return 1
      }
      
      axios.post('https://lofibackend-production.up.railway.app/api/v1/favs', {
        userId: this.userId, // Reemplaza esto con el ID de usuario apropiado
        cancion: this.currentPlayingSong
      })
      .then(response => {
        this.favorites.push(response.data)
      })
      .catch(error => console.error(error));
      console.log(this.currentPlayingSong)
    },
    
    removeLike(){
      if(!this.logged){
        alert("primero tienes que hacer el login")
        return 1
      }

      let index=this.favorites.findIndex(fav=>fav.song.id==this.currentPlayingSong.id)
      console.log("index ",index, "this.favorites[index].id ",this.favorites[index].id,"la cancion es:",this.favorites[index].song)
      axios.delete(`https://lofibackend-production.up.railway.app/api/v1/favs/${this.favorites[index].id}`)
      .then(response => {
      let removed=response.data[0].id;
      let index=this.favorites.findIndex(fav=>fav.id==removed);
      this.favorites.splice(index,1);
      this.showSongs=false;
    })
    .catch(error => {
      console.error('Error deleting playlist', error);
      });
    },

    getUserPlaylists(){
      axios.get(`https://lofibackend-production.up.railway.app/api/v1/playlists/userP/${this.userId}`) //pedimos las playlist del usuario 1 que se acaba de logear y estas son las playlist que se podran alterar
        .then(response => {
          this.lists=this.lists.concat(response.data); // assuming the response contains a "playlists" property
      })
        .catch(error => console.error(error));
    },

    deletePlaylist(id){
      axios.delete(`https://lofibackend-production.up.railway.app/api/v1/playlists/${id}`)
      .then(response => {
      let removed=response.data
      console.log('Playlist deleted successfully');
      let index=this.lists.findIndex(lista=>lista.id==removed.id);
      this.lists.splice(index,1);
      this.showSongs=false;
    })
    .catch(error => {
      console.error('Error deleting playlist', error);
      });
    },

    mostrarAdds(){
      if(!this.logged){
        alert("primero tienes que hacer el login")
        return 1
      }
      this.showAdds=!this.showAdds;
    },

    getUserFavs(){
      axios.get(`https://lofibackend-production.up.railway.app/api/v1/favs/userF/${this.userId}`) //pedimos las playlist del usuario 1 que se acaba de logear y estas son las playlist que se podran alterar
        .then(response => {
          this.favorites=response.data; // assuming the response contains a "playlists" property
      })
        .catch(error => console.error(error));
    },
    checkFavorite(){
      let index=this.favorites.findIndex(fav=>fav.song.id==this.currentPlayingSong.id);
      return index!=-1
    },
    
    register(){

    },
    
    postUser(user){
      //TODO
    },
    
    login(){
      this.getUserPlaylists();
      this.getUserFavs()

    },

    addEvents(){

        this.cancion.addEventListener("timeupdate", () => {
        this.currentTime = this.cancion.currentTime;
      });
        this.cancion.addEventListener("loadedmetadata", () => {
        this.totalTime = this.cancion.duration;
      });
    },
    getMails(){
      axios.get(`https://lofibackend-production.up.railway.app/api/v1/users/checkM`) //pedimos las playlist del usuario 1 que se acaba de logear y estas son las playlist que se podran alterar
        .then(response => {
          this.emails=response.data.emails; // assuming the response contains a "playlists" property
      })
        .catch(error => console.error(error));
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
    axios.get('https://lofibackend-production.up.railway.app/api/v1/playlists/userP/1') //pedimos las playlist del usuario 1 que son las del admin, es decir, las predeterminadas.
    .then(response => {
      this.lists = response.data; // assuming the response contains a "playlists" property
      this.songs=this.lists[0].songs;
      this.cancion = new Audio(`https://lofibackend-production.up.railway.app/${this.songs[0].route}`)
      this.CPP= this.songs;
      this.currentPlayingSong=this.songs[0];
      this.currentPlayingSongName=this.songs[0].name;
      this.cancion.addEventListener('ended', () => {
      console.log('Song has ended');
      this.nextSong()
      });
      this.currentTime=this.cancion.currentTime;
      this.volume=this.cancion.volume;
      this.totalTime=this.cancion.duration;
      this.addEvents();
      this.getMails();
      this.login()
    })
    .catch(error => console.error(error));

  }
}



</script>



<template>
  <ul class="nav">
    <li>
      <h2 class="cPointer" v-on:click="tooglePlaylists">playlist</h2>
    </li>

    <li>
      <h2 class="cPointer" v-on:click="toogleFavs">favorites</h2>
    </li>

    <li>
      <h1>LoFiPlayer</h1>
    </li>

    <li>
      <h2 class="cPointer">account</h2>
    </li>

    <li>
      <h2 class="blotted">search</h2>
    </li>
  </ul>

  <div v-if="playlists" class="playlists" >
    <div class="playlist" v-for="list in lists" :key="list.id" v-on:click="showSongsP(list.id)">
      <p>{{list.name}}</p>
      <img v-if="list.userId==userId" class="cruz" src="../assets/icons/Cross.svg" alt="cruz" v-on:click="deletePlaylist(list.id)">
    </div>


      <img class="addPl" src="../assets/icons/addWheat.svg" v-show="!showModal && logged" v-on:click="openModal">
      <div v-show="showModal" class="modal">
        <div class="modal-content">
          <h2 class="titulo">Create a new playlist</h2>
          <input class="entrada" v-model="newPlaylistName" placeholder="Playlist name" type="text">
          <div class="modal-buttons">
            <button class="botones cancelar" v-on:click="closeModal">Cancel</button>
            <button class="botones crearPL" v-on:click="createPlaylist">Create</button>
          </div>
        </div>
      </div>


  </div> 

  <div class="favs" v-if="showFavs" >
    <div class="fav" v-for="fav in favorites" :key="fav.id" >
      <p>{{ fav.song.name }} - <span class="autor">{{ fav.song.artist }}</span></p> <img class="songPlay" src="../assets/icons/playWheat.svg" v-on:click="changeSong(fav.song)">
    </div>  
  </div>
  
  <div class="songs" v-if="showSongs" >
    <div class="song" v-for="song in songs" :key="song.id" >
      <p>{{ song.name }} - <span class="autor">{{ song.artist }}</span></p> <img class="songPlay" src="../assets/icons/playWheat.svg" v-on:click="changeSong(song)">
    </div>  
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

      <div class="playerControl-bg">      
        <img v-on:click="resetSong" class="playerControl-icon" src="../assets/icons/repeatWheat.svg">  
      </div>

      <div class="playerControl-bg"> 
      <img v-on:click="showSongsP(currentPlayingPlaylist)" class="playerControl-icon" src="../assets/icons/CurrentPLWheat.svg">
      </div>

      <p class="playerControls-text" v-on:click="previousSong">previous</p>

      <button class="playbutton" v-on:click="reproduce">
      <img v-if="!playing" src="../assets/icons/play.svg">
      <img v-else src="../assets/icons/pause.svg">
      </button>

      <p class="playerControls-text" v-on:click="nextSong">next</p>

      <div class="playerControl-bg"> 
      <img v-if="!checkFavorite()" v-on:click="likeSong" class="playerControl-icon" src="../assets/icons/likeWheat.svg">
      <img v-if="checkFavorite()" v-on:click="removeLike" class="playerControl-icon" src="../assets/icons/liked.svg">
      </div>

      <div class="playerControl-bg"> 
      <img class="playerControl-icon" src="../assets/icons/addWheat.svg" v-on:click="mostrarAdds()">
      </div>
    </div>

  </div>
  <div v-if="showAdds" class="addingPlaylist">
  <div class="playlistToAdd" v-for="list in lists" :key="list.id" >
      <p :class="list.userId!=userId?'blotted':'null'">{{list.name}}</p>
      <img v-if="list.userId==userId" class="cruz" src="../assets/icons/plus.svg" alt="cruz" v-on:click="addSongToPlaylist(list.id)">
    </div>
  </div>
</template>


<!--CSS-->
<style >
@import url("../assets/css/index.css")
</style>
