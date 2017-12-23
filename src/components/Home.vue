<template>
    <div class="">
        <div class="header">
            <h1>Live Tweets 
                <span v-if="mode === 'stream'">
                    | Total: {{ statusList.length }}
                </span>
            </h1>

            <div class="options-container" v-if="mode === 'options'">
                <div class="spread">
                    <label for="language">Stream Language(s)</label>
                    <input type="text" v-model="language" />
                </div>

                <div class="spread">
                    <label for="track-string">Track</label>
                    <input type="text" v-model="trackString" />
                    <p>* Keywords to track. Phrases of keywords are specified by a comma-separated list.</p>
                </div>

                <div>
                    <button @click="saveStreamSettings" :disabled="canSaveOptions">Save Settings</button>
                </div>
            </div>

            <br>

            <div v-if="mode === 'stream'">
                <div>
                    <button :disabled="canStartStream" @click="startStream">Start Stream</button>

                    <button @click="stopStream" :disabled="!streaming">Stop Stream</button>
                </div>

                <div class="filter-input">
                    <p>Search by Name, Username or Tweet Text</p>
                    <input type="text" placeholder="Filter" v-model="filterString" />
                    <p v-if="filterString"> Filter Results: {{ filterArray.length }}</p>
                </div>
            </div>
        </div>

        <div class="tweet-container">
            <tweet v-for="(status, i) in filterArray" :key="i" :status="status"></tweet>
        </div>
    </div>
</template>

<script>
export default {
    components: {
        'tweet': require('./Tweet.vue'),
    },

    data () {
        return {
            connectingStream: false,
            filterString: '',
            language: '',
            mode: 'options',
            trackString: '',
            socket: null,
            statusList: [],
            streaming: false,
        }
    },

    computed: {
        canSaveOptions() {
            return (!this.language || !this.trackString);
        },

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

        formatOptions() {
            return {
                language: this.language,
                track: this.trackString,
            };
        },

        saveStreamSettings() {
            this.socket.emit('set-options', this.formatOptions());
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
