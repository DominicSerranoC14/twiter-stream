<template>
    <div>
        <div class="jumbotron">
            <h2>About</h2>

            <p class="lead">Twitter Stream is a full stack JavaScript application using the Twitter streaming API to filter custom tweets selected by a user. A user can filter tweets based on selected languages, certain words or phrases, hashtags, user accounts, locations and other options. Check out <a href="https://twitter.com/search-advanced" target="_blank">Twitter's advanced search</a> for a full example of these search parameters.</p>

            <h4>Pick your stream options</h4>

            <div class="row">
                <div class="form-group col-lg-6 col-md-6 col-sm-12">
                    <label for="language">Language</label>
                    <select class="form-control" name="language" id="language" v-model="language">
                        <option value="">---</option>
                        <option value="all">All</option>
                        <option value="en">English</option>
                        <option value="es">Spanish</option>
                        <option value="fr">French</option>
                    </select>
                </div>

                <div class="form-group col-lg-6 col-md-6 col-sm-12">
                    <label for="track">Track</label>
                    <input type="text" name="track" id="track" placeholder="Words or phrases to track"
                        class="form-control" v-model="trackString" />
                </div>

                <div class="col-lg-12 text-center">
                    <button class="btn btn-primary" @click="saveOptions" :disabled="canSaveOptions">Save Settings</button>
                </div>
            </div>
        </div>

        <div class="row marketing">
            <h2 class="col-lg-12">About Stream Options</h2>

            <div class="col-lg-6">
                <h4>Track</h4>
                <p>Keywords to track. Phrases of keywords are specified by a comma-separated list.</p>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        saveStreamSettings: { required: true, type: Function },
    },

    data() {
        return {
            language: '',
            trackString: '',
        };
    },

    computed: {
        canSaveOptions() {
            return (!this.language || !this.trackString);
        },
    },

    methods: {
        formatOptions() {
            return {
                language: this.language,
                track: this.trackString,
            };
        },

        saveOptions() {
            this.saveStreamSettings(this.formatOptions());
        },
    },
}
</script>

<style lang="scss" scoped>
</style>
