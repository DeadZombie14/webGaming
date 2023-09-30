<template>
  <main>
    <div class="left-sidebar">
      <button class="sidebar-btn" style="background-color: darkviolet;" @click="copyShareLink">
        <span class="material-symbols-outlined">offline_share</span>
      </button>
      <button class="sidebar-btn" style="background-color: darkviolet;">
        <span class="material-symbols-outlined">settings</span>
      </button>
    </div>
    <div id="screenContainer" ref="mainScreen">
      <video ref="screen" autoplay muted></video>
      <div class="videoOverlay">
        <button class="btn playBtn" @click="play()" v-if="videoStreamSource == null && !hostID">Start transmision</button>
        <span v-if="videoStreamSource == null && hostID">Waiting for host to start...</span>
      </div>
    </div>
    <div class="right-sidebar">
      <button class="sidebar-btn" style="background-color: orange;">
        <span>Player 1</span>
        <span class="material-symbols-outlined" style="font-size: 2rem;">sports_esports</span>
      </button>
      <button class="sidebar-btn" style="background-color: red;">
        <span>Player 2</span>
        <span class="material-symbols-outlined" style="font-size: 2rem;">sports_esports</span>
      </button>
      <button class="sidebar-btn" style="background-color: green;">
        <span>Player 3</span>
        <span class="material-symbols-outlined" style="font-size: 2rem;">sports_esports</span>
      </button>
      <button class="sidebar-btn" style="background-color: blue;">
        <span>Player 4</span>
        <span class="material-symbols-outlined" style="font-size: 2rem;">sports_esports</span>
      </button>
    </div>
  </main>
  <Gamepad></Gamepad>
</template>

<style scoped>
main {
  padding: 1rem 0 1rem 0;
  display: flex;
  gap: .5rem;
  background: var(--mainColor);
  height: 100vh;
}
.left-sidebar {
  height: 100%;
  margin-right: auto;
  display: grid;
  gap: 1rem;
  justify-content: space-around;
}
.right-sidebar {
  height: 100%;
  margin-left: auto;
  display: grid;
  gap: 1rem;
  justify-content: space-around;
}
.sidebar-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  padding: .5rem;
  white-space: nowrap;
  cursor: pointer;
  border: 1px solid black;
  transition: all .2s;
  overflow: hidden;
}
.sidebar-btn.disabled {
  background-color: gray !important;
}
.left-sidebar .sidebar-btn {
  border-radius: 0 1rem 1rem 0;
}
.right-sidebar .sidebar-btn {
  border-radius: 1rem 0 0 1rem;
}
.left-sidebar .sidebar-btn:hover {
  background-color: violet !important;
}

#screenContainer {
  width: 100%;
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
#screenContainer .videoOverlay {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0; 
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
}
.playBtn {
  
}
</style>

<script setup lang="ts">
import '../components/Gamepad.vue'
import { ref, computed } from "vue"
import { useRoute } from 'vue-router'
const route = useRoute()

import { Peer, DataConnection } from "peerjs"

// Query params
const hostID = route.params.id as string | null

// References of elements
const screen = ref<HTMLVideoElement | null>(null)

// Main code
let shareURL = ""
const connected: DataConnection[] = []
let mediaStreamCalls: any[] = []
let videoStreamSource = ref<MediaStream | null>(null)

const peer = new Peer()
let conn: DataConnection | null = null // keeps connection ref if we are clients
peer.on('open',function(id) {
  if (!hostID) {
    // We are the master, listen for calls
    shareURL = window.location.origin + '/room/' + id
    console.log('Join via link: ' + shareURL)
    peer.on('connection', function(conn) {
      conn.on('open', function() {
        conn.send(`welcome ${conn.peer} !`)
        if(videoStreamSource.value) {
            mediaStreamCalls.push(peer.call(conn.peer, videoStreamSource.value))
        }
        connected.push(conn) // Add to connected list
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
    conn = peer.connect(hostID)
    conn.on('open', function() {
      // Send messages
      conn!.send('Hello!')
      
      // Receive messages
      conn!.on('data', function(data) {
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
        // If video stream ended, nullify source
        remoteStream.getVideoTracks()[0].addEventListener('ended', () => {
          videoStreamSource.value = null
        })
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
    for (let conn of connected) {
      mediaStreamCalls.push(peer.call(conn.peer, stream))
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

function copyShareLink() {
  navigator.clipboard.writeText(shareURL)
}

</script>
