<template>
  <div>
    <h1 style='font-size: 3.2em; font-family: cursive;'><b>Agora.io</b></h1>
    <button @click="this.enterProfessional()" style='width: 200px; margin: 10px'>Entrar/Fechar Profissional</button>
    <button @click="this.enterClient()" style='width: 200px'>Entrar/Fechar Cliente</button>
  </div>
  
  <div v-if="this.open === true"> 
    <div class="outside"> 
      <div class="player">
        <!-- player do video -->
        <iframe width="1280" height="720" src="https://www.youtube.com/embed/KCc4TEbZCmc" title="O TRANSMISSOR de RÁDIO MAIS SIMPLES do MUNDO!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
        <div class="professional">
          <!-- player DA CAMERA do PROFISSIONAL -->
        </div>
      </div>
    </div>
  </div>
  <div v-else style="display:none;"> 
    <div class="outside"> 
      <div class="player">
        <!-- player do video -->
        <iframe width="1886" height="760" src="https://www.youtube.com/embed/KCc4TEbZCmc" title="O TRANSMISSOR de RÁDIO MAIS SIMPLES do MUNDO!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
        <div class="professional">
          <!-- player DA CAMERA do PROFISSIONAL -->
          <iframe width="200" height="200" src="https://www.youtube.com/embed/zIQ5gjErAyQ" title="Inclusão. Vídeo em Libras.Com legenda e áudio." frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {Url} from '../Url.js'
import AgoraRTC from "agora-rtc-sdk";

export default {
  components:{
    Url,
  },
  data(){
    return{
      open: false,    
    }
  },
  
  methods: {
    enterProfessional() {
      // fetch(Url)
      // .then()
      // .catch()
      
    },
    
    enterClient() {
      this.open= !this.open
    }
  },

}
</script>

<style scoped>
  .outside {
    display: flex;
    align-content: center;
    justify-content: center;
  }
  .player {
    border: 1px black solid;
    display: flex;
    align-content: center;
    justify-content: flex-end;
    align-items: flex-end;
    width: 1280px;
    height: 720px;
    border-width: 2px;
    border-radius: 4px;
  }
   .professional {
    position: absolute;
    border: 2px whitesmoke solid;
    width: 200px;
    height: 200px;
    border-width: 2px;
    border-radius: 6px;
  }


</style>
