<template>
    <div class="container">
        <div class="header clearfix">
            <h3 class="text-muted">
                Twitter Stream
                <span v-if="mode === 'stream'">
                    | Total: {{ statusList.length }}
                </span>
            </h3>
        </div>

        <options-menu v-if="mode === 'options'"
            :save-stream-settings="saveStreamSettings">
        </options-menu>

        <div class="jumbotron" v-if="mode === 'stream'">
            <div class="text-center">
                <button class="btn btn-primary" :disabled="canStartStream" @click="startStream">Start Stream</button>

                <button class="btn btn-danger" @click="stopStream" :disabled="!streaming">Stop Stream</button>
            </div>

            <div class="filter-input text-center mt-4">
                <p>Search by Name, Username or Tweet Text</p>
                <input type="text" placeholder="Filter" v-model="filterString" />
                <p v-if="filterString"> Filter Results: {{ filterArray.length }}</p>
            </div>
        </div>

        <div class="tweet-container">
            <tweet v-for="(status, i) in filterArray" :key="i" :status="status"></tweet>
        </div>

        <footer class="footer">
            <p>Built with Vue.js, Node.js and Socket.io</p>
        </footer>
    </div>
</template>

<script>
export default {
    components: {
        'options-menu': require('./Options.vue'),
        'tweet': require('./Tweet.vue'),
    },

    data () {
        return {
            connectingStream: false,
            filterString: '',
            mode: 'options',
            socket: null,
            statusList: [],
            streaming: false,
        }
    },

    computed: {
        canStartStream() {
            return (this.connectingStream || this.streaming);
        },

        filterArray() {
            return this.statusList.filter(status => {
                const { displayText, name, screen_name, status_id } = status;
                const keyword = this.filterString.toLowerCase();

                if (displayText.toLowerCase().includes(keyword)) {
                    return status;
                } else if (name.toLowerCase().includes(keyword)) {
                    return status;
                } else if (screen_name.toLowerCase().includes(keyword)) {
                    return status;
                } else if (status_id.includes(keyword)) {
                    return status;
                }
            });
        },
    },

    methods: {
        formatStatusData(status) {
            this.statusList.unshift(status);
        },

        saveStreamSettings(options) {
            this.socket.emit('set-options', options);
            this.mode = 'stream';
        },

        startStream() {
            this.socket.emit('start-stream');
        },

        stopStream() {
            this.socket.emit('stop-stream');
        },
    },

    mounted() {
        this.socket = io('https://stream-manager.herokuapp.com/');

        this.socket.on('connect', () => {
            this.socket.emit('register-worker');
        });

        this.socket.on('stream-active', (active) => {
            this.streaming = active;

            // If a new client connects, and a stream is active, this will put them in the stream viewing mode
            if (this.mode === 'options' && this.streaming) {
                this.mode = 'stream';
            }
        });

        this.socket.on('received-status', (data) => {
            this.formatStatusData(data);
        });

        this.socket.on('stream-closed', () => {
            // Only execute the following code once. This may be firing for every socket connected to a room
            this.mode = 'options';
            this.statusList = [];
            alert('The stream has closed, try to start a new one.');
        });

        this.socket.on('user-error', (msg) => {
            alert(msg);
        });

        this.socket.on('on-error', (msg) => {
            console.error(msg);
        });
    },

    beforeDestroy() {
        // Disconnects the client socket when the component unmounts to avoid multiple sockets still being attached to the server and duplicating events.
        this.socket.disconnect();
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.header {
    text-align: center;
    margin-bottom: 20px;

    button {
        font-size: 14px;
        padding: 5px 10px;
    }

    .filter-input {
        margin-top: 10px;

        input {
            font-size: 14px;
            padding: 5px;
        }
    }
}

.tweet-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
}

.spread {
    margin-bottom: 20px;
}

h1, h2 {
    font-weight: normal;
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
