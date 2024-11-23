<template>
    

    <div class="card bg-base-100 w-96 shadow-xl px-4 py-6" :style="activeLyric == '%music' ? `outline: 1px solid rgba(255,255,255,${nbi})` : 'outline: 1px solid rgba(255,255,255,0.5)'">
        <div class="img-backdrop" style="z-index: 1; backdrop-filter: blur(2em);" />
        <div class="img-backdrop" style="z-index: 0;" :style="`background-image: url('${cover}');`" />
        <div class="flex" style="z-index: 2;">
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
        <div v-if="lyricsEnabled" style="z-index: 2;">
            <p class="gray">Lyrics</p>
            <div v-if="activeLyric == '%music'" class="flex">
                <svg v-for="_ in 3" class="w-4s h-4 invert mr-2 gray_filter" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M499.1 6.3c8.1 6 12.9 15.6 12.9 25.7l0 72 0 264c0 44.2-43 80-96 80s-96-35.8-96-80s43-80 96-80c11.2 0 22 1.6 32 4.6L448 147 192 223.8 192 432c0 44.2-43 80-96 80s-96-35.8-96-80s43-80 96-80c11.2 0 22 1.6 32 4.6L128 200l0-72c0-14.1 9.3-26.6 22.8-30.7l320-96c9.7-2.9 20.2-1.1 28.3 5z"/></svg>
            </div>
            <p   v-else class="gray">{{ activeLyric }}</p>
        </div>
    </div>

</template>

<style scoped>
.img-backdrop {
    width: 100%; height: 100%; position: absolute; margin: -1em; margin-top: -1.5em; border-radius: 1em;
}

#audioPlayer {
    padding: 1em;
    border-radius: 0.75em;
    width: 400px;
}

.gray {
    color: #999
}
.gray_filter {
    filter: brightness(0) saturate(100%) invert(65%) sepia(15%) saturate(7%) hue-rotate(323deg) brightness(90%) contrast(97%);
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
export default {
    props: ['title', 'author', 'cover', 'audio', 'lyrics', 'debug'],
    data() {
        return {
            moduleVersion: "v2.1.0",
            tracks: new Object(),
            audioProgress: 10,
            audioPaused: true,
            audioPosition: 0,
            audioDuration: 0,
            audioPositionPercent: 0,
            lyricsEnabled: false,
            activeLyric: "",
            nbi: 0,
        }
    },

    async mounted() {
        const main_track = this.make_track('main');

        //---   Preload Lyrics   ---//
        let lyrics = await $fetch(this.lyrics);
        if (lyrics) {
            try {
                lyrics = JSON.parse(lyrics);
            } catch (e) {}
            this.lyricsEnabled = true;
        }

        //---   Set Up Bass Track   ---//
        const bass_analysis_track = this.make_track('bass_analysis')
        const audioCtx = new AudioContext();
        const analyser = audioCtx.createAnalyser();
        const source = audioCtx.createMediaElementSource(bass_analysis_track);
        source.connect(analyser);


        setInterval(() => {
            //---   Progress Bar   ---//
            this.audioPosition = main_track.currentTime;
            this.audioDuration = main_track.duration;
            this.audioPositionPercent = (100 * main_track.currentTime) / main_track.duration;

            //---   Lyric Sync   ---//
            const lyric_time = Math.round(main_track.currentTime);
            if (this.debug == "true") console.log("Lyric Time:", lyric_time);
            for (const i in lyrics) {
                let lyric = lyrics[i];
                if (lyric[0] == lyric_time) {
                    this.activeLyric = lyric[1];
                }
            }

            //---   Calcualte Bass Intensity   ---//
//            if (this.activeLyric == "%music") {
//                const frequencyData = new Uint8Array(analyser.frequencyBinCount);
//                analyser.getByteFrequencyData(frequencyData);
//                const bassRange = Math.floor(analyser.frequencyBinCount * 0.10);
//                const trebleRange = Math.floor(analyser.frequencyBinCount * 0.90);
//                let totalBassEnergy = 0;
//                let totalTrebleEnergy = 0;
//                for (let i = 0; i < bassRange; i++) {
//                    const energy = frequencyData[i];
//                    totalBassEnergy += energy;
//                }
//                for (let i = 0; i < trebleRange; i++) {
//                    const energy = frequencyData[i];
//                    totalTrebleEnergy  += energy;
//                }
//                const normalizedBassIntensity = totalBassEnergy / (bassRange * 256); // Normalize the bass energy to a value between 0 and 1
//                const normalizedTrebleIntensity = totalTrebleEnergy / (trebleRange * 256);
//                this.nbi = normalizedTrebleIntensity;
//                console.log(normalizedBassIntensity, normalizedTrebleIntensity);
//            }

            //---   Stop At EOF   ---//
            if (this.audioPositionPercent >= 100) {
                main_track.currentTime = 0;
                this.audioPause();
            }
        }, 250);

    },

    methods: {
        make_track(name) {
            const new_track = document.createElement('audio');
            new_track.src = this.audio;
            new_track.type = 'audio/mpeg';
            document.body.appendChild(new_track);
            this.tracks[name] = new_track;
            return new_track;
        },

        audioPlay() {
            for (const id in this.tracks) {
                this.tracks[id].play();
            }
            this.audioPaused = false;
        },
        audioPause() {
            for (const id in this.tracks) {
                this.tracks[id].pause();
            }
            this.audioPaused = true;
        }

    }

}
</script>
