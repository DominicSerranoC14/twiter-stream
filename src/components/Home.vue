<template>
    <div class="">
        <div class="header">
            <h1>Live Tweets | Total: {{ statusList.length }}</h1>

            <button :disabled="connectingStream || streaming" @click="startStream">Start Stream</button>

            <div class="filter-input">
                <p>Search by Name, Username or Tweet Text</p>
                <input type="text" placeholder="Filter" v-model="filterString" />
                <p v-if="filterString"> Filter Results: {{ filterArray.length }}</p>
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
            socket: null,
            streaming: false,
            connectingStream: false,
            filterString: '',
            statusList: [],
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
        }
    },

    methods: {
        formatStatusData(status) {
            this.statusList.unshift(status);
        },

        startStream() {
            this.socket.emit('start-stream');
        },
    },

    mounted() {
        this.socket = io('https://stream-manager.herokuapp.com/');

        this.socket.on('connect', () => {
            this.socket.emit('register-worker');
        });

        this.socket.on('stream-active', (active) => {
            this.streaming = active;
        });

        this.socket.on('received-status', (data) => {
            this.formatStatusData(data);
        });

        this.socket.on('stream-closed', () => {
            alert('The stream has closed, try to start a new one.');
        });

        this.socket.on('on-error', (msg) => {
            console.error(msg);
        });
    }
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
