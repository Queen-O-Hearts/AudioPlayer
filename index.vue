<template>
    

    <div class="card bg-base-100 w-96 shadow-xl px-4 py-6">
        <div class="flex">
            <img class="noSelect rounded-xl w-24 h-24 mr-4" :src="cover" />
            <div>
                <h2 class="card-title">{{ title }}</h2>
                <p style="margin-top: -10px;">{{ author }}</p>
                <input type="range" min="0" max="100" disabled :value="audioPositionPercent" class="range range-primary" />

                <div class="flex items-right">
                    <div class="flex">
                        <button class="audioAction w-8 h-10 invert" v-if="!audioPaused" @click="audioPause()">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512"><path d="M48 64C21.5 64 0 85.5 0 112L0 400c0 26.5 21.5 48 48 48l32 0c26.5 0 48-21.5 48-48l0-288c0-26.5-21.5-48-48-48L48 64zm192 0c-26.5 0-48 21.5-48 48l0 288c0 26.5 21.5 48 48 48l32 0c26.5 0 48-21.5 48-48l0-288c0-26.5-21.5-48-48-48l-32 0z"/></svg>
                        </button>
                        <button class="audioAction w-8 h-18 invert" v-if="audioPaused" @click="audioPlay()">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512"><path d="M73 39c-14.8-9.1-33.4-9.4-48.5-.9S0 62.6 0 80L0 432c0 17.4 9.4 33.4 24.5 41.9s33.7 8.1 48.5-.9L361 297c14.3-8.7 23-24.2 23-41s-8.7-32.2-23-41L73 39z"/></svg>
                        </button>
                    </div>


                    <span class="w-48 text-right">{{ new Date(audioPosition * 1000).toISOString().substr(14, 8).split('.')[0] }} <span class="gray">/ {{ new Date(audioDuration * 1000).toISOString().substr(14, 8).split('.')[0] }}</span></span>
                </div>
            </div>   
        </div>
        <div v-if="lyricsEnabled">
            <p class="gray">Lyrics</p>
            <p class="gray">{{ activeLyric }}</p>
        </div>
    </div>

</template>

<style scoped>
#audioPlayer {
    padding: 1em;
    border-radius: 0.75em;
    width: 400px;
}

.gray {
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

.flex {
    display: flex;
}
</style>

<script>
let sound;
export default {
    props: ['title', 'author', 'cover', 'audio', 'lyrics'],
    data() {
        return {
            moduleVersion: "v2.0.0",
            audioProgress: 10,
            audioPaused: true,
            audioPosition: 0,
            audioDuration: 0,
            audioPositionPercent: 0,
            lyricsEnabled: false,
            activeLyric: "",
        }
    },

    async mounted() {
        sound      = document.createElement('audio');
        sound.src  = this.audio;
        sound.type = 'audio/mpeg';
        document.body.appendChild(sound);


        let lyrics = await $fetch(this.lyrics);
        if (lyrics) {
            try {
                lyrics = JSON.parse(lyrics);
            } catch (e) {}
            this.lyricsEnabled = true;
        }

        setInterval(() => {
            this.audioPosition = sound.currentTime;
            this.audioDuration = sound.duration;
            this.audioPositionPercent = (100 * sound.currentTime) / sound.duration;

            console.log(Math.round(sound.currentTime));
            for (const i in lyrics) {
                let lyric = lyrics[i];
                if (lyric[0] == Math.round(sound.currentTime)) {
                    this.activeLyric = lyric[1];
                }
            }

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
