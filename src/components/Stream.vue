<template>
    <div class="stream-container">
        <div class="jumbotron flex-one mr-2">
            <h4>Stream Stats</h4>

            <p>State: {{ (streaming) ? 'Streaming' : 'Not Streaming' }}</p>

            <stream-timer
                :total-seconds="totalSeconds">
            </stream-timer>

            <p>Number of Tweets: {{ statusListLength }}</p>

            <p>Language: {{ language }}</p>
            
            <p>Track String: {{ trackString }}</p>
        </div>

        <div class="jumbotron flex-three">
            <div class="text-center">
                <button class="btn btn-primary" :disabled="canStartStream" @click="startStreamHandler">Start Stream</button>

                <button class="btn btn-danger" @click="stopStreamHandler" :disabled="!streaming">Stop Stream</button>
            </div>

            <div class="form-group text-center mt-4 filter-container">
                <p>Search by Name, Username or Tweet Text</p>
                
                <input type="text" placeholder="Filter" class="form-control" 
                    :value="filterString" @input="setFilterString" />
                
                <p v-if="filterString"> Filter Results: {{ filterArray.length }}</p>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        connectingStream: { required: true, type: Boolean },
        createdAt: { required: true },
        filterArray: { required: true, type: Array },
        filterString: { required: true, type: String },
        language: { required: true },
        statusListLength: { required: true, type: Number },
        setFilterString: { required: true, type: Function },
        startStream: { required: true, type: Function },
        stopStream: { required: true, type: Function },
        streaming: { required: true, type: Boolean },
        trackString: { required: true, type: String },
    },

    components: {
        streamTimer: require('./StreamTimer.vue'),
    },

    data() {
        return {
            totalSeconds: 0,
            streamInterval: null,
        }
    },

    computed: {
        canStartStream() {
            return (this.connectingStream || this.streaming);
        },
    },

    methods: {
        getDurationDiff() {
            // Calculate the time that has elapsed since the stream was created and the client connected to the stream
            this.totalSeconds = Math.floor((new Date() - new Date(this.createdAt)) / 1000);
        },

        startStreamHandler() {
            this.startStream();
            this.startTimer();
        },

        startTimer() {
            this.streamInterval = setInterval(() => ++this.totalSeconds, 1000);
        },

        stopStreamHandler() {
            this.stopStream();
            this.stopTimer();
        },

        stopTimer() {
            this.totalSeconds = 0;
            clearInterval(this.streamInterval);
        },
    },

    mounted() {
        // If a new user joins the stream, start the timer and update seconds.
        // TODO get the start time from the stream. For now each user will have a different start time
        if (this.streaming) {
            this.getDurationDiff();
            this.startTimer();
        }
    }
}
</script>

<style lang="scss" scoped>
.stream-container {
    display: flex;
    flex-direction: row;
    width: 90%;
    margin: auto;

    .flex-one { flex: 1; }
    .flex-three { flex: 3; }
}

.filter-container {
    width: 60%;
    margin: auto;
}
</style>