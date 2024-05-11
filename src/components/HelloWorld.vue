<!--
 * @Author: zyl
 * @Date: 2023-07-17 15:13:29
 * @Description: 
-->
<template>
    <div ref="chartRef" class="dubang-chart">
        <canvas class="canvas" ref="canvasRef" :style="`width:${state.width}px;height:${state.height}px;`"></canvas>
        <div v-for="item in state.list">
            <div class="box" :style="{
                top: item.top + 'px',
                left: item.left + 'px'
            }">
                <div>{{ 'ID:' + item.key }} {{ '层级:' + item.level }}</div>
                <div>{{ '子节点个数: ' + item.subCount }}</div>
                <div>{{ '父节点ID: ' + item.parent }}</div>
            </div>
        </div>
    </div>
</template>
<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({
    name: 'meta-dubang-chart'
})
</script>
<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
const state = reactive({
    source: [],
    width: 0,
    height: 0,
    list: [] as any[],
    indexs: [] as any[]
})
const chartRef: any = ref()
const canvasRef: any = ref()
const boxWidth = 108
const boxHeight = 68
const boxMargin = 30
onMounted(() => {
    calculatePosition()
})
function calculatePosition() {
    const indexs = [
        { label: "0-1", top: 0, left: 0, key: "1", parent: null, level: null, offset: 0, children: [] },
        { label: "1-2", top: 0, left: 0, key: "2", parent: "1", level: null, offset: 0, children: [] },
        { label: "1-3", top: 0, left: 0, key: "3", parent: "1", level: null, offset: 0, children: [] },
        { label: "1-4", top: 0, left: 0, key: "4", parent: "1", level: null, offset: 0, children: [] },
        { label: "1-5", top: 0, left: 0, key: "5", parent: "1", level: null, offset: 0, children: [] },
        { label: "2-6", top: 0, left: 0, key: "6", parent: "5", level: null, offset: 0, children: [] },
        { label: "2-7", top: 0, left: 0, key: "7", parent: "5", level: null, offset: 0, children: [] },
        { label: "2-8", top: 0, left: 0, key: "8", parent: "2", level: null, offset: 0, children: [] },
        { label: "2-9", top: 0, left: 0, key: "9", parent: "3", level: null, offset: 0, children: [] },
        { label: "2-10", top: 0, left: 0, key: "10", parent: "3", level: null, offset: 0, children: [] },
        { label: "2-11", top: 0, left: 0, key: "11", parent: "3", level: null, offset: 0, children: [] },
        { label: "2-12", top: 0, left: 0, key: "12", parent: "2", level: null, offset: 0, children: [] },
        { label: "0-13", top: 0, left: 0, key: "13", parent: null, level: null, offset: 0, children: [] },
        { label: "1-14", top: 0, left: 0, key: "14", parent: "13", level: null, offset: 0, children: [] },
        { label: "1-15", top: 0, left: 0, key: "15", parent: "13", level: null, offset: 0, children: [] },
        { label: "1-16", top: 0, left: 0, key: "16", parent: "13", level: null, offset: 0, children: [] },
        { label: "3-17", top: 0, left: 0, key: "17", parent: "12", level: null, offset: 0, children: [] },
        { label: "3-18", top: 0, left: 0, key: "18", parent: "12", level: null, offset: 0, children: [] },
        { label: "3-19", top: 0, left: 0, key: "19", parent: "12", level: null, offset: 0, children: [] },
        { label: "3-20", top: 0, left: 0, key: "20", parent: null, level: null, offset: 0, children: [] },
        { label: "3-21", top: 0, left: 0, key: "21", parent: "20", level: null, offset: 0, children: [] },
        { label: "3-22", top: 0, left: 0, key: "22", parent: "20", level: null, offset: 0, children: [] },
        { label: "1-23", top: 0, left: 0, key: "23", parent: "14", level: null, offset: 0, children: [] },
        { label: "1-24", top: 0, left: 0, key: "24", parent: "14", level: null, offset: 0, children: [] },
    ] as any[]
    let treeList = JSON.parse(JSON.stringify(indexs))
    treeList.forEach((e: any) => {
        e.children = []
        e.offset = 0
        e.left = 0
        e.top = 0
    })
    treeList.forEach((e: any) => {
        if (e.parent) {
            const parent = treeList.find((p: any) => p.key == e.parent)
            if (parent) {
                e.parentProto = parent
                parent.children.push(e)
                e.delete = true
            }
        }
    })
    treeList = treeList.filter((e: any) => !e.delete)
    const uilist = [] as any[]
    //初步设置布局
    const loop = (list: any[], parent?: any) => {
        if (parent) {
            parent.offset = parent.left - ((list.length * (boxWidth + boxMargin)) / 2) + (boxWidth + boxMargin) / 2
            parent.subCount = list.length
        }
        list.forEach((e: any) => {
            if (parent) {
                e.level = parent.level + 1
                e.left = parent.offset
                parent.offset = parent.offset + boxWidth + boxMargin
            } else {
                e.level = 0
            }
            if (e.children) loop(e.children, e)
            e.top = e.level * (boxHeight + boxMargin)
            uilist.push(e)
        })
    }
    loop(treeList)

    //控制偏移量
    const setOffset = (list: any[], offset: any) => {
        list.forEach((e: any) => {
            e.left = e.left - offset
            if (e.children) {
                setOffset(e.children, offset)
            }
        })
    }

    //获取子节点的最左或者最后位置
    const getChildLeft = (list: any[], cur: any, position: 'left' | 'right') => {
        if (list?.length > 0) {
            if (position === 'left') {
                return list[0].left
            } else {
                return list[list.length - 1].left + boxWidth + boxMargin
            }
        }
        return null
    }

    //获取最右边的节点位置
    const getMaxRight = (list: any[]) => {
        let max = null as any
        list.forEach((e: any) => {
            const right = getChildLeft(e.children, e, 'right')
            if (right > max) {
                max = right
            }
        })
        return max
    }

    //控制元素偏移
    const loopOffset = (list: any[], parent?: any) => {
        if (parent) {
            let offset = 0
            let pre_list = [] as any[]
            list.forEach((e: any, index: number) => {
                const pre_most_right = getMaxRight(pre_list)
                const next = (index === list.length - 1) ? null : list[index + 1]
                const next_left = getChildLeft(next?.children, next, 'left')
                if (pre_most_right != null && next_left != null && pre_most_right > next_left) {
                    offset = pre_most_right - next_left
                    e.left = e.left - offset
                    setOffset(e.children, offset)
                    setOffset(pre_list, offset)
                }
                if (e.children?.length > 0) {
                    if (parent) {
                        if (next?.children?.length > 0) {
                            const cur_right = getChildLeft(e.children, e, 'right')
                            if (cur_right > next_left) {
                                offset = cur_right - next_left
                                e.left = e.left - offset
                                setOffset(e.children, offset)
                                setOffset(pre_list, offset)
                            }
                        }
                    }
                }
                pre_list.push(e)
            })
        }
        list.forEach((e: any) => {
            if (e.children) {
                loopOffset(e.children, e)
            }
        })
    }
    loopOffset(treeList)
    let min_left = 0
    uilist.forEach((e: any) => {
        if (min_left > e.left) {
            min_left = e.left
        }
    })

    //获取树的最大宽度
    const getTreeSizeWidth = (list: any, size: any, cur?: any) => {
        if (list?.length > 0) {
            list?.forEach((e: any) => {
                if (size.most_left == null) {
                    size.most_left = e.left
                }
                if (size.most_right == null) {
                    size.most_right = e.left + boxWidth + boxMargin
                }
                if (e.left + boxWidth + boxMargin > size.most_right) {
                    size.most_right = e.left + boxWidth + boxMargin
                }
                if (e.left < size.most_left) {
                    size.most_left = e.left
                }
                if (e.children) {
                    getTreeSizeWidth(e.children, size)
                }
            })
        } else if (cur) {
            size.most_left = cur.left
            size.most_right = cur.left + boxWidth + boxMargin
        }
    }
    let pre_most_width = null as any
    //控制树最外层的偏移量
    treeList.forEach((e: any) => {
        const size = { most_left: null, most_right: null } as any
        getTreeSizeWidth(e.children, size, e)
        let tree_size_width = size.most_right - size.most_left
        if (pre_most_width != null) {
            let half_width = e.left - size.most_left  //头部节点到最左节点的距离
            min_left = - pre_most_width - half_width
        }
        setOffset([e], min_left)
        pre_most_width = pre_most_width + tree_size_width
    })
    let max_width = 0
    let max_height = 0
    uilist.forEach((e: any) => {
        if (e.left + boxWidth + boxMargin > max_width) {
            max_width = e.left + boxWidth + boxMargin
        }
        if (e.top + boxHeight + boxMargin > max_height) {
            max_height = e.top + boxHeight + boxMargin
        }
    })
    state.width = max_width
    state.height = max_height
    state.list = uilist
    if (uilist.length > 0) drawLine(uilist)
}

