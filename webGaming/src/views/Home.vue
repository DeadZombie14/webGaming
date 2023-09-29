<template>
  <div id="screenContainer" ref="mainScreen">
    <video ref="screen" autoplay muted></video>
    <div class="videoOverlay">
      <button class="btn playBtn" @click="play()" v-if="videoStreamSource == null">Start transmision</button>
    </div>
  </div>
</template>

<style scoped>
#screenContainer {
  padding: 1rem;
  width: 100%;
  height: 100vh;
  background: var(--mainColor);
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}
#screenContainer video {
  max-width: 100%;
  max-height: 100%;
  height: 100%;
  background: black;
  border: 1rem solid black;
  border-radius: 1rem;
}
.videoOverlay {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0; 
  display: flex;
  justify-content: center;
  align-items: center;
}
.playBtn {
  
}
</style>

<script setup lang="ts">
import { ref, computed } from "vue"
import { useRoute } from 'vue-router';
const route = useRoute()

import { Peer } from "peerjs"

// Query params
const hostID = route.params.id as string | null

// References of elements
const screen = ref<HTMLVideoElement | null>(null)

// Main code
const connected: string[] = []
let mediaStreamCalls: any[] = []
let videoStreamSource = ref<MediaStream | null>(null)

const peer = new Peer()
peer.on('open',function(id) {
  if (!hostID) {
    // We are the master, listen for calls
    console.log('Join via link: ' + window.location.origin + '/room/' + id)
    peer.on('connection', function(conn) {
      conn.on('open', function() {
        conn.send(`welcome ${conn.peer} !`)
        if(videoStreamSource.value) {
            mediaStreamCalls.push(peer.call(conn.peer, videoStreamSource.value))
        }
        connected.push(conn.peer) // Add to connected list
      })
      
      // Receive messages
      conn.on('data', function(data) {
          console.log(`${conn.peer} says: ${data}`)
      })
      
      // Client left, remove from list
      conn.on('close', function() {
          conn.close()
          console.log(conn)
          console.log(`${conn.peer} left the room!`)
      })
      conn.on('error', function() {
          conn.close()
          console.log(`${conn.peer} left the room!`)
      })
    })
  } else {
    // We are "clients", connect to master
    console.log('My client ID is: '+ id)
    const conn = peer.connect(hostID)
    conn.on('open', function() {
      // Send messages
      conn.send('Hello!')
      
      // Receive messages
      conn.on('data', function(data) {
        console.log(`Master says: ${data}`)
      })
    })

    peer.on('call', function(call) {
      call.answer() // Answer master's call
      
      // Show master stream in some video/canvas element.
      call.on('stream', function(remoteStream) {
        // const videotrack = remoteStream.getVideoTracks()[0]
        // videotrack.applyConstraints({ frameRate: { min: 60 } })
        screen.value!.srcObject = remoteStream
        videoStreamSource.value = remoteStream
      })
      
      // Video stream ended
      call.on('close', function() {
        screen.value!.srcObject = null
      })
    })
  }
})

function play() {
  navigator.mediaDevices.getDisplayMedia({audio: true}).then(stream => {
    const videotrack = stream.getVideoTracks()[0]
    videotrack.applyConstraints({ frameRate: { min: 60 } })

    screen.value!.srcObject = stream
    videoStreamSource.value = stream
    
    // Video stream started, create one mediaStreamCall per client
    for (let peerID of connected) {
      mediaStreamCalls.push(peer.call(peerID, stream))
    }

    // Video stream ended, kill all mediaStreamCalls
    stream.getVideoTracks()[0].addEventListener('ended', () => {
      mediaStreamCalls.forEach(call => {
        call.close()
      })
      mediaStreamCalls = []
      videoStreamSource.value = null
    })
  })
}

</script>
