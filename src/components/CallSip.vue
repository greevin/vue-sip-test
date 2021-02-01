<template>
  <div class="hello">
    <div id="errorMessage">must set sip uri/password</div>
    <div id="wrapper">
      <div id="incomingCall" style="display: none">
        <div class="callInfo">
          <h3>Incoming Call</h3>
          <p id="incomingCallNumber">Unknown</p>
        </div>
        <div id="answer"><i class="fa fa-phone"></i></div>
        <div id="reject"><i class="fa fa-phone"></i></div>
      </div>
      <div id="callStatus" style="display: none">
        <div class="callInfo">
          <h3 id="callInfoText">info text goes here</h3>
          <p id="callInfoNumber">info number goes here</p>
        </div>
        <div id="hangUp"><i class="fa fa-phone"></i></div>
      </div>
      <!---------TO FIELD---------------------------------------------------->
      <!---------DIALPAD---------------------------------------------------->
      <div id="inCallButtons" style="display: none">
        <div id="dialPad">
          <div class="dialpad-char" data-value="1" unselectable="on">1</div>
          <div class="dialpad-char" data-value="2" unselectable="on">2</div>
          <div class="dialpad-char" data-value="3" unselectable="on">3</div>
          <div class="dialpad-char" data-value="4" unselectable="on">4</div>
          <div class="dialpad-char" data-value="5" unselectable="on">5</div>
          <div class="dialpad-char" data-value="6" unselectable="on">6</div>
          <div class="dialpad-char" data-value="7" unselectable="on">7</div>
          <div class="dialpad-char" data-value="8" unselectable="on">8</div>
          <div class="dialpad-char" data-value="9" unselectable="on">9</div>
          <div class="dialpad-char" data-value="*" unselectable="on">*</div>
          <div class="dialpad-char" data-value="0" unselectable="on">0</div>
          <div class="dialpad-char" data-value="#" unselectable="on">#</div>
        </div>
        <div id="mute">
          <i id="muteIcon" class="fa fa-microphone"></i>
        </div>
      </div>

      <!---------DIAL CONTROLS-------------------------------------------->
      <div id="callControl">
        <div id="to">
          <input id="toField" type="text" placeholder="Enter number here" />
        </div>
        <div id="connectCall"><i class="fa fa-phone"></i></div>
      </div>
    </div>
  </div>
</template>

<script>
import JsSIP from "jssip";
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      configuration: {
        sockets: [new JsSIP.WebSocketInterface("wss://tryit.jssip.net:10443")],
        uri: "sip:teste_cbrdxv@tryit.jssip.net", // FILL SIP URI HERE like sip:sip-user@your-domain.bwapp.bwsip.io
        username: "teste", // FILL USERNAME HERE
        password: "1234", // FILL PASSWORD HERE
        register: true,
      },
      phone: null,
      session: null,
      incomingCallAudio: new window.Audio("@/assets/dial-tone.mp3"),
      remoteAudio: new window.Audio(),
      callOptions: {
        mediaConstraints: { audio: true, video: false },
      },
    };
  },
  methods: {
    incomingCallOptions() {
      this.incomingCallAudio.loop = true;
      this.incomingCallAudio.crossOrigin = "anonymous";
    },
    remoteAudioOptions() {
      this.remoteAudio.autoplay = true;
      this.remoteAudio.crossOrigin = "anonymous";
    },
    startApp() {
      this.incomingCallOptions();
      this.remoteAudioOptions();

      if (this.configuration.uri && this.configuration.password) {
        JsSIP.debug.enable("JsSIP:*");
        this.phone = new JsSIP.UA(this.configuration);
        this.phone.on("registrationFailed", function (ev) {
          console.log(ev);
          alert("Registering on SIP server failed with error: " + ev.cause);
          this.configuration.uri = null;
          this.configuration.password = null;
          // updateUI();
        });

        this.phone.on("newRTCSession", function (ev) {
          let newSession = ev.session;

          if (this.session) {
            // hangup any existing call
            this.session.terminate();
          }

          this.session = newSession;
          let completeSession = function () {
            this.session = null;
            // updateUI();
          };

          this.session.on("ended", completeSession);
          this.session.on("failed", completeSession);
          // this.session.on("accepted", updateUI);
          this.session.on("confirmed", function () {
            let localStream = this.session.connection.getLocalStreams()[0];
            let dtmfSender = this.session.connection.createDTMFSender(
              localStream.getAudioTracks()[0]
            );
            this.session.sendDTMF = function (tone) {
              dtmfSender.insertDTMF(tone);
            };
            // updateUI();
          });

          this.session.on("peerconnection", (e) => {
            console.log("peerconnection", e);
            // let logError = "";
            const peerconnection = e.peerconnection;

            peerconnection.onaddstream = function (e) {
              console.log("addstream", e);
              // set remote audio stream (to listen to remote audio)
              // remoteAudio is <audio> element on pag
              this.remoteAudio.srcObject = e.stream;
              this.remoteAudio.play();
            };

            let remoteStream = new MediaStream();
            console.log(peerconnection.getReceivers());
            peerconnection.getReceivers().forEach(function (receiver) {
              console.log(receiver);
              remoteStream.addTrack(receiver.track);
            });
          });

          if (this.session.direction === "incoming") {
            this.incomingCallAudio.play();
          } else {
            console.log("con", this.session.connection);
            this.session.connection.addEventListener("addstream", function (e) {
              this.incomingCallAudio.pause();
              this.remoteAudio.srcObject = e.stream;
            });
          }
          // updateUI();
        });

        this.phone.start();
      }
    },
    updateUI() {
      console.log("updateUI");
    },
  },
  mounted() {
    this.startApp();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@charset "UTF-8";

/* CSS Document */
html,
body {
  margin: 0;
  padding: 0;
  font-family: "Open Sans", sans-serif;
  color: #494949;
}
#wrapper {
  width: 300px;
  margin: 0 auto;
}
.callInfo {
  width: 190px;
  height: 70px;
  float: left;
}
#connectedCall .callInfo {
  width: 240px;
}
#dialPad div {
  -moz-user-select: -moz-none;
  -khtml-user-select: none;
  -webkit-user-select: none;

  /*
    Introduced in IE 10.
    See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
    */
  -ms-user-select: none;
  user-select: none;
}
.callInfo h3 {
  color: #389400;
  margin: 10px 0 10px 0;
}
.callInfo p {
  margin: 0px 25px 0px 0px;
  float: left;
}
#answer,
#hangUp,
#reject,
#connectCall,
#mute {
  color: #fff;
  background-color: #389400;
  width: 50px;
  height: 50px;
  float: right;
  text-align: center;
  font-size: 30px;
  margin: 10px 0px 10px 0px;
  border-radius: 25px 25px 25px 25px;
  -moz-border-radius: 25px 25px 25px 25px;
  -webkit-border-radius: 25px 25px 25px 25px;
  cursor: pointer;
  cursor: hand;
}
#mute:active,
#connectCall:active,
#reject:active,
#hangUp:active,
#answer:active {
  background-color: #b6b6b6;
  -webkit-box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  -moz-box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  border: none;
}
#hangUp,
#reject {
  -ms-transform: rotate(135deg);
  /* IE 9 */
  -webkit-transform: rotate(135deg);
  /* Chrome, Safari, Opera */
  transform: rotate(135deg);
}
#hangUp {
  background-color: #a90002;
}
#reject {
  background-color: #fff;
  color: #a90002;
  margin-right: 10px;
}
#connectCall,
#mute {
  color: #fff;
  background-color: #389400;
  width: 260px;
  height: 50px;
  font-size: 30px;
  text-align: center;
  margin: 20px 20px 0px 20px;
  border-radius: 5px 5px 5px 5px;
  -moz-border-radius: 5px 5px 5px 5px;
  -webkit-border-radius: 5px 5px 5px 5px;
  cursor: pointer;
  cursor: hand;
}
#mute {
  color: #545454;
  margin: 0 auto;
  width: 50px;
  height: 50px;
  margin: 15px 125px 10px 20px;
  border-radius: 25px 25px 25px 25px;
  -moz-border-radius: 25px 25px 25px 25px;
  -webkit-border-radius: 25px 25px 25px 25px;
  background-color: #fff;
}
.muteActive {
  color: #fff;
  background-color: #389400;
}
#to {
  border-bottom: 2px solid #bbbbbb;
}
#toField {
  margin-top: 20px;
  padding-left: 10px;
  font-size: 1em;
  font-family: "Open Sans", sans-serif;
  width: 300px;
  height: 40px;
  border-radius: 2px 2px 2px 2px;
  -moz-border-radius: 2px 2px 2px 2px;
  -webkit-border-radius: 2px 2px 2px 2px;
  border: 0px solid #b8b8b8;
}
#dialPad {
  width: 240px;
  height: 308px;
  margin: 0 auto;
}
#dialPad div {
  float: left;
  width: 50px;
  height: 50px;
  text-align: center;
  font-size: 26px;
  margin: 25px 15px 0px 15px;
  box-sizing: border-box;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  border-radius: 25px 25px 25px 25px;
  -moz-border-radius: 25px 25px 25px 25px;
  -webkit-border-radius: 25px 25px 25px 25px;
  border: 1px solid #e8e8e8;
  padding-top: 5px;
}
#dialPad div:hover {
  background-color: #389400;
  color: #fff;
  cursor: pointer;
  cursor: hand;
}
#dialPad div:active {
  background-color: #b2b2b2;
  -webkit-box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  -moz-box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  box-shadow: 0px 1px 7px 2px rgba(0, 0, 0, 0.45);
  border: none;
}
.fa {
  margin-top: 11px;
}
#answer .fa {
  -webkit-animation: callp 1s;
  animation: callp 1s;
  -webkit-animation-iteration-count: infinite;
  -webkit-animation-direction: alternate;
  -webkit-animation-play-state: running;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  animation-play-state: running;
}
@-webkit-keyframes callp {
  from {
    color: #fff;
  }
  to {
    color: #389400;
  }
}
@keyframes callp {
  from {
    color: #fff;
  }
  to {
    color: #389400;
  }
}
#answer {
  -webkit-animation: call 1s;
  animation: call 1s;
  -webkit-animation-iteration-count: infinite;
  -webkit-animation-direction: alternate;
  -webkit-animation-play-state: running;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  animation-play-state: running;
}
@-webkit-keyframes call {
  from {
    background: #389400;
  }
  to {
    background: #fff;
  }
}
@keyframes call {
  from {
    background: #389400;
  }
  to {
    background: #fff;
  }
}
</style>
