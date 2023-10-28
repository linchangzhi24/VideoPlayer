<template>
    <div class="layout-header"></div>
    <div class="head_placeholder"></div>
    <div class="content">
        <div class="left-container">
            <div class="video-container">
                <video id="video" :src="videoPath" @dblclick.stop="autoFullscreen()"></video>
                <div class="player-control-box" :class="{ show: ctrlDisplay }">
                    <!-- progress bar -->
                    <div class="player-control-top">
                        <div class="player-progress-schedule" @click="progressJumpTo($event)">
                            <div class="player-progress-buffer" style="transform: scaleX(1);"></div>
                            <div class="player-progress-current"
                                :style="{ transform: `scaleX(${duration != 0 ? currentTime / duration : 0})` }">
                            </div>
                        </div>
                    </div>
                    <!-- control -->
                    <div class="player-control-bottom">
                        <div class="player-control-area">
                            <div class="player-ctrl-left">
                                <!-- play pause -->
                                <div class="player-ctrl-btn">
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-player-paused.svg" v-show="videoPaused"
                                            @click.stop="videoPlay()" />
                                    </Transition>
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-player-playing.svg" v-show="!videoPaused"
                                            @click.stop="videoPause()" />
                                    </Transition>
                                </div>
                                <!-- time -->
                                <div class="player-ctrl-btn player-ctrl-time">
                                    <span>{{ formatSeconds(currentTime) }}</span>
                                    <span>/</span>
                                    <span>{{ formatSeconds(duration) }}</span>
                                </div>
                            </div>
                            <div class="player-ctrl-right">
                                <!-- volume -->
                                <div class="player-ctrl-btn player-ctrl-volume-icon">
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-player-volume.svg"
                                            v-show="currentVolume > 0" />
                                    </Transition>
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-player-volume-mute.svg"
                                            v-show="currentVolume == 0" />
                                    </Transition>
                                    <!-- volume progress bar -->
                                    <div class="player-ctrl-volume-box">
                                        <div class="player-ctrl-volume-number">
                                            {{ Math.round(currentVolume * 10000) / 100 }}
                                        </div>
                                        <div id="volumeProgressBar" class="player-ctrl-volume-progress-area"
                                            @mousedown="dotMousedown($event)" @mouseup="clearDotMousemove($event)">
                                            <div class="player-ctrl-volume-progress">
                                                <div class="player-ctrl-volume-progress-bar player-ctrl-volume-progress-schedule"
                                                    style="transform: scaleY(1);"></div>
                                                <div class="player-ctrl-volume-progress-bar player-ctrl-volume-progress-current"
                                                    :style="{ transform: `scaleY(${currentVolume})` }"></div>
                                                <div class="thumb-dot-box"
                                                    :style="{ transform: `translateY(${(1 - currentVolume) * 100 + 'px'})` }">
                                                    <div class="thumb-dot"></div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <!-- fullscreen -->
                                <div class="player-ctrl-btn">
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-exit-full-screen.svg"
                                            v-show="fullscreenState" @click.stop="exitFullscreen()" />
                                    </Transition>
                                    <Transition name="slide-fade">
                                        <img height="22" src="../assets/icons/icon-full-screen.svg"
                                            v-show="!fullscreenState" @click.stop="enterFullscreen()" />
                                    </Transition>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- loading -->
                <div class="mask-container" v-show="isloading">
                    <div class="loader">
                        <div data-glitch="Loading..." class="glitch">Loading...</div>
                    </div>
                </div>
            </div>
        </div>
        <div class="right-container">
            <div class="catalog-container">
                <div class="catalog-list">
                    <div class="catalog-col-box" v-for="(file, index) in files" @click="setVideoSource(index)"
                        v-bind:key="file"
                        :class="[{ 'catalog-col-seleted': playIndex == index }, file.name.length > 7 ? 'catalog-col-box-wide' : 'catalog-col-box-narrow']">
                        <div class="catalog-col">
                            {{ file.name.length > 7 ? file.name : file.name.split('.')[0] }}
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="container">
        <label class="neon-btn">
            <span class="span">
                <div data-glitch="打开视频..." class="glitch">打开视频...</div>
            </span>
            <input class="input-btn" type="file" @change="selectFolder($event)" multiple>
        </label>
    </div>
