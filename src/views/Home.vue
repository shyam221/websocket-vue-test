<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld @sendMsg="send" @disconnect="disconnect" @connect="connect" msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import SockJS from 'sockjs-client'
import Stomp from 'webstomp-client'
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    HelloWorld
  },
  data () {
    return {
      received_messages: [],
      send_message: null,
      connected: false
    }
  },
  methods: {
    send () {
      if (this.stompClient && this.stompClient.connected) {
        const msg = {
          nik: '',
          nama: '',
          tanggalLahir: '',
          jenisPembiayaan: ''
        }
        this.stompClient.send('/app/inisiasi/konsumen', JSON.stringify(msg), {})
      }
    },
    connect () {
      const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiMjAxNjA1MzUiLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJVc2VycyIsImV4cCI6MTY2NTk3ODM0NCwiaXNzIjoiaHR0cHM6Ly9sb2NhbGhvc3Q6NTAwMSIsImF1ZCI6Imh0dHBzOi8vbG9jYWxob3N0OjUwMDEifQ.0tofHm_1zTn5ZP01N0UFuKX04iNtn9f6xjp5H3b3VCs'
      const sessionId = Math.random().toString(36).substring(3, 9)
      this.socket = new SockJS('http://localhost:8020/ws/connect?authorization=' + token, [], {
        sessionId: () => { return sessionId }
      })
      console.log(this.socket)
      this.stompClient = Stomp.over(this.socket)
      console.log(this.stompClient)
      this.stompClient.connect(
        { },
        (frame) => {
          this.connected = true
          console.log(frame)
          this.stompClient.subscribe('/user/queue/inisiasi/failed' + '-user' + sessionId, (tick) => {
            console.log(tick)
          })
        },
        (error) => {
          console.log(error)
          this.connected = false
        }
      )
    },
    disconnect () {
      if (this.stompClient) {
        this.stompClient.disconnect()
      }
      this.connected = false
    },
    tickleConnection () {
      this.connected ? this.disconnect() : this.connect()
    }
  },
  mounted () {
    // this.connect()
  }
}
</script>