function drawLine(list: any[]) {
    const pen = canvasRef.value.getContext("2d");
    const devicePixelRatio = window.devicePixelRatio || 1
    const backingStoreRatio = pen.webkitBackingStorePixelRatio || 1
    const ratio = devicePixelRatio / backingStoreRatio
    canvasRef.value.width = state.width * ratio
    canvasRef.value.height = state.height * ratio
    pen.scale(ratio, ratio)
    pen.clearRect(0, 0, state.width, state.height);
    pen.strokeStyle = "#DCDFE6";
    pen.lineWidth = 1
    list.forEach((e: any) => {
        if (e.parentProto) {
            pen.beginPath();
            pen.moveTo(e.left + boxWidth / 2, e.top);
            pen.lineTo(e.left + boxWidth / 2, e.top - boxMargin / 2);
            pen.lineTo(e.parentProto.left + boxWidth / 2, e.parentProto.top + boxHeight + boxMargin / 2);
            pen.lineTo(e.parentProto.left + boxWidth / 2, e.parentProto.top + boxHeight);
            pen.stroke();
            pen.closePath();

            pen.beginPath();
            pen.moveTo(e.left + boxWidth / 2, e.top);
            pen.lineTo(e.left + boxWidth / 2 - 6, e.top - 8);
            pen.lineTo(e.left + boxWidth / 2 + 6, e.top - 8);
            pen.closePath();
            pen.fillStyle = "#337ECC";
            pen.fill();
        }
    })
}

</script>

<style scoped>
.dubang-chart {
    width: 100%;
    min-height: 60px;
    .canvas{
        left: 0px;
        top: 0px;
        display: flex;
        position: absolute;
    }
    .box {
        display: flex;
        position: absolute;
        flex-direction: column;
        padding: 8px;
        width: 108px;
        height: 88px;
        text-align: center;
        justify-content: start;
        align-items: flex-start;
        font-size: 12px;
        border-radius: 4px;
        box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.1), inset 0px 2px 0px 0px #409EFF;
    }
}
</style>
