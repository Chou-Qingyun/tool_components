<template>
    <div class="boom-main" >
        <canvas id="canvas-i" ref="canvasRef" :height="height" :width="canwidth"></canvas>
    </div>
</template>

<script setup>
import {defineProps, onMounted, onUnmounted, ref, defineEmits, watch, toRefs} from "vue";
const prop = defineProps({
    height: { //设置弹幕的高度
        type: Number,
        default: 300
    },
    skip: { //弹幕每帧移动的距离
        type: Number,
        default: 1,
    },
    fontColor: { //设置弹幕字体的颜色
        type: Array,
        default: ['#000']
    },
    words: { //设置弹幕的内容
        type: Array,
        default: []
    },
    fontSize: { //设置弹幕的字体大小
        type: Number,
        default: 20
    },
    intervalWidth: { //设置弹幕字幕水平之间的间隔距离
        type: Number,
        default: 50,
    },
    lineInterval: { //设置弹幕字幕垂直之间的距离
        type: Number,
        default: 10
    }
})
const emit = defineEmits(['change'])
let canvas = null
let ctx = null
const canvasRef = ref()
const canwidth = ref(window.innerWidth)
const xcount = []
const hPosition = ref([])
const xPosition = ref([])
const lineRecord = []
const wordsWidth = ref([])
const colors = []
const alive = []
let moveLength = 0
const barrageLoop = () => {
    window.requestAnimationFrame(barrageLoop)
    //重置画布
    clearCanvas()
    drawCanvas()
}
const clearCanvas = () => {
    ctx.clearRect(0, 0, canwidth.value, prop.height)
}
const initBorn = () => {
    ctx.save()
    const mod = Math.floor(prop.height / (prop.fontSize + prop.lineInterval))
    for (let i = 0; i < prop.words.length; i++) {
        if (wordsWidth.value[i]) {
            continue
        }
        const line = Math.floor(Math.random() * (mod - 1)) + 1;
        ctx.font = `${prop.fontSize}px Microsoft Yahei`
        wordsWidth.value[i] = ctx.measureText(prop.words[i]).width //获取弹幕内容的长度
        //判断字体颜色是不是超过一个
        if (prop.fontColor.length > 1) {
            const index = Math.floor(Math.random() * prop.fontColor.length)
            colors[i] = prop.fontColor[index]
        }
        if (lineRecord[line] && alive[lineRecord[line]['index']]) {
            xPosition.value[i] = bornExistsPosition(line, xcount[lineRecord[line]['index']])
        } else {
            xPosition.value[i] = boomPosition()
        }
        lineRecord[line] = {index: i, value: xPosition.value[i] + wordsWidth.value[i] + prop.intervalWidth, swidth: wordsWidth.value[i]}
        if (line === 1) {
            hPosition.value[i] = prop.fontSize
        } else {
            hPosition.value[i] = line * prop.fontSize + (line - 1) * prop.lineInterval
        }
        alive[i] = true
        xcount[i] = 0
    }
    ctx.restore()
}
//弹幕字的出生位置
const boomPosition = () => {
    return window.innerWidth + Math.floor(Math.random() * 100) + 50
}
//出生在已有字幕的行中，该出生的位置
const bornExistsPosition = (line, moveLength) => {
    if (moveLength > 0) {
        //判断移动的距离是否超过出生的偏移量
        if (lineRecord[line]['value'] - moveLength > 0) {
            const second_distance =  lineRecord[line]['value'] - moveLength + lineRecord[line]['swidth'] + prop.intervalWidth
            return second_distance < window.innerWidth ? boomPosition() : second_distance
        } else {
            return boomPosition()
        }
    } else {
        return lineRecord[line]['value'] + prop.intervalWidth
    }
}
//在画布上写字
const drawCanvas = () => {
    if (!prop.words.length) {
        return
    }
    for (let i = 0; i < prop.words.length; i++) {
        if (!alive[i]) {
            continue
        }
        ctx.save()
        if (prop.fontColor.length > 1) {
            ctx.fillStyle = `${colors[i]}`
        } else {
            ctx.fillStyle = `${prop.fontColor}`
        }
        ctx.font = `${prop.fontSize}px Microsoft Yahei`
        const text = prop.words[i]
        const eleWidth = wordsWidth.value[i]
        xcount[i] += prop.skip
        // console.log(`${prop.words[i]}: `, xPosition.value[i] - xcount[i] + eleWidth);
        // debugger
        if ((xPosition.value[i] + eleWidth - xcount[i]) < 0) {
            alive[i] = false
        }
        ctx.fillText(text, xPosition.value[i] - xcount[i], hPosition.value[i])
        ctx.restore()
    }
}
const initCanvas = () => {
    canvas = document.getElementById('canvas-i')
    ctx = canvas.getContext('2d')
}
onMounted(() => {
    initCanvas()
    initBorn()
    barrageLoop()
    window.addEventListener('resize', () => {
        canwidth.value = window.innerWidth
        initCanvas()
    })
})
watch(prop, (newVal, oldVal) => {
    initBorn()
})

</script>

<style scoped lang="scss">
.boom-main {
    &, & * {
        box-sizing: border-box;
    }
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
}

</style>