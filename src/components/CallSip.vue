<template>
  <div class="hello">
    <TopNavBar />
    <div id="errorMessage">{{ errorMessage }}</div>
    <div style="text-align: center" v-if="disconnected">
      <p style="margin-bottom: 0px">
        reconnection attempt: {{ recoverAttempts }}
      </p>
      <p>next connection attempt in {{ recoveryTimer }} seconds</p>
    </div>

    <div id="wrapper" v-if="showNumbers">
      <div
        id="incomingCall"
        v-if="incomingCall"
        class="d-flex mt-3"
        style="top: 185px"
      >
        <div class="callInfo">
          <h5>Incoming Call</h5>
          <p id="incomingCallNumber">{{ incomingCallNumber }}</p>
        </div>
        <div
          id="answer"
          @click="answerCallClick"
          class="d-flex justify-content-center"
        >
          <i class="fa fa-phone"></i>
        </div>
        <div
          id="reject"
          @click="rejectCallClick"
          class="d-flex justify-content-center"
        >
          <i class="fa fa-phone"></i>
        </div>
      </div>

      <div id="callStatus" v-if="callStatus">
        <div class="callInfo">
          <h5 id="callInfoText">{{ callInfoText }}</h5>
          <p id="callInfoNumber">{{ callInfoNumber }}</p>
        </div>
        <div id="hangUp" class="d-flex justify-content-center">
          <i class="fa fa-phone" @click="hangUp"></i>
        </div>
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
        <div id="mute" @click="muteCall">
          <i id="muteIcon" class="fa" :class="muteIcon"></i>
        </div>
        <hr />
      </div>

      <!---------DIAL CONTROLS-------------------------------------------->
      <div id="callControl" class="mt-3" style="display: block">
        <div id="to">
          <input
            id="toField"
            v-model="dest"
            type="text"
            placeholder="Enter number here"
          />
        </div>
        <div id="connectCall" @click="connectCallClick()">
          <i class="fa fa-phone"></i>
        </div>
      </div>

      <div class="pt-4 notification-body" id="listCall">
        <strong>Ligações</strong>
        <hr />
        <div v-for="(item, index) in notifications" :key="index">
          <b-row>
            <b-col cols="10" class="d-flex flex-column"
              ><span>{{ item.name }}</span> <small>{{ item.sip }}</small></b-col
            >
            <b-col
              cols="2"
              class="d-flex justify-content-center align-items-center"
              style="cursor: pointer"
            >
              <i class="fa fa-phone" @click="clickCall(item.sip)"></i>
            </b-col>
          </b-row>
          <hr />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import JsSIP from 'jssip';