</template>

<script>
import { ref } from 'vue'

export default {
    setup() {
        const videoPath = ref('')
        const duration = ref(0)
        const currentTime = ref(0)
        const currentVolume = ref('')
        const videoPaused = ref(true)
        const fullscreenState = ref(false)
        const ctrlDisplay = ref(false)
        const files = ref(new Array())
        const playIndex = ref(null)
        const canplay = ref(false)
        const isloading = ref(false)

        return {
            videoPath,
            duration,
            currentTime,
            currentVolume,
            videoPaused,
            fullscreenState,
            ctrlDisplay,
            files,
            playIndex,
            canplay,
            isloading,
        }
    },
    methods: {
        playerEvents() {
            const vm = this

            let fadeTimer

            const video = document.querySelector('#video')

            this.currentVolume = video.volume

            video.ondurationchange = (event) => {
                this.duration = video.duration
            }
            video.ontimeupdate = (event) => {
                this.currentTime = video.currentTime
            }
            video.onplaying = (event) => {
                this.videoPaused = false
            }
            video.onpause = (event) => {
                this.videoPaused = true
            }
            video.onvolumechange = (event) => {
                this.currentVolume = video.volume
            }
            video.onmousemove = (event) => {
                this.ctrlDisplay = true
                const vm = this
                clearTimeout(fadeTimer)
                fadeTimer = setTimeout(() => {
                    vm.ctrlDisplay = false
                }, 1000)
            }
            video.oncanplay = (event) => {
                this.canplay = true
                this.isloading = false
                this.videoPlay()
            };
        },
        documentEvents() {
            const video = document.querySelector('#video')

            const volumeChangeSize = 0.01
            const timeChangeSize = 5

            document.onkeydown = (event) => {
                const e = event || window.event || arguments.callee.caller.arguments[0]

                if (e && e.keyCode === 38) {
                    // ↑
                    1 - video.volume >= volumeChangeSize ? video.volume += volumeChangeSize : video.volume = 1
                    return false
                } else if (e && e.keyCode === 40) {
                    // ↓
                    video.volume >= volumeChangeSize ? video.volume -= volumeChangeSize : video.volume = 0
                    return false
                } else if (e && e.keyCode === 37) {
                    // ←
                    video.currentTime !== 0 ? video.currentTime -= timeChangeSize : 1
                    return false
                } else if (e && e.keyCode === 39) {
                    // →
                    video.currentTime !== video.duration ? video.currentTime += timeChangeSize : 1
                    return false
                } else if (e && e.keyCode === 32) {
                    // space
                    video.paused === true ? this.videoPlay() : this.videoPause()
                    return false
                } else if (e && e.keyCode === 80) {
                    // p
                    this.screenshot()
                    return false
                }
            }

            document.onfullscreenchange = (event) => {
                this.fullscreenState = document.fullscreen
            }
        },
        enterFullscreen() {
            const videoContainer = document.querySelector('.video-container')
            videoContainer.requestFullscreen()
        },
        exitFullscreen() {
            document.exitFullscreen()
        },
        autoFullscreen() {
            document.fullscreen ? this.exitFullscreen() : this.enterFullscreen()
        },
        videoPlay() {
            const video = document.querySelector('#video')
            if (this.canplay) {
                video.play()
            }
        },
        videoPause() {
            const video = document.querySelector('#video')
            video.pause()
        },
        progressJumpTo(event) {
            const video = document.querySelector('#video')
            video.currentTime = event.offsetX / event.currentTarget.offsetWidth * this.duration
        },
        dotMousedown(event) {
            const volumeProgressBar = document.querySelector('#volumeProgressBar')

            const volume = (1 - event.offsetY / volumeProgressBar.offsetHeight).toFixed(2)
            this.setVideoVolume(volume)

            volumeProgressBar.onmousemove = (event) => {
                const volume = (1 - event.offsetY / volumeProgressBar.offsetHeight).toFixed(2)
                this.setVideoVolume(volume)
            }
            volumeProgressBar.onmouseleave = (event) => {
                this.clearDotMousemove(event)
            }
        },
        clearDotMousemove(event) {
            const volumeProgressBar = document.querySelector('#volumeProgressBar')
            volumeProgressBar.onmousemove = null
            volumeProgressBar.onmouseleave = null

            const volume = (1 - event.offsetY / volumeProgressBar.offsetHeight).toFixed(2)
            this.setVideoVolume(volume)
        },
        setVideoVolume(volume) {
            const video = document.querySelector('#video')
            if (volume > 1) {
                volume = 1
            } else if (volume < 0) {
                volume = 0
            }
            video.volume = volume
        },
        formatSeconds(seconds) {
            seconds = parseInt(seconds)
            let hour = Math.floor(seconds / 3600) >= 10 ? Math.floor(seconds / 3600) : '0' + Math.floor(seconds / 3600)
            seconds -= 3600 * hour;
            let min = Math.floor(seconds / 60) >= 10 ? Math.floor(seconds / 60) : '0' + Math.floor(seconds / 60)
            seconds -= 60 * min;
            let sec = seconds >= 10 ? seconds : '0' + seconds

            let result = ''
            if (hour != '00') {
                result = hour + ':' + min + ':' + sec
            } else {
                result = min + ':' + sec
            }

            return result
        },
        selectFolder(event) {
            if (event.target.files.length > 0) {
                this.clearVideoSource()

                var fileList = event.target.files;

                this.files = new Array()

                for (let file of fileList) {
                    this.files.push(file)
                }

                this.files.sort(
                    (a, b) => {
                        return a.name.split('.')[0] - b.name.split('.')[0]
                    }
                )
            }
        },
        setVideoSource(i) {
            if (this.playIndex == i) return

            this.clearVideoSource()

            this.isloading = true
            this.playIndex = i

            URL.revokeObjectURL(this.videoPath);
            const objectURL = URL.createObjectURL(this.files[i]);
            this.videoPath = objectURL
        },
        clearVideoSource() {
            this.videoPause()
            document.querySelector('#video').src = ''
            this.videoPath = ''
            this.videoPaused = true
            this.duration = 0
            this.currentTime = 0
            this.canplay = false
            this.playIndex = null
        },
        screenshot() {
            const canvas = document.createElement('canvas')
            const ctx = canvas.getContext('2d')

            const ratio = window.devicePixelRatio

            const video = document.querySelector('#video')
            canvas.width = video.videoWidth * ratio
            canvas.height = video.videoHeight * ratio

            ctx.drawImage(video, 0, 0, canvas.width, canvas.height)

            const imgName = 'screenshot_' + this.files[this.playIndex].name.replaceAll('.', '_') + '-' + Number.parseInt(this.currentTime)
            this.downloadImage(canvas, imgName)
        },
        downloadImage(canvas, name) {
            let image = new Image();
            image.setAttribute("crossOrigin", "anonymous");

            let imgUrl = canvas.toDataURL("image/png");
            image.src = imgUrl;

            let a = document.createElement("a");
            a.download = name || "screenshot_" + new Date().getTime();
            a.href = imgUrl;
            a.setAttribute("id", "link");

            let event = new MouseEvent("click");
            a.dispatchEvent(event);
        },
    },
    mounted() {
        this.playerEvents()
        this.documentEvents()
    },
}
</script>

<style scoped src="../assets/css/video-player.css"></style>