<template>
  <div class="hello">
    <input type="text" v-model="username">
    <button @click="createRoom">Create Room</button>
    <video id="selfview" autoplay></video>
    <video id="remoteview"></video>
  </div>
</template>

<script>
import io from 'socket.io-client'

export default {
  name: 'HelloWorld',
  props: {
    theSocket: Object,
    userId: String
  },
  data () {
    return {
      caller: null,
      username: null,
      socketio: null, //io('http://localhost:8100/qwerasdfqwer'),
      isStreamer: false,
      streamerId: null,
      localsdp: null,
      remotesdp: null,
      localUserMedia: null,
      sessionDesc: null
    }
  },
  methods: {
    getCam: function () {
      return navigator.mediaDevices.getUserMedia({
        video: { width: 640, height: 400 },
        audio: true
      })
    },
    createRoom: function () {
      return null
    },
    prepareCaller: function () {
      console.log('preparing caller')
      // Initializing a peer connection
      this.caller = new window.RTCPeerConnection()
      // Listen for ICE Candidates and send them to remote peers
      let onice = function (evt) {
        console.log('onicecandidate called')
        console.log(evt)
        if (!evt.candidate) return
        this.onIceCandidate(this.caller, evt)
      }
      this.caller.onicecandidate = onice.bind(this)
      // onaddstream handler to receive remote feed and show in remoteview video element
      this.caller.onaddstream = function (evt) {
        console.log('onaddstream called')
        console.log(window.URL)
        if (window.URL) {
          document.getElementById('remoteview').srcObject = evt.stream
        } else {
          document.getElementById('remoteview').src = evt.stream
        }
      }
    },
    enterConference: function () {
      console.log('...joining ' + this.theSocket.namespace)
      if (!this.isStreamer) {
        console.log('no es streamer')
        this.getCam()
          .then((stream) => {
            this.localUserMedia = stream
            if (window.URL) {
              document.getElementById('selfview').srcObject = stream
            } else {
              document.getElementById('selfview').src = stream
            }
            this.caller.addStream(stream)
            this.caller.createOffer()
            .then((desc) => {
              console.log('setting local description')
              this.caller.setLocalDescription(new RTCSessionDescription(desc))
              this.socketio.emit('client-sdp',{
                'sdp': desc,
                'userId': this.userId
                })
            })
          })
      } else {
        this.getCam()
          .then((stream) => {
            this.caller.addStream(stream)
            this.localUserMedia = stream
            if (window.URL) {
              document.getElementById('selfview').srcObject = stream
            } else {
              document.getElementById('selfview').src = stream
            }
          })
      }
    },
    clientSDP: function (msg) {
      console.log('enter clientSDP')
      // pidiendo al streamer que encienda su camara
      this.sessionDesc = new RTCSessionDescription(msg.sdp)
      console.log('setting remote description')
      this.caller.setRemoteDescription(this.sessionDesc)
      this.caller.createAnswer()
        .then((sdp) => {
          console.log('setting local description')
          this.caller.setLocalDescription(new RTCSessionDescription(sdp))
          this.socketio.emit('client-answer', {
            'sdp': sdp, 'user': this.userId, 'streamer': this.streamerId
          })
        })
      .catch(error => {
        console.log('an error occured', error)
      })
    },
    onIceCandidate: function (peer, evt) {
      console.log('onIceCandidate')
      // console.log(this.room)
      if (evt.candidate) {
        console.log('candidato')
        console.log(evt.candidate)
        this.socketio.emit('client-candidate', {
          'candidate': evt.candidate,
          'user': this.userId,
          'streamer': this.streamerId
        })
      }
    },
    clientAnswer: function (answer) {
      //  this.caller.addStream(stream)
      console.log('setting remote description')
      this.caller.setRemoteDescription(new RTCSessionDescription(answer.sdp))
    },
    clientCandidate: function (msg) {
      this.caller.addIceCandidate(new RTCIceCandidate(msg.candidate))
    }
  },
  created: function () {
    console.log('created')
  },
  mounted () {
    console.log('mounted')
    console.log(this.theSocket)
    this.socketio = io('http://localhost:8100/qwerasdfqwer')
    console.log(this.socketio)

    GetRTCPeerConnection()
    GetRTCSessionDescription()
    GetRTCIceCandidate()
    this.prepareCaller()
    function GetRTCIceCandidate () {
      window.RTCIceCandidate = window.RTCIceCandidate || window.webkitRTCIceCandidate ||
          window.mozRTCIceCandidate || window.msRTCIceCandidate
      return window.RTCIceCandidate
    }
    function GetRTCPeerConnection () {
      window.RTCPeerConnection = window.RTCPeerConnection || window.webkitRTCPeerConnection ||
          window.mozRTCPeerConnection || window.msRTCPeerConnection
      return window.RTCPeerConnection
    }
    function GetRTCSessionDescription () {
      window.RTCSessionDescription = window.RTCSessionDescription || window.webkitRTCSessionDescription ||
          window.mozRTCSessionDescription || window.msRTCSessionDescription
      return window.RTCSessionDescription
    }

    this.socketio.on('client-answer', (data) => {
      console.log('clientAnswer')
      // check if answer is for me
      if (this.userId == data.user) {
        console.log('ignoring my answer')
      } else {
        console.log('accepting answer')
        this.clientAnswer(data)
      }
    })

    // viene del server
    this.socketio.on('client-sdp', (data) => {
      console.log('clientSDP server')
      if (this.userId == data.from) {
        console.log('ignored')
        return null
      } else {
        this.streamerId = data.streamer
        return this.clientSDP(data)
      }
    })

    this.socketio.on('streamer', (data) => {
      console.log('streamer')
      console.log(data)
      this.isStreamer = data.isStreamer
      this.enterConference()
    })

    this.socketio.on('client-candidate', (data) => {
      if (data.from == this.userId) {
        console.log('ignore candidate')
      } else {
        this.clientCandidate(data)
      }
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