import TopNavBar from './TopNavBar';
import notifications from '../data/notifications';
export default {
  name: 'CallSip',
  props: {
    msg: String
  },
  components: {
    TopNavBar
  },
  data() {
    return {
      notifications,
      callOptions: {
        mediaConstraints: { audio: true, video: false }
      },
      configuration: {
        sockets: [new JsSIP.WebSocketInterface('wss://20.51.122.220:8089/ws')],
        // FILL SIP URI HERE like sip:sip-user@your-domain.bwapp.bwsip.io
        uri: 'sip:6001@20.51.122.220',
        password: '6001$', // FILL PASSWORD HERE
        register: true
      },
      dest: 'sip:teste_cbrdxv@tryit.jssip.net',
      incomingCallAudio: new window.Audio(require('@/assets/dial-tone.mp3')),
      phone: null,
      remoteAudio: new window.Audio(),
      session: null,
      errorMessage: 'must set sip uri/password',
      showNumbers: false,
      muteIcon: 'fa-microphone',
      callInfoNumber: '',
      callInfoText: '',
      callStatus: false,
      incomingCallNumber: '',
      incomingCall: false,
      recoverAttempts: '',
      recoveryTimer: '',
      disconnected: false
    };
  },
  methods: {
    incomingCallOptions() {
      this.incomingCallAudio.loop = true;
      this.incomingCallAudio.crossOrigin = 'anonymous';
    },
    remoteAudioOptions() {
      this.remoteAudio.autoplay = true;
      this.remoteAudio.crossOrigin = 'anonymous';
    },
    startApp() {
      this.incomingCallOptions();
      this.remoteAudioOptions();

      this.phone = new JsSIP.UA(this.configuration);

      if (this.configuration.uri && this.configuration.password) {
        JsSIP.debug.enable('JsSIP:*');
        this.phone.on('registrationFailed', ev => {
          this.errorMessage =
            'Registering on SIP server failed with error: ' + ev.cause;
          this.configuration.uri = null;
          this.configuration.password = null;
          this.showNumbers = false;

          this.updateUI();
        });

        this.phone.on('connected', function () {
          console.log('connected');
        });

        this.phone.on('disconnected', () => {
          this.disconnected = true;
          this.recoverAttempts = this.phone.transport.recover_attempts + 1;
          this.recoveryTimer = this.phone.transport.recovery_timer + 1;
        });

        this.phone.on('newRTCSession', ev => {
          const newSession = ev.session;

          if (this.session) {
            // hangup any existing call
            this.session.terminate();
          }

          this.session = newSession;
          const completeSession = () => {
            this.session = null;
            this.updateUI();
          };

          this.session.on('peerconnection', data => {
            data.peerconnection.addEventListener('addstream', e => {
              this.incomingCallAudio.pause();
              this.remoteAudio.srcObject = e.stream;
            });
          });

          this.session.on('ended', completeSession);
          this.session.on('failed', completeSession);
          this.session.on('accepted', () => {
            this.updateUI();
          });
          this.session.on('confirmed', () => {
            this.updateUI();
          });

          if (this.session.direction === 'incoming') {
            this.incomingCallAudio.play();
          } else {
            this.session.connection.addEventListener('addstream', e => {
              this.incomingCallAudio.pause();
              this.remoteAudio.srcObject = e.stream;
            });
          }
          this.updateUI();
        });

        this.updateUI();
        this.phone.start();
      }
    },
    updateUI() {
      if (this.configuration.uri && this.configuration.password) {
        this.errorMessage = '';
        this.showNumbers = true;

        if (this.session) {
          if (this.session.isInProgress()) {
            if (this.session.direction === 'incoming') {
              this.incomingCallNumber = this.session.remote_identity.uri;
              this.incomingCall = true;
            } else {
              this.callInfoText = 'Ringing...';
              this.callInfoNumber = this.session.remote_identity.uri.user;
              this.callStatus = true;
            }
          } else if (this.session.isEstablished()) {
            this.callInfoText = 'In Call';
            this.callInfoNumber = this.session.remote_identity.uri.user;
            this.callStatus = true;
            this.incomingCallAudio.pause();
          }
        } else {
          this.callStatus = false;
          this.incomingCall = false;
          this.incomingCallAudio.pause();
        }

        //microphone mute icon
        if (this.session && this.session.isMuted().audio) {
          this.muteIcon = 'fa-microphone-slash';
        } else {
          this.muteIcon = 'fa-microphone';
        }
      } else {
        this.showNumbers = false;
      }
    },
    connectCallClick() {
      this.phone.call(this.dest, this.callOptions);
    },
    answerCallClick() {
      this.session.answer(this.callOptions);
    },
    terminateCall() {
      this.session.terminate();
    },
    rejectCallClick() {
      this.terminateCall();
    },
    hangUp() {
      this.terminateCall();
    },
    muteCall() {
      if (this.session.isMuted().audio) {
        this.session.unmute({ audio: true });
      } else {
        this.session.mute({ audio: true });
      }
    },
    clickCall(sip) {
      this.dest = sip;
      this.connectCallClick();
    }
  },
  mounted() {
    this.startApp();
  }
};
</script>

<style scoped>
@charset "UTF-8";

#listCall .fa-phone {
  font-size: 24px;
}

/* prettier-ignore */
#incomingCall,
#callStatus {
  position: fixed;
  top: 76px;
  right: 30px;
  z-index: 20;
  background: white;
  box-shadow: 0 10px 16px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  padding: 15px;
}

/* CSS Document */
#wrapper {
  width: 300px;
  margin: 0 auto;
}
#errorMessage {
  text-align: center;
}
.callInfo {
  width: 150px;
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
.callInfo h5 {
  color: #389400;
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
  float: right;
}
#hangUp {
  background-color: #a90002;
}
#reject {
  background-color: #fff;
  color: #a90002;
  margin-right: 10px;
  margin-left: 10px;
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
  padding-left: 10px;
  font-size: 1em;
  font-family: 'Open Sans', sans-serif;
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
