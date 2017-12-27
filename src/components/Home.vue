<template>
    <div>
        <div class="header">
            <h3 class="text-muted">Twitter Stream</h3>

            <button class="btn btn-primary"
                :disabled="streaming" v-if="mode === 'stream'"
                @click="changeMode('options')">Options Menu
            </button>
        </div>

        <options-menu v-show="mode === 'options'"
            :language="language"
            :save-stream-settings="saveStreamSettings"
            :set-language="setLanguage"
            :set-track-string="setTrackString"
            :track-string="trackString">
        </options-menu>

        <stream-menu v-if="mode === 'stream'"
            :connecting-stream="connectingStream"
            :created-at="createdAt"
            :filter-array="filterArray"
            :filter-string="filterString"
            :language="language"
            :start-stream="startStream"
            :set-filter-string="setFilterString"
            :status-list-length="statusListLength"
            :stop-stream="stopStream"
            :streaming="streaming"
            :track-string="trackString">
        </stream-menu>    

        <div class="tweet-container" v-show="mode === 'stream'">
            <tweet v-for="(status, i) in filterArray" :key="i" :status="status"></tweet>
        </div>

        <footer class="container footer">
            <p>Built with Vue.js, Node.js and Socket.io by
                <a href="https://dominicserrano.com">Dominic Serrano</a>
            </p>
        </footer>
    </div>
</template>

<script>
export default {
    components: {
        'options-menu': require('./Options.vue'),
        'stream-menu': require('./Stream.vue'),
        'tweet': require('./Tweet.vue'),
    },

    data () {
        return {
            // Move all data values to vuex store eventually
            connectingStream: false,
            createdAt: '',
            filterString: '',
            language: null,
            mode: 'options',
            socket: null,
            statusList: [],
            streaming: false,
            trackString: '',
        }
    },

    computed: {
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

        statusListLength() {
            return this.statusList.length;
        }
    },

    methods: {
        changeMode(mode) {
            this.mode = mode;
        },

        formatStatusData(status) {
            this.statusList.push(status);
        },

        saveStreamSettings(options) {
            this.socket.emit('set-options', options);
            this.mode = 'stream';
        },

        setFilterString(event) {
            this.filterString = event.target.value;
        },

        setLanguage(language) {            
            this.language = language;
        },

        setTrackString(event) {
            this.trackString = event.target.value;            
        },

        setStreamOptions(options) {
            this.language = options.language;
            this.trackString = options.track;
            this.createdAt = options.createdAt;
        },

        startStream() {
            // Clear list if user had a stream previously open
            this.statusList = [];
            this.socket.emit('start-stream');
        },

        stopStream() {
            this.socket.emit('stop-stream');
        },
    },

    mounted() {
        this.socket = io('https://stream-manager.herokuapp.com/');
        // this.socket = io('localhost:3000');

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

        this.socket.on('stream-options', this.setStreamOptions);

        this.socket.on('received-status', (data) => {
            this.formatStatusData(data);
        });

        this.socket.on('stream-closed', () => {
            // Only execute the following code once. This may be firing for every socket connected to a room
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
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    align-items: center;
    margin-bottom: 20px;

    h3 { margin: 0; }
}

.tweet-container {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
}
</style>
