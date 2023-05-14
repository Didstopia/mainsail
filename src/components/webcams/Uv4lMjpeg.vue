<template>
    <div class="d-flex justify-center">
        <img
            ref="webcamUv4lMjpegImage"
            v-observe-visibility="viewportVisibilityChanged"
            :style="webcamStyle"
            class="webcamImage"
            @load="onload" />
    </div>
</template>

<script lang="ts">
import { Component, Mixins, Prop, Watch } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import { GuiWebcamStateWebcam } from '@/store/gui/webcams/types'

@Component
export default class Uv4lMjpeg extends Mixins(BaseMixin) {
    private aspectRatio: null | number = null
    private isVisible = false
    private isVisibleViewport = false
    private isVisibleDocument = true

    @Prop({ required: true }) declare readonly camSettings: GuiWebcamStateWebcam
    @Prop() declare printerUrl: string | undefined

    declare $refs: {
        webcamUv4lMjpegImage: HTMLImageElement
    }

    get url() {
        const baseUrl = this.camSettings.urlStream
        let url = new URL(baseUrl, this.printerUrl === undefined ? this.hostUrl.toString() : this.printerUrl)
        if (baseUrl.startsWith('http') || baseUrl.startsWith('://')) url = new URL(baseUrl)

        return decodeURIComponent(url.toString())
    }

    get webcamStyle() {
        const output = {
            transform: 'none',
            aspectRatio: 16 / 9,
            maxHeight: window.innerHeight - 155 + 'px',
            maxWidth: 'auto',
        }

        let transforms = ''
        if ('flipX' in this.camSettings && this.camSettings.flipX) transforms += ' scaleX(-1)'
        if ('flipX' in this.camSettings && this.camSettings.flipY) transforms += ' scaleY(-1)'
        if (transforms.trimStart().length) output.transform = transforms.trimStart()

        if (this.aspectRatio) {
            output.aspectRatio = this.aspectRatio
            output.maxWidth = (window.innerHeight - 155) * this.aspectRatio + 'px'
        }

        return output
    }

    mounted() {
        document.addEventListener('visibilitychange', this.documentVisibilityChanged)
    }

    beforeDestroy() {
        console.warn('beforeDestroy')

        document.removeEventListener('visibilitychange', this.documentVisibilityChanged)
        this.stopStream()
    }

    startStream() {
        console.warn('Preparing to start stream')

        if (this.isVisible) {
            console.warn('Unable to start stream, already visible')
            return
        }

        if (this.$refs.webcamUv4lMjpegImage) {
            console.warn('Starting stream')
            this.$refs.webcamUv4lMjpegImage.setAttribute('src', this.url)
        } else {
            console.warn('Could not start stream')
            throw new Error('Could not start stream, webcamUv4lMjpegImage not found')
        }
    }

    stopStream() {
        console.warn('Preparing to stop stream')

        // HACK: Workaround for a known browser issue (see link below)
        //       https://bugs.chromium.org/p/chromium/issues/detail?id=73395
        console.warn('Calling window.stop()')
        window.stop()

        if (this.$refs.webcamUv4lMjpegImage) {
            console.warn('Stopping stream')
            this.$refs.webcamUv4lMjpegImage.removeAttribute('src')
            URL.revokeObjectURL(this.url)
        } else {
            console.warn('Could not stop stream')
            throw new Error('Could not step stream, webcamUv4lMjpegImage not found')
        }
    }

    // this function checks if the browser tab was changed
    documentVisibilityChanged() {
        console.warn('documentVisibilityChanged')

        const visibility = document.visibilityState
        this.isVisibleDocument = visibility === 'visible'
        if (!this.isVisibleDocument) this.stopStream()
        this.visibilityChanged()
    }

    // this function checks if the webcam is in the viewport
    viewportVisibilityChanged(newVal: boolean) {
        console.warn('viewportVisibilityChanged', newVal)

        this.isVisibleViewport = newVal
        this.visibilityChanged()
    }

    visibilityChanged() {
        console.warn('visibilityChanged')

        if (this.isVisibleViewport && this.isVisibleDocument) {
            console.warn('visibilityChanged: calling startStream')
            this.startStream()
            return
        } else {
            console.warn('visibilityChanged: calling stopStream')
        }

        this.stopStream()
    }

    onload() {
        console.warn('onload')

        if (this.aspectRatio === null && this.$refs.webcamUv4lMjpegImage) {
            this.aspectRatio =
                this.$refs.webcamUv4lMjpegImage.naturalWidth / this.$refs.webcamUv4lMjpegImage.naturalHeight
        }
    }

    @Watch('url')
    async urlChanged() {
        console.warn('urlChanged', this.url, 'calling stopStream and startStream')

        await this.stopStream()
        await this.startStream()
    }
}
</script>

<style scoped>
.webcamImage {
    width: 100%;
    background: lightgray;
}
</style>
