<template lang="html">
    <div class="tweet">
        <div class="header">
            <img class="profile-image" :src="status.profile_image_url">

            <div class="header-txt">
                <span class="user-name">{{ status.name }}</span>
                <a target="_blank" :href="`https://twitter.com/${status.screen_name}`"><span>@{{ status.screen_name }}</span></a>
            </div>
        </div>

        <!-- <p class="description"><strong>Description:</strong> {{ status.description }}</p> -->

        <div class="display-text">
            <p v-if="status.reply_name" class="reply-text">Replying to
                <a target="_blank" :href="`https://twitter.com/${status.reply_name}`"><span class="reply-name" v-text="`@${status.reply_name}`"></span></a>
            </p>

            <p>{{ status.displayText }}</p>
        </div>

        <template v-if="status.media">
            <div class="media-container">
                <img class="media-image" :src="img.media_url" v-for="(img, i) in status.media" :key="i">
            </div>
        </template>
    </div>
</template>

<script>
export default {
    props: {
        status: { required: true, type: Object },
    },

    // Code to grab the url from the end of a tweet string
    // Just kidding, need to figure out another way, the http url can be in another section of the text, or there can be multiple, etc.
    // m.slice(m.indexOf('http'))
}
</script>

<style lang="scss" scoped>
.tweet {
    width: 450px;
    margin: 10px;
    padding: 10px;
    border-bottom: solid #ccc 1px;

    a {
        text-decoration: none;
        color: inherit;
    }

    a:hover {
        color: #428bca;
    }
}

.header {
    display: flex;
    flex-direction: row;
    align-items: center;
    padding: 0 10px;

    .profile-image {
        margin-right: 10px;
        border: solid #428bca 1px;
        border-radius: 50%;
    }

    .header-txt {
        .user-name {
            margin-right: 5px;
            font-weight: bold;
        }
    }
}

.display-text {
    font-size: 16px;
    padding: 0 10px;
    font-weight: bold;

    .reply-text {
        font-weight: normal;
    }

    .reply-name {
        font-weight: bold;
        color: #428bca;
    }
}


.media-container {
    display: flex;
    flex-direction: row;
    align-items: center;
    padding-bottom: 15px;

    .media-image {
        flex: 1;
        max-width: 100px;
        max-height: 200px;
        margin-left: 10px;
        border: solid #428bca 1px;
        border-radius: 2px;
    }
}
</style>
