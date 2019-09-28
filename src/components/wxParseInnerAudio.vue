<template>

    <div class="record_box">

        <div @click='playRecord' class='record_btns'>
            <i v-if="isPaused" class="icon icon-play"></i>
            <i v-else class="icon icon-pause"></i>
        </div>

        <div class='record_actions pt-1' v-show="isSeekBtn">
            <span class="record_speed mr-1" @click="backwardMusic(10)">
                <img mode="widthFix" style="width: 35px;" src="/static/images/10_sec_backward.png">
            </span>
            <span class="record_speed" @click="forwardMusic(10)">
                <img mode="widthFix" style="width: 35px;" src="/static/images/10_sec_forward.png">
            </span>
        </div>

        <div class='record_wave' :class=" isPaused ? '': 'active'">
            <div class='wave1'></div>
            <div class='wave2'></div>
            <div class='wave3'></div>
            <div class='wave4'></div>
            <div class='wave5'></div>
            <div class='wave6'></div>
            <div class='wave7'></div>
            <div class='wave8'></div>
            <div class='wave9'></div>
            <div class='wave10'></div>
        </div>

        <div class="features">
            <div class='record_time'>
                <span>{{lastTime}}</span>
            </div>
        </div>

        <slot></slot>
    </div>
</template>

<script>
    export default {
        name    : 'audio',
        props   : {
            url: {
                type : String,
                value: ''
            },
            seek:{
                type : Boolean,
                value : ''
            }
        },
        data() {
            return {
                innerAudio       : {},
                isPaused         : true, //是否暂停,true 暂停
                lastTime         : '00:00"', //剩余时间
                currentTime      : 0, //播放时间
                clearWaveInterval: null, //其他音频播放的时候暂停动画,
                clearTimeTrigger : null, //其他音频播放的时候暂停动画,
                isSeekBtn        : this.seek === '' ? false : this.seek
            }
        },
        onLoad( options ) {
            console.log( this.isSeekBtn, this.seek )
            this.innerAudio = wx.createInnerAudioContext()
            this.innerAudio.onEnded( () => {
                this.innerAudio.offTimeUpdate()
                this.isPaused    = true
                this.currentTime = 0
            } )
            this.innerAudio.onPlay( () => {
                this.isPaused = false
                if( this.innerAudio.currentTime === 0 ) {
                    this.clearTimeTrigger = setTimeout( () => {
                        console.log( 'currentTime ：', this.innerAudio.currentTime )
                    }, 500 )
                }else{
                    clearTimeout( this.clearTimeTrigger )
                }

                this.innerAudio.onTimeUpdate( () => {
                    this.currentTime = this.innerAudio.currentTime
                    this.lastTime    = this.formatSeconds( this.currentTime )
                    console.log( 'Last Time: ', this.lastTime, this.currentTime, this.innerAudio.currentTime )
                    clearTimeout( this.clearWaveInterval )
                    this.clearWaveInterval = setTimeout( () => {
                        this.isPaused = true
                    }, 500 )
                } )
            } )
            this.innerAudio.onSeeked( () => {
                this.isPaused    = false
                this.currentTime = this.innerAudio.currentTime
                this.innerAudio.play()
            } )
        },
        onUnload: function( e ) {
            this.innerAudio.destroy()//销毁这个实例
        },
        methods : {
            formatSeconds( s ) {
                let str = ''
                if( s > 0 ) {
                    const minutes = Math.floor( s / 60 )
                    const seconds = Math.floor( s - minutes * 60 )
                    let mStr      = minutes < 10 ? '0' + minutes : minutes
                    let sStr      = seconds < 10 ? '0' + seconds : seconds
                    str           = mStr + ':' + sStr + '"'
                }

                return str
            },
            forwardMusic( second ) {
                let diff = this.currentTime + second
                console.log( 'forwardMusic', this.currentTime, diff, this.innerAudio.duration )
                if( diff > this.innerAudio.duration ) {
                    this.currentTime = this.innerAudio.duration
                    this.innerAudio.offEnded()
                }else{
                    this.innerAudio.seek( diff )
                }
            },
            backwardMusic( second ) {
                let diff = this.currentTime - second
                console.log( 'backwardMusic', this.currentTime, diff )
                if( diff < 0 ) {
                    this.currentTime = 0
                    this.innerAudio.stop()
                }else{
                    this.currentTime = diff
                    this.innerAudio.seek( diff )
                }
            },
            pauseMuisc() {
                this.innerAudio.pause()
                this.currentTime = this.innerAudio.currentTime
                this.isPaused    = true
                this.innerAudio.offEnded()
                this.innerAudio.offStop()
                this.innerAudio.offTimeUpdate()
            },
            playRecord() {
                this.innerAudio.src = this.url
                if( this.isPaused ) {
                    this.innerAudio.play()
                    this.innerAudio.startTime = this.currentTime
                    this.isPaused        = false
                }else{
                    this.pauseMuisc()
                }
            }
        }
    }
