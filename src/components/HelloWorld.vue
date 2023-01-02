<template>
  <div>
    <h1 style='font-size: 3.2em; font-family: cursive;'><b>Agora.io</b></h1>
    <button @click="this.enterVideo()" style='width: 200px'>Criar um Canal</button>
    <button @click="this.joinChannel()" style='width: 200px'>Entrar no Canal</button>
    <button @click="this.leaveChannel()" style='width: 200px'>Sair do Canal</button>
  </div>

  <div v-if="this.open === true">
    <div class="outside">
      <div class="player">
        <!-- <iframe width="1280" height="720" src="https://www.youtube.com/embed/KCc4TEbZCmc" title="O TRANSMISSOR de RÁDIO MAIS SIMPLES do MUNDO!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->
        <div class="professional">
          <div id="video"></div>
        </div>
      </div>
    </div>
  </div>

</template>

<script>

import AgoraRTC from "agora-rtc-sdk";


export default {

  data(){
    return{
      open: false,
      appId: 'b5cd1cffff564ab5a4dbf08d3e7d59b1',
      appKey:'007eJxTYHB5cOLCybbcvtTTk1MF/l2VvL48S/7JqrW/ove6fromEvpQgSHJNDnFMDkNCEzNTBKTTBNNUpLSDCxSjFPNU0wtkwyPL1ua3BDIyKB9JZuVkQECQXwWhoD8ZCMGBgChnyPc',
      channelName: 'Poc2',
      client: AgoraRTC.createClient({ mode: 'rtc', codec: 'h264' }),
      localStream:{},
      // localStream: AgoraRTC.createStream({streamID: uid, audio: true, video: true, screen: false}),
      // localStream: {
      //   uid: '',
      //   camera: {
      //     camId: '',
      //     micId: '',
      //     stream: {}
      //   }
      // },

      uid: 0,
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
  enterVideo() {
    this.open= !this.open

    this.client.init(this.appId, () =>
    console.log('AgoraRTC this.client initialized'));

    let uidId = 0;

    // console.log(this.localStream)
    // console.log('================= 98 =========================');
    console.log('================= 1 =========================');
    this.client.join(this.appKey, this.channelName, null, (uid) => {
      console.log('================= localStream =========================');

      this.localStream = AgoraRTC.createStream({streamID: uid, audio: true, video: true, screen: false})

      console.log(localStream)

      uidId = uid;

      // this.localStream = AgoraRTC.createStream({streamID: uid, audio: true, video: true, screen: false})
      // console.log('==========================================');
      console.log(`User ${uid} joined channel`);
      // console.log('==========================================');
    });

    // console.log('==========================================');
    // console.log(this.localStream);
    console.log('==================2========================');


    this.localStream.init(() => {
      console.log('=================3=========================');
      console.log('Local stream initialized');
      this.localStream.play('video');
      this.client.publish(this.localStream, (err) =>
      console.log('Publish local stream error: ' + err));
    });

    // Publish the local stream
    this.client.publish(this.localStream, function (err) {
      console.log("publish failed");
      console.error(err);
    });
    console.log('==================4========================');

    this.client.on('stream-added', (evt) => {
      // const stream = evt.stream;
      var remoteStream = evt.stream;
      var id = remoteStream.getId();
      if (id !== uidId) {
        this.client.subscribe(remoteStream, function (err) {
          console.log("stream subscribe failed", err);
        });
      }
      console.log("stream-added remote-uid: ", id);
      console.log(`Stream added:${evt}`)
    });
    console.log('===================5=======================');

    this.client.on("stream-subscribed", function (evt) {
      var remoteStream = evt.stream;
      var id = remoteStream.getId();
      // Add a view for the remote stream.
      let streamDiv =  document.createElement("div"); // Create a new div for every stream
      streamDiv.id = id;                            // Assigning id to div
      streamDiv.style.transform="rotateY(180deg)"; // Takes care of lateral inversion (mirror image)
      remoteContainer.appendChild(streamDiv);
      // Play the remote stream.
      remoteStream.play("remote_video_" + id);
      console.log("stream-subscribed remote-uid: ", id);
    });
    console.log('================6==========================');
  },

  joinChannel() {
    var token = generateToken();
    var userID = null; // set to null to auto generate uid on successfull connection
    // set the role
    this.client.setClientRole('host', function() {
      console.log('this.Client role set as host.');
    }, function(e) {
      console.log('setthis.ClientRole failed', e);
    });

    // this.client.join(token, 'allThingsRTCLiveStream', 0, function(uid) {
    this.client.join(token, channelName, userID, function(uid) {
        createCameraStream(uid, {});
        this.localStreams.uid = uid; // keep track of the stream uid
        console.log('User ' + uid + ' joined channel successfully');
    }, function(err) {
        console.log('[ERROR] : join channel failed', err);
    });
  },

// video streams for channel
     createCameraStream(uid, deviceIds) {
      console.log('Creating stream with sources: ' + JSON.stringify(deviceIds));

      this.localStream.setVideoProfile(cameraVideoProfile);

      // The user has granted access to the camera and mic.
      this.localStream.on("accessAllowed", function() {
        if(devices.cameras.length === 0 && devices.mics.length === 0) {
          console.log('[DEBUG] : checking for cameras & mics');
          getCameraDevices();
          getMicDevices();
        }
        console.log("accessAllowed");
      });
      // The user has denied access to the camera and mic.
      this.localStream.on("accessDenied", function() {
        console.log("accessDenied");
      });

      this.localStream.init(function() {
        console.log('getUserMedia successfully');
        this.localStream.play('full-screen-video'); // play the local stream on the main div
        // publish local stream

        if($.isEmptyObject(this.localStreams.camera.stream)) {
          enableUiControls(this.localStream); // move after testing
        } else {
          //reset controls
          $("#mic-btn").prop("disabled", false);
          $("#video-btn").prop("disabled", false);
          $("#exit-btn").prop("disabled", false);
        }

        this.client.publish(this.localStream, function (err) {
          console.log('[ERROR] : publish local stream error: ' + err);
        });

        localStreams.camera.stream = this.localStream; // keep track of the camera stream for later
      }, function (err) {
        console.log('[ERROR] : getUserMedia failed', err);
      });
    },



      leaveChannel() {
        this.client.leave(function() {
          console.log('this.client leaves channel');
          this.localStream.stop() // stop the camera stream playback
          this.localStream.close(); // clean up and close the camera stream
          this.client.unpublish(this.localStream); // unpublish the camera stream
          //disable the UI elements
          $('#mic-btn').prop('disabled', true);
          $('#video-btn').prop('disabled', true);
          $('#exit-btn').prop('disabled', true);
          $("#add-rtmp-btn").prop("disabled", true);
          $("#rtmp-config-btn").prop("disabled", true);
        }, function(err) {
          console.log('this.client leave failed ', err); //error handling
        });
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
  #video {
    width: 640px;
    height: 480px;
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
    border: 2px solid;
    width: auto;
    height: auto;
    display: grid;
    grid-column: auto auto auto;
    border-width: 2px;
    border-radius: 6px;
  }


</style>
