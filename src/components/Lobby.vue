<template>
    <div>
        chats
        <input type="text" v-model="userId">
        <div v-for="r in rooms" v-bind:key="r.namespace">
            <div>{{ r.name }}</div>
            <button @click="join(r)">
                Join Room
            </button>
        </div>
    <div v-if="socketio != null">
        <HelloWorld
            :theSocket="socketio"
            :userId="userId"/>
    </div>
    </div>
</template>

<script>
import axios from 'axios'
import HelloWorld from './HelloWorld.vue'
export default {
    name: 'Lobby',
    components: {
        HelloWorld
    },
    data() {
        return {
            userId: null,
            rooms: [],
            socketio: null
        }
    },
    methods: {
        getRooms: function () {
        return axios.get('http://localhost:8100/api/videorooms')
            .then((res) => {
            this.rooms = res.data.data.videorooms || []
            return
            })
        },
        join: function (r) {
            console.log(r)
            this.socketio = r
        }
    },
    created: function () {
        this.getRooms()
    }
}
</script>