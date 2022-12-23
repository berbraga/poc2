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
          <video id="video"> </video>
        </div>
      </div>
    </div>
  </div>
  <div v-else style="display:none;">
    <div class="outside">
      <div class="player">
        <!-- player do video -->
        <div class="professional">
          <!-- player DA CAMERA do PROFISSIONAL -->
          <video id="video"> </video>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {Url} from '../Url.js'
import AgoraRTC from "agora-rtc-sdk";
import {T} from '../Functions.js';

export default {
  components:{
    Url,
  },
  data(){
    return{
      open: false,
      appId: 'b5cd1cffff564ab5a4dbf08d3e7d59b1',
      appKey:'007eJxTYHB5cOLCybbcvtTTk1MF/l2VvL48S/7JqrW/ove6fromEvpQgSHJNDnFMDkNCEzNTBKTTBNNUpLSDCxSjFPNU0wtkwyPL1ua3BDIyKB9JZuVkQECQXwWhoD8ZCMGBgChnyPc',
      channelName: 'Poc2',
// const AgoraRTC = require('agora-rtc-sdk'); const appId = "YOUR_APP_ID"; const appKey = "YOUR_APP_KEY";


      localStreams: {
        uid: '',
        camera: {
          camId: '',
          midId: '',
          stream: {},
        }
      },

      devices: {
        cameras: [],
        mic: []
      },

      externalBroadcastUrl: '',
      injectedStreamURL: '',

      defaultConfigRTMP: {
      width: 640,
      height: 360,
      videoBitrate: 400,
      videoFramerate: 15,
      lowLatency: false,
      audioSampleRate: 48000,
      audioBitrate: 48,
      audioChannels: 1,
      videoGop: 30,
      videoCodecProfile: 100,
      userCount: 0,
      userConfigExtraInfo: {},
      backgroundColor: 0x000000,
      transcodingUsers: [],
      },

    }
  },

  methods: {

    enterProfessional() {
      this.open= !this.open
    navigator.mediaDevices.getUserMedia({video: true})
    .then(function (mediaStream) {
      const video = document.querySelector('#video');
      video.srcObject = mediaStream;
      video.play();
    })
    .catch(function (err) {
      console.log('Não há permissões para acessar a webcam')
    })
  },


    enterClient() {
      this.open= !this.open

      const client = AgoraRTC.createClient({ mode: 'rtc', codec: 'vp8' });

      client.init(this.appId, () =>
        console.log('AgoraRTC client initialized'));


      client.join(this.appKey, this.channelName, null, (uid) => {
        console.log('================PASSOU ======================')
        var uidId = uid;
        console.log(`User ${uid} joined channel`);
      });
      console.log(uidId)
      const localStream = AgoraRTC.createStream({ streamID: uidId, audio: true, video: true, screen: false });
      console.log('========== ANTES LOCAL STREAM ==========');

      localStream.init(() => {
        console.log('Local stream initialized');
        console.log('==========entrou ==========');

        localStream.play('video');
        console.log('==========tyocando no video==========');
        client.publish(localStream, (err) =>
        console.log('Publish local stream error: ' + err));
      });

      client.on('stream-added', (evt) => {
        const stream = evt.stream;
        console.log(`Stream added:${evt}`)

      });

      // client.init(this.agoraAppId, function() {
      //   //console.log('===========1==============')
  		// 	var userID = null; // set to null to auto generate uid on successfull connection
      //   //console.log('===========2==============')

  		// 	client.setClientRole('host', function() {
      //  //console.log('===========3==============')
  		// 		console.log('Client role set as host.');
  		// 	}, function(e) {
      //   //console.log('===========4==============')
  		// 		console.log('setClientRole failed', e);
  		// 	});
      //   let channel = 'poc2'

  		// 	// this.localStreams.uid = uid; // keep track of the stream uid





      //   console.log('===========5==============')

  		// 	client.join(null,  channel, null, function(uid) {

      //   console.log('===========5.5==============')
  		// 			//createCameraStream(uid, {});
      //   console.log('===========6==============')
      //   console.log('===========7==============')
  		// 	console.log('User ' + uid + ' joined channel successfully');
  		// 	}, function(err) {
      //   console.log('=========== ERRO ==============')
  		// 			console.log('[ERROR] : join channel failed', err);
  		// 	});
      // }, function(err) {
      //   console.log('[ERROR] : AgoraRTC client init failed', err);
      // });

    },

    // NAO TA FUNCIONANDO A PARTE DE IR PARA ESSA FUNCAO!!

	  // joinChannel() {
		// 	var userID = null; // set to null to auto generate uid on successfull connection
		// 	console.log('teste Agora Agora2');
		// 	// set the role
		// 	client.setClientRole('host', function() {
		// 		console.log('Client role set as host.');
		// 	}, function(e) {
		// 		console.log('setClientRole failed', e);
		// 	});
		// 	// client.join('allThingsRTCLiveStream', 0, function(uid) {
		// 	client.join(channelName, userID, function(uid) {
		// 			createCameraStream(uid, {});
		// 			localStreams.uid = uid; // keep track of the stream uid
		// 	console.log('User ' + uid + ' joined channel successfully');
		// 	}, function(err) {
		// 			console.log('[ERROR] : join channel failed', err);
		// 	});
		// },

    // createCameraStream (uid, deviceIds) {
		// 	console.log('Creating stream with sources: ' + JSON.stringify(deviceIds));

    //   var localStream = AgoraRTC.createStream({
    //     streamID: uid,
    //     audio: true,
    //     video: true,
    //     screen: false
    //   });
    //   localStream.setVideoProfile(cameraVideoProfile);

    //   // The user has granted access to the camera and mic.
    //   localStream.on("accessAllowed", function() {
    //     if(devices.cameras.length === 0 && devices.mics.length === 0) {
    //       console.log('[DEBUG] : checking for cameras & mics');
    //       getCameraDevices();
    //       getMicDevices();
    //     }
    //     console.log("accessAllowed");
    //   });
    //   // The user has denied access to the camera and mic.
    //   localStream.on("accessDenied", function() {
    //     console.log("accessDenied");
    //   });

    //   localStream.init(function() {
    //     console.log('getUserMedia successfully');
    //     localStream.play('full-screen-video'); // play the local stream on the main div
    //     // publish local stream

    //     if($.isEmptyObject(localStreams.camera.stream)) {
    //       enableUiControls(localStream); // move after testing
    //     } else {
    //       //reset controls
    //       $("#mic-btn").prop("disabled", false);
    //       $("#video-btn").prop("disabled", false);
    //       $("#exit-btn").prop("disabled", false);
    //     }

    //     client.publish(localStream, function (err) {
    //       console.log('[ERROR] : publish local stream error: ' + err);
    //     });

    //     localStreams.camera.stream = localStream; // keep track of the camera stream for later
    //   }, function (err) {
		// 		console.log('[ERROR] : getUserMedia failed', err);
    //   });
	  // },


//     createCameraStream(uid, deviceIds) {
//     console.log('Creating stream with sources: ' + JSON.stringify(deviceIds));

//     var localStream = AgoraRTC.createStream({
//       streamID: uid,
//       audio: true,
//       video: true,
//       screen: false
//     });
//     localStream.setVideoProfile(cameraVideoProfile);

//     // The user has granted access to the camera and mic.
//     localStream.on("accessAllowed", function() {
//       if(devices.cameras.length === 0 && devices.mics.length === 0) {
// 				console.log('[DEBUG] : checking for cameras & mics');
//         getCameraDevices();
//         getMicDevices();
//       }
//       console.log("accessAllowed");
//     });
//     // The user has denied access to the camera and mic.
//     localStream.on("accessDenied", function() {
//       console.log("accessDenied");
//     });

//     localStream.init(function() {
//       console.log('getUserMedia successfully');
//       localStream.play('full-screen-video'); // play the local stream on the main div
//       // publish local stream

//       if($.isEmptyObject(localStreams.camera.stream)) {
//         enableUiControls(localStream); // move after testing
//       } else {
//         //reset controls
//         $("#mic-btn").prop("disabled", false);
//         $("#video-btn").prop("disabled", false);
//         $("#exit-btn").prop("disabled", false);
//       }

//       client.publish(localStream, function (err) {
//         console.log('[ERROR] : publish local stream error: ' + err);
//       });

//       localStreams.camera.stream = localStream; // keep track of the camera stream for later
//     }, function (err) {
//       console.log('[ERROR] : getUserMedia failed', err);
//     });
// 	}
},

}
</script>

<style scoped>
  .outside {
    display: flex;
    align-content: center;
    justify-content: center;
  }
  #video {
    width: 200px;
    height: 200px;
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
    /* border: 2px solid; */
    width: 200px;
    height: 250px;
    border-width: 2px;
    border-radius: 6px;
  }


</style>
