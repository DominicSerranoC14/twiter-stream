<template>
    <p>
        Duration of Stream: {{ duration }}
    </p>
</template>

<script>
export default {
    props: {
        totalSeconds: { required: true, type: Number },
    },

    computed: {
        duration() {
            const hours = Math.floor(this.totalSeconds / 3600);
            const minutes = Math.floor((this.totalSeconds - hours * 3600) / 60);
            const seconds = this.totalSeconds - (hours * 3600 + minutes * 60);

            const hoursString = (hours > 9) ? hours : `0${hours}`;
            const minutesString = (minutes > 9) ? minutes : `0${minutes}`;
            const secondsString = (seconds > 9) ? seconds : `0${seconds}`;

            return `${hoursString}:${minutesString}:${secondsString}`;
        },
    }
}
</script>

<style lang="scss" scoped>
</style>