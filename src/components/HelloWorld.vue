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
// b6a396427ce84882a61f8b7036de501a
  data(){
    return{
      open: false,


      rtc: {
        client: null,
        joined: false,
        published: false,
        localStream: null,
        remoteStreams: [],
        params: {}
      },
      option: {
        mode: "rtc",
        codec: "h264",
        appID: "b6a396427ce84882a61f8b7036de501a",
        channel: "poc2",
        uid: null,
        token: "007eJxTYNh1QWzvyavrOEy+vd3z80mzTY7sk5rHMdlrrYLvXJr6yltUgSHRzCzR3CDZzNA0zdAkzSLJ0tDC0jQxxTDNJMnY0DzReMbPTckNgYwM7BH5LIwMEAjiszAU5CcbMTAAAEKPIeM="
      },

      uid: 0,
      devices: {
        cameras: [],
        mic: []
      },


    }
  },

methods: {
  enterVideo() {
    this.open= !this.open
      // let client =  AgoraRTC.createClient({ mode: 'rtc', codec: 'h264' })
    this.rtc.client = AgoraRTC.createClient({mode: this.option.mode, codec: this.option.codec});
    this.rtc.client.init(this.option.appID, () =>
    console.log('AgoraRTC this.client initialized'));

    let uidId = 0;


    this.rtc.client.join(this.option.token, this.option.channel, null, (uid) => {

      this.rtc.localStream = AgoraRTC.createStream({streamID: uid, audio: true, video: true, screen: false})

      console.log(this.rtc.localStream)

      uidId = uid;

      console.log(`User ${uid} joined channel`);
    },(err) => {
      console.log(err)
    });



    this.rtc.localStream.init(() => {
      console.log('Local stream initialized');
      this.rtc.localStream.play('video');
      this.rtc.client.publish(this.rtc.localStream, (err) =>
      console.log('Publish local stream error: ' + err));

    });

    // Publish the local stream
    this.rtc.client.publish(this.rtc.localStream, function (err) {
      console.log("publish failed");
      console.error(err);
    });


    this.rtc.client.on('stream-added', (evt) => {
      // const stream = evt.stream;
      var remoteStream = evt.stream;
      var id = remoteStream.getId();
      if (id !== uidId) {
        this.rtc.client.subscribe(remoteStream, function (err) {
          console.log("stream subscribe failed", err);
        });
      }
      console.log("stream-added remote-uid: ", id);
      console.log(`Stream added:${evt}`)
    });


    this.rtc.client.on("stream-subscribed", function (evt) {
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

  },
// use tokens for added security
generateToken() {
  return null; // TODO: add a token generation
},

  joinChannel() {
    var tokens = this.generateToken();
    var userID = null; // set to null to auto generate uid on successfull connection
    // set the role
    this.rtc.client.setClientRole('host', function() {
      console.log('this.Client role set as host.');
    }, function(e) {
      console.log('setthis.ClientRole failed', e);
    });
    console.log('=========================')
    // this.client.join(token, 'allThingsRTCLiveStream', 0, function(uid) {
    this.rtc.client.join(tokens, this.option.channel, userID, function(uid) {
        createCameraStream(uid, {});
        this.rtc.localStream.uid = uid; // keep track of the stream uid
        console.log('User ' + uid + ' joined channel successfully');
    }, function(err) {
        console.log('[ERROR] : join channel failed', err);
    });
  },

// video streams for channel
     createCameraStream(uid, deviceIds) {
      console.log('Creating stream with sources: ' + JSON.stringify(deviceIds));

      this.rtc.localStream.setVideoProfile(cameraVideoProfile);

      // The user has granted access to the camera and mic.
      this.rtc.localStream.on("accessAllowed", function() {
        if(devices.cameras.length === 0 && devices.mics.length === 0) {
          console.log('[DEBUG] : checking for cameras & mics');
          getCameraDevices();
          getMicDevices();
        }
        console.log("accessAllowed");
      });
      // The user has denied access to the camera and mic.
      this.rtc.localStream.on("accessDenied", function() {
        console.log("accessDenied");
      });

      this.rtc.localStream.init(function() {
        console.log('getUserMedia successfully');
        this.rtc.localStream.play('full-screen-video'); // play the local stream on the main div
        // publish local stream

        if($.isEmptyObject(this.rtc.localStreams.camera.stream)) {
          enableUiControls(this.rtc.localStream); // move after testing
        } else {
          //reset controls
          $("#mic-btn").prop("disabled", false);
          $("#video-btn").prop("disabled", false);
          $("#exit-btn").prop("disabled", false);
        }

        this.rtc.client.publish(this.rtc.localStream, function (err) {
          console.log('[ERROR] : publish local stream error: ' + err);
        });

        localStreams.camera.stream = this.rtc.localStream; // keep track of the camera stream for later
      }, function (err) {
        console.log('[ERROR] : getUserMedia failed', err);
      });
    },
    leaveChannel() {
      this.rtc.client.leave(function() {
        console.log('this.rtc.client leaves channel');
        this.localStream.stop() // stop the camera stream playback
        this.localStream.close(); // clean up and close the camera stream
        this.rtc.client.unpublish(this.localStream); // unpublish the camera stream
        //disable the UI elements
        $('#mic-btn').prop('disabled', true);
        $('#video-btn').prop('disabled', true);
        $('#exit-btn').prop('disabled', true);
        $("#add-rtmp-btn").prop("disabled", true);
        $("#rtmp-config-btn").prop("disabled", true);
      }, function(err) {
        console.log('this.rtc.client leave failed ', err); //error handling
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
    justify-content: flex-start;
    align-items: flex-start;
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
