<style type="text/css">
    .in-out-translate-demo-wrapper {
        position: relative;
        height: 58px;
    }
    
    .in-out-translate-demo-wrapper button {
        position: absolute;
    }
    
    .in-out-translate-fade-enter-active,
    .in-out-translate-fade-leave-active {
        transition: all .5s;
        -webkit-transition: all .5s;
        -moz-transition: all .5s;
        -o-transition: all .5s;
    }
    
    .in-out-translate-fade-enter,
    .in-out-translate-fade-leave-active {
        opacity: 0;
    }
    
    .in-out-translate-fade-enter {
        transform: translateX(54px);
        -webkit-transform: translateX(54px);
        -moz-transform: translateX(54px);
        -o-transform: translateX(54px);
    }
    
    .in-out-translate-fade-leave-active {
        transform: translateX(-54px);
        -webkit-transform: translateX(-54px);
        -moz-transform: translateX(-54px);
        -o-transform: translateX(-54px);
    }
    
    .down-up-translate-fade-enter-active,
    .down-up-translate-fade-leave-active {
        transition: all .1s;
        -webkit-transition: all .1s;
        -moz-transition: all .1s;
        -o-transition: all .1s;
    }
    
    .down-up-translate-fade-enter,
    .down-up-translate-fade-leave-active {
        opacity: 1;
    }
    
    .down-up-translate-fade-enter {
        /*transform: translateY(40px);
                        -webkit-transform: translateY(40px);
                        -moz-transform: translateY(40px);
                        -o-transform: translateY(40px);*/
    }
    
    .down-up-translate-fade-leave-active {
        transform: translateY(-50px);
        -webkit-transform: translateY(-50px);
        -moz-transform: translateY(-50px);
        -o-transform: translateY(-50px);
    }
    
    .num {
        /*position: relative;*/
        width: 100%;
        height: 50px;
        overflow: hidden;
    }
    
    .num .span {
        /*position: absolute;*/
        background: #0062CC;
        color: #fff;
        width: 30px;
        height: 50px;
        line-height: 50px;
        font-size: 40px;
        font-weight: bold;
        float: left;
        margin-right: 2px;
    }
    
    .num .span div {
        background: #0062CC;
        color: #fff;
        width: 30px;
        height: 50px;
        line-height: 50px;
        font-size: 40px;
        font-weight: bold;
    }
</style>


<template>
    <div id="demo" class="demo">
        <div class="in-out-translate-demo-wrapper mui-text-center">
            <transition name="in-out-translate-fade">
                <button v-if="show" v-on:click="start" key="start" type="button" class="mui-btn mui-btn-blue mui-btn-outlined">开始</button>
                <button v-else key="end" v-on:click="end" type="button" class="mui-btn mui-btn-yellow mui-btn-outlined">结束</button>
            </transition>
    
        </div>
        <div class="num mui-text-center">
            <div class="span">
                <div>1</div>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="i">{{i}}</div>
                </transition>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="j">{{j}}</div>
                </transition>
            </div>
            <div class="span">
                <div>*</div>
            </div>
            <div class="span">
                <div>*</div>
            </div>
            <div class="span">
                <div>*</div>
            </div>
            <div class="span">
                <div>*</div>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="k">{{k}}</div>
                </transition>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="l">{{l}}</div>
                </transition>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="i">{{i}}</div>
                </transition>
            </div>
            <div class="span">
                <transition name="down-up-translate-fade">
                    <div :key="j">{{j}}</div>
                </transition>
            </div>
        </div>
    </div>
</template>

<script type="text/javascript">
    export default {
        el: '#demo',
        data() {
            return {
                show: true,
                i: 0,
                j: 0,
                k: 0,
                l: 0,
                interval: null
            }
        },
        methods: {
            start: function() {
                this.show = !this.show
                var _this = this;
    
                if (!this.interval) {
                    this.interval = setInterval(function() {
                        _this.i = Math.floor(Math.random() * 10);
                        _this.j = Math.floor(Math.random() * 10);
                        _this.k = Math.floor(Math.random() * 10);
                        _this.l = Math.floor(Math.random() * 10);
                    }, 10)
                }
            },
            end: function() {
                this.show = !this.show
    
                clearInterval(this.interval)
                this.interval = null
            }
        }
    }
</script>

