<template>
    
    <v-card id="audioPlayer" elevation="0">

        <div class="flex">
            
            <v-img :src=trackCover class="trackCover" />

            <div style="display: block">
                <div v-text="trackTitle" class="noSelect" />
                <div v-text="trackAuthor" class="trackAuthor noSelect" item-disabled />

                <div class="flex">
                    <v-btn text class="audioAction" v-if="audioPaused" @click="audioPlay()"><v-icon>mdi-play</v-icon></v-btn>
                    <v-btn text class="audioAction" v-if="!audioPaused" @click="audioPause()"><v-icon>mdi-pause</v-icon></v-btn>
                </div>

            </div>
        </div>

        <v-progress-linear class="audioProgress" :value=audioPositionPercent />

        <div class="flex" style="margin-top: 0.5em;">
            <v-spacer />
            <div>
                {{ new Date(audioPosition * 1000).toISOString().substr(14, 8).split('.')[0] }}
                <span style="color: #999;">
                    /
                    {{ new Date(audioDuration * 1000).toISOString().substr(14, 8).split('.')[0] }}
                </span>
            </div>
        </div>


    </v-card>

</template>

<style scoped>
#audioPlayer {
    padding: 1em;
    border-radius: 0.75em;
    width: 400px;
}

.trackAuthor {
    color: #999
}
.trackCover {
    max-width: 100px;
    max-height: 100px;
    margin-right: 1em;
}

.audioAction {
    min-width: 15px !important;
    min-height: 15px !important;
    padding: 0.5em !important;
}

.audioProgress {
    margin-top: 0.5em;
    border-bottom-left-radius: 1px !important;
    border-bottom-right-radius: 1px !important;
    border-top-left-radius: 1px !important;
    border-top-right-radius: 1px !important;
}

.noSelect {
    -webkit-user-select:none;
    -khtml-user-select:none;
    -moz-user-select:none;
    -ms-user-select:none;
    -o-user-select:none;
    user-select:none;
}

.credits {
    font-size: 0.5em;
    color: #777;
}

.flex {
    display: flex;
}
</style>

<script>
let sound;
export default {
    data() {
        return {
            moduleVersion: "v1.0.0",

            trackTitle:  "Each Other",
            trackAuthor: "Notaker, Eric Lumiere",
            trackCover:  "/images/trackImage.jpg",
            trackPath:   "/audio/Notaker - Each Other (feat. Eric Lumiere) [Monstercat Release].mp3",

            audioProgress: 10,
            audioPaused: true,
            audioPosition: 0,
            audioDuration: 0,
            audioPositionPercent: 0,
        }
    },

    mounted() {

        sound      = document.createElement('audio');
        sound.src  = this.trackPath;
        sound.type = 'audio/mpeg';
        document.body.appendChild(sound);

        setInterval(() => {
            this.audioPosition = sound.currentTime;
            this.audioDuration = sound.duration;
            this.audioPositionPercent = (100 * sound.currentTime) / sound.duration;

            if (this.audioPositionPercent >= 100) {
                sound.currentTime = 0;
                this.audioPause();
            }
        }, 250)

    },

    methods: {

        audioPlay() {
            sound.play();
            this.audioPaused = false;
        },
        audioPause() {
            sound.pause();
            this.audioPaused = true;
        }

    }

}
</script>