</script>

<style scoped>
    .icon {
        display: inline-block;
        width: 25px;
        height: 25px;
        background-size: cover;
        margin-top: 12px;
    }

    .icon-play {
        width: 40px;
        height: 40px;
        background-image: url("data:image/svg+xml, %3Csvg class='icon' viewBox='0 0 1024 1024' xmlns='http://www.w3.org/2000/svg' width='200' height='200'%3E%3Cdefs%3E%3Cstyle/%3E%3C/defs%3E%3Cpath d='M694.628 510.982L381.157 667.785V354.19l313.471 156.793z' fill='%23097C25'/%3E%3Cpath d='M512 54.63C260.17 54.63 56.02 259.402 56.02 512S260.17 969.37 512 969.37 967.98 764.598 967.98 512 763.83 54.63 512 54.63zm0 853.83c-218.294 0-395.256-177.501-395.256-396.46S293.706 115.54 512 115.54 907.256 293.04 907.256 512 730.294 908.46 512 908.46z' fill='%23097C25'/%3E%3C/svg%3E");
    }

    .icon-pause {
        width: 40px;
        height: 40px;
        background-image: url("data:image/svg+xml, %3Csvg class='icon' viewBox='0 0 1024 1024' xmlns='http://www.w3.org/2000/svg' width='200' height='200'%3E%3Cdefs%3E%3Cstyle/%3E%3C/defs%3E%3Cpath d='M409.28 310.62a36 36 0 0 0-36 36v329.3a36 36 0 0 0 72 0v-329.3a36 36 0 0 0-36-36zm205.44 1.46a36 36 0 0 0-36 36v329.3a36 36 0 0 0 72 0v-329.3a36 36 0 0 0-36-36z' fill='%23097C25'/%3E%3Cpath d='M512 0C229.23 0 0 229.23 0 512s229.23 512 512 512 512-229.23 512-512S794.77 0 512 0zm311.13 823.13a438.4 438.4 0 1 1 94.33-139.88 438.62 438.62 0 0 1-94.33 139.88z' fill='%23097C25'/%3E%3C/svg%3E");
    }

    .record_box {
        display: flex;
        flex-wrap: nowrap;
        background: #E9E9E9;
        height: 60px;
        padding: 0 10px;
        align-items: center;
        justify-content: space-between;
        color: #097C25;
    }

    .record_time {
        width: 100%;
        text-align: right;
        flex-grow: 0;
        font-size: 12px;
        margin-top: 5px;
    }

    .record_btns {
        height: 60px;
        padding: 0 15px;
        line-height: 50px;
    }

    .record_wave {
        display: flex;
        align-items: center;
        margin-top: 5px;
    }

    .record_wave div {
        width: 4px;
        height: 15px;
        transform: scaleY(0.2);
        background: #ff7b00;
        margin: 0 3px;
        border-radius: 5px;
    }

    .record_wave.active div {
        animation: wave 0.8s infinite ease-in-out;
    }

    .record_wave.active .wave1, .record_wave.active .wave6 {
        animation-delay: 0s;
    }

    .record_wave.active .wave2, .record_wave.active .wave7 {
        animation-delay: 0.2s;
    }

    .record_wave.active .wave3, .record_wave.active .wave8 {
        animation-delay: 0.4s;
    }

    .record_wave.active .wave4, .record_wave.active .wave9 {
        animation-delay: 0.6s;
    }

    .record_wave.active .wave5, .record_wave.active .wave10 {
        animation-delay: 0.8s;
    }

    @keyframes wave {
        0% {
            transform: scaleY(0.2)
        }
        50% {
            transform: scaleY(1);
        }
        0% {
            transform: scaleY(0.2);
        }
    }
</style>
