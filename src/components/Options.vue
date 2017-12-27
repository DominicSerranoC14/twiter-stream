<template>
    <div class="container">
        <div class="jumbotron">
            <h2>About</h2>

            <p class="lead">Twitter Stream is a full stack JavaScript application using the Twitter streaming API to filter custom tweets selected by a user. A user can filter tweets based on selected languages, certain words or phrases, hashtags, user accounts, locations and other options. Check out <a href="https://twitter.com/search-advanced" target="_blank">Twitter's advanced search</a> for a full example of these search parameters.</p>

            <p class="lead">Currently only one stream can be created at a time due to the limitation of the Stream API, but multiple users can view the ongoing stream and it's details.</p>

            <h4>Select your stream options</h4>

            <div class="row">
                <div class="form-group col-lg-6 col-md-6 col-sm-12">
                    <label for="language">Language</label>
                    <select class="form-control" name="language" id="language" 
                        v-model="languageSelect">
                        <option value="null">---</option>
                        <option v-for="(l, i) in languages"
                            :value="l.value" :key="i">{{ l.name }}
                        </option>
                    </select>
                </div>

                <div class="form-group col-lg-6 col-md-6 col-sm-12">
                    <label for="track">Track</label>
                    <input type="text" name="track" id="track" 
                        placeholder="Words or phrases to track"
                        class="form-control" :value="trackString" 
                        @input="setTrackString" @keyup.enter="saveOptions" />
                </div>

                <div class="col-lg-12 text-center">
                    <button class="btn btn-primary" @click="saveOptions" :disabled="!canSaveOptions">Save Settings</button>
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
        language: { required: true },
        setLanguage: { required: true, type: Function },
        saveStreamSettings: { required: true, type: Function },
        setTrackString: { required: true, type: Function },
        trackString: { required: true, type: String },
    },

    data() {
        return {
            languages: [
                { name: 'All', value: '' }, 
                { name: 'English', value: 'en' }, 
                { name: 'Spanish', value: 'es' }, 
                { name: 'French', value: 'fr' }
            ],
        }
    },

    computed: {
        canSaveOptions() {
            return (this.language === null || !this.trackString) ? false : true;
        },

        languageSelect: {
            get: vm => vm.language,
            set(value) {
                this.setLanguage(value);
            }
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
            if (!this.canSaveOptions) return;

            this.saveStreamSettings(this.formatOptions());
        },
    },
}
</script>

<style lang="scss" scoped>
</style>
