<template>
    <div style="position: relative;flex: 1 1 auto;">
        <div class="container-view">
            <div class="canvas-view" :class="cursorStatus" @keyup.space="stagePointer"></div>
            <div class="layer-view"></div>
        </div>
        <div class="propertiesPanelContainer"></div>
    </div>
</template>

<script>
import { Application, Container, Sprite, Graphics, Point, settings, SCALE_MODES } from 'pixi.js'

export default {
    name: 'custom-canvas',
    data () {
        return {
            app: {},
            container: {},
            focusedGraphics: {},
            canvasData: {
                id: 'parentContainer',
                x: 152,
                y: 50,
                type: 'Container',
                children: [
                    {
                        id: 'blankPresets',
                        x: 0,
                        y: 0,
                        width: 485,
                        height: 975,
                        type: 'Rectangle',
                        scale: {
                            x: 0.8,
                            y: 0.8
                        },
                        zIndex: 0,
                    },
                    {
                        id: 'image1',
                        x: 20,
                        y: 20,
                        width: 300,
                        height: 200,
                        type: 'Image',
                        url: 'http://anata.me/img/avatar.jpg',
                        zIndex: 1
                    },
                    {
                        id: 'circle1',
                        x: 200,
                        y: 200,
                        radius: 32,
                        type: 'Circle',
                        scale: {
                            x: 1,
                            y: 1
                        },
                        zIndex: 1
                    }
                ]
            },
            struct: new Map,
            focusBorder: new Map,
            cursorStatus: ''
        }
    },

    mounted () {
        this.app = new Application({
            resizeTo: document.getElementsByClassName('canvas-view')[0],
            backgroundColor: 0xe5e5e5,
            antialias: true
        })
        settings.SCALE_MODE = SCALE_MODES.NEAREST
        console.log('缩放模式', settings.SCALE_MODE)
        document.getElementsByClassName('canvas-view')[0].appendChild(this.app.view)
        this.parseCanvasData(this.canvasData)

        // 滚轮缩放监听
        document.getElementsByClassName('canvas-view')[0].addEventListener('mousewheel', (event) => {
            const step = event.wheelDelta > 0 ? 0.1 : -0.1
            if (this.app.stage.scale.x + step >= 0.1) {
                this.app.stage.scale.x += step
                this.app.stage.scale.y += step
            }
        })
    },
    methods: {
        stagePointer () {
            this.cursorStatus = 'cursor_pointer'
        },
        parseCanvasData (canvasData) {  
            // // 父容器
            this.container = new Container()
            this.container.x = canvasData.x
            this.container.y = canvasData.y
            // console.log(this.container.width, this.container.height)
            this.container.sortableChildren = true
            this.app.stage.addChild(this.container)
            
            // container.mask = new Graphics().beginFill(0xffffff).drawRect(20, 20, 487, 975).endFill()

            // this.struct.set()
            canvasData.children.forEach(element => {
                if (element.type === 'Rectangle') {
                    let blankPresets = new Graphics()
                    blankPresets.beginFill(0xffffff)
                    blankPresets.drawRect(0, 0, element.width, element.height)
                    blankPresets.endFill()
                    blankPresets.x = element.x
                    blankPresets.y = element.y
                    blankPresets.zIndex = 0
                    blankPresets.scale.set(element.scale.x, element.scale.y)
                    blankPresets.name = element.id
                    this.container.addChild(blankPresets)
                } else if (element.type === 'Image') {
                    let sprite = new Sprite.from(element.url)
                    sprite.width = 180
                    sprite.height = 120
                    sprite.interactive = true
                    sprite.buttonMode = true
                    
                    sprite.zIndex = 1
                    sprite.x = element.x
                    sprite.y = element.y
                    sprite.name = element.id
                    this.container.addChild(sprite)
                    this.bindEvent(sprite, 'Image')
                    console.log(sprite.position, sprite.toGlobal(new Point(0, 0)))

                    // this.struct.push({
                    //     id: 'avatar',
                    //     type: 'image',
                    //     width: 180,
                    //     height: 120,
                    // })
                } else if (element.type === 'Circle') {
                    let circle = new Graphics()
                    circle.beginFill(0xc4c4c4)
                    circle.drawCircle(0, 0, 32)
                    circle.endFill()
                    circle.x = element.x
                    circle.y = element.y
                    // circle.position.set(400, 400)
                    circle.zIndex = 1
                    circle.scale.set(element.scale.x, element.scale.y)
                    circle.interactive = true
                    circle.buttonMode = true

                    circle.name = element.id
                    this.container.addChild(circle)
                    this.bindEvent(circle, 'Circle')
                }
            })
            // this.app.stage.addChild(blankPresets)
        },
        // 绑定事件
        bindEvent (graphics, type) {
            if (type === 'Circle' || type === 'Image' || type === 'Rectange') {
                let borderCir
                let startPoint
                let dragStatus = false
                // let line1, line2, line3, line4
                // let scalePoint1, scalePoint2, scalePoint3, scalePoint4
                graphics.on('pointerover', () => {
                    console.log('指针移入')
                    if (!borderCir && this.focusedGraphics.name !== graphics.name) {
                        if (type === 'Circle') {
                            borderCir = new Graphics()
                            borderCir.beginFill(0x18a0fb)
                            borderCir.drawCircle(0, 0, 33.5)
                            borderCir.endFill()
                            borderCir.x = graphics.x
                            borderCir.y = graphics.y
                            borderCir.zIndex = 0
                            this.container.addChild(borderCir)
                        } else if (type === 'Image') {
                            borderCir = new Graphics()
                            borderCir.beginFill(0x18a0fb)
                            borderCir.drawRect(0, 0, graphics.width + 3, graphics.height + 3)
                            borderCir.endFill()
                            borderCir.x = graphics.x - 1.5
                            borderCir.y = graphics.y - 1.5
                            borderCir.zIndex = 0
                            this.container.addChild(borderCir)
                        }
                        
                    }
                })
                graphics.on('pointerout', () => {
                    console.log('指针移出')
                    if (dragStatus) return
                    // dragStatus = false
                    borderCir && borderCir.destroy()
                    borderCir = null
                })
                graphics.on('pointerdown', (event) => {
                    console.log('指针按下')
                    dragStatus = true
                    startPoint = {
                        // x: event.data.global.x,
                        // y: event.data.global.y
                        x: event.data.getLocalPosition(this.container).x,
                        y: event.data.getLocalPosition(this.container).y
                    }
                    // console.log(event.data.getLocalPosition(this.container))
                    // console.log(graphics.position)
                    // if (this.focusedGraphics.name !== graphics.name) {
                    borderCir && borderCir.destroy()
                    borderCir = null
                    this.focusGraphics(graphics, type)
                    // }
                })
                graphics.on('pointerup', () => {
                    console.log('指针释放')
                    dragStatus = false
                    borderCir && borderCir.destroy()
                    borderCir = null
                    if (this.focusedGraphics.name === graphics.name) {
                        // let borderArr
                        // if (graphics.type === 'Circle') {
                        //     borderArr = this.drawFocusBorderForCircle(graphics)
                        // } else if (graphics.type === 'Image') {
                        //     borderArr = this.drawFocusBorderForRect(graphics)
                        // }
                        // this.container.addChild(...borderArr)

                        // this.focusedGraphics = graphics
                        // if (!this.focusBorder.has(graphics.name)) {
                        //     this.focusBorder.set(graphics.name, borderArr)
                        //     console.log(this.focusBorder.get(graphics.name))
                        // }
                        this.focusGraphics(graphics, type)
                    }
                })
                // graphics.on('pointertap', () => {
                //     console.log('指针点击')
                //     if (type === 'Circle' || type === 'Image') {
                //         console.log('划线', graphics.width, graphics.x, graphics.y)
                //         console.log(this.focusedGraphics.name, graphics.name)
                //         if (this.focusedGraphics.name !== graphics.name) {
                //             // let radius = graphics.width / 2 - 2
                //             // line1 = this.drawDash({ x: graphics.x - radius, y: graphics.y - radius }, { x: graphics.x + radius, y: graphics.y - radius })
                //             // line2 = this.drawDash({ x: graphics.x + radius, y: graphics.y - radius }, { x: graphics.x + radius, y: graphics.y + radius })
                //             // line3 = this.drawDash({ x: graphics.x + radius, y: graphics.y + radius }, { x: graphics.x - radius, y: graphics.y + radius })
                //             // line4 = this.drawDash({ x: graphics.x - radius, y: graphics.y + radius }, { x: graphics.x - radius, y: graphics.y - radius })
                            
                //             // // scalePoint1 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y - graphics.width / 2 })
                //             // // scalePoint2 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y - graphics.width / 2 })
                //             // // scalePoint3 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y + graphics.width / 2 })
                //             // // scalePoint4 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y + graphics.width / 2 })
                //             // let scaleRadius1 = graphics.width / 2 - 3
                //             // let scaleRadius2 = graphics.width / 2 + 3
                //             // scalePoint1 = this.drawRect(graphics.x - scaleRadius2, graphics.y - scaleRadius2, 6, 6)
                //             // scalePoint2 = this.drawRect(graphics.x + scaleRadius1, graphics.y - scaleRadius2, 6, 6)
                //             // scalePoint3 = this.drawRect(graphics.x + scaleRadius1, graphics.y + scaleRadius1, 6, 6)
                //             // scalePoint4 = this.drawRect(graphics.x - scaleRadius2, graphics.y + scaleRadius1, 6, 6)
                //             // let borderArr
                //             // if (type === 'Circle') {
                //             //     borderArr = this.drawFocusBorderForCircle(graphics)
                //             // } else if (type === 'Image' || type === 'Rectange') {
                //             //     borderArr = this.drawFocusBorderForRect(graphics)
                //             // }
                            
                //             // this.container.addChild(...borderArr)
                //             // this.focusedGraphics = graphics
                //             // if (!this.focusBorder.has(graphics.name)) {
                //             //     // let borderArr = []
                //             //     // borderArr.push(line1, line2, line3, line4, scalePoint1, scalePoint2, scalePoint3, scalePoint4)
                //             //     // this.focusBorder.set(graphics.name, borderArr)
                //             //     this.focusBorder.set(graphics.name, borderArr)
                //             //     console.log(this.focusBorder.get(graphics.name))
                //             // }
                //             console.log('点击聚焦', graphics)
                //             this.focusGraphics(graphics, type)
                //         }
                //     }
                // })
                graphics.on('pointermove', (event) => {
                    if (dragStatus && this.focusedGraphics.name === graphics.name) {
                        console.log('指针移动')
                        // line1 && (line1.visible = false)
                        // line2 && (line2.visible = false)
                        // line3 && (line3.visible = false)
                        // line4 && (line4.visible = false)
                        // scalePoint1 && (scalePoint1.visible = false)
                        // scalePoint2 && (scalePoint2.visible = false)
                        // scalePoint3 && (scalePoint3.visible = false)
                        // scalePoint4 && (scalePoint4.visible = false)
                        borderCir && borderCir.destroy()
                        borderCir = null
                        if (this.focusBorder.has(this.focusedGraphics.name) && this.focusedGraphics) {
                            // this.focusBorder.get(this.focusedGraphics.name).forEach(border => {
                            //     border.visible = false
                            //     border = null
                            // })
                            // console.log(this.focusBorder.get(this.focusedGraphics.name))
                            for (let border of this.focusBorder.get(this.focusedGraphics.name)) {
                                // border.visible = false
                                // console.log(border)
                                border.destroy()
                            }
                            this.focusBorder.delete(this.focusedGraphics.name)
                        }
                        // const dx = event.data.global.x - startPoint.x
                        // const dy = event.data.global.y - startPoint.y
                        const dx = event.data.getLocalPosition(this.container).x - startPoint.x
                        const dy = event.data.getLocalPosition(this.container).y - startPoint.y
                        // console.log(dx, dy)
                        graphics.position.x += dx
                        graphics.position.y += dy
                        // borderCir.position.x += dx
                        // borderCir.position.y += dy
                        console.log(graphics.position.x, graphics.position.y)
                        startPoint = { x: event.data.getLocalPosition(this.container).x, y: event.data.getLocalPosition(this.container).y }
                        // startPoint = { x: event.data.global.x, y: event.data.global.y }
                    }
                })
            }
        },
        // 划线
        drawDash (startPoint, endPoint, alignment) {
            // console.log(startPoint, endPoint)
            let line = new Graphics()
            line.lineStyle(1.5, 0x18a0fb, 1, alignment)
            line.moveTo(startPoint.x, startPoint.y)
            line.lineTo(endPoint.x, endPoint.y)
            line.x = 0
            line.y = 0
            // console.log(line.position, line.width, line.height)
            // this.container.addChild(line)
            return line
        },
        // 画矩形
        drawRect (x, y, width, height, direction, parent) {
            let rect = new Graphics()
            rect.beginFill(0xffffff)
            rect.lineStyle(1.5, 0x18a0fb, 1)
            rect.drawRect(0, 0, width, height)
            rect.endFill()
            rect.x = x
            rect.y = y
            rect.interactive = true
            this.bindScaleEvent(rect, direction, parent)
            // this.container.addChild(rect)
            return rect
        },
        // 绘制聚焦边框(圆形)
        drawFocusBorderForCircle (graphics) {
            let borderArr = []
            // console.log('绘制边框', graphics.width, graphics.height, graphics.x, graphics.y)
            let dx = graphics.width / 2 - 2
            let dy = graphics.height / 2 - 2
            let line1 = this.drawDash({ x: graphics.x - dx, y: graphics.y - dy }, { x: graphics.x + dx, y: graphics.y - dy }, 2)
            let line2 = this.drawDash({ x: graphics.x + dx, y: graphics.y - dy }, { x: graphics.x + dx, y: graphics.y + dy }, 2)
            let line3 = this.drawDash({ x: graphics.x + dx, y: graphics.y + dy }, { x: graphics.x - dx, y: graphics.y + dy }, 2)
            let line4 = this.drawDash({ x: graphics.x - dx, y: graphics.y + dy }, { x: graphics.x - dx, y: graphics.y - dy }, 2)
            
            // scalePoint1 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y - graphics.width / 2 })
            // scalePoint2 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y - graphics.width / 2 })
            // scalePoint3 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y + graphics.width / 2 })
            // scalePoint4 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y + graphics.width / 2 })
            dx = graphics.width / 2 - 3
            dy = graphics.height / 2 + 3
            let dm = graphics.width / 2 + 3
            let dn = graphics.height / 2 - 3
            let scalePoint1 = this.drawRect(graphics.x - dm, graphics.y - dy, 6, 6, 'LEFTTOP', graphics)
            let scalePoint2 = this.drawRect(graphics.x + dx, graphics.y - dy, 6, 6, 'RIGHTTOP', graphics)
            let scalePoint3 = this.drawRect(graphics.x + dx, graphics.y + dn, 6, 6, 'RIGHTBOTTOM', graphics)
            let scalePoint4 = this.drawRect(graphics.x - dm, graphics.y + dn, 6, 6, 'LEFTBOTTOM', graphics)
            borderArr.push(line1, line2, line3, line4, scalePoint1, scalePoint2, scalePoint3, scalePoint4)
            return borderArr
        },
        // 绘制聚焦边框(矩形)
        drawFocusBorderForRect (graphics) {
            let borderArr = []
            let d = 2
            let line1 = this.drawDash({ x: graphics.x + d, y: graphics.y }, { x: graphics.x + graphics.width - d, y: graphics.y }, 1)
            let line2 = this.drawDash({ x: graphics.x + graphics.width, y: graphics.y + d }, { x: graphics.x + graphics.width, y: graphics.y + graphics.height - d }, 1)
            let line3 = this.drawDash({ x: graphics.x + graphics.width - d, y: graphics.y + graphics.height }, { x: graphics.x + d, y: graphics.y + graphics.height }, 1)
            let line4 = this.drawDash({ x: graphics.x, y: graphics.y + graphics.height - d }, { x: graphics.x, y: graphics.y + d }, 1)
            
            d = 3
            let scalePoint1 = this.drawRect(graphics.x - d, graphics.y - d, 6, 6, 'LEFTTOP', graphics)
            let scalePoint2 = this.drawRect(graphics.x + graphics.width - d, graphics.y - d, 6, 6, 'RIGHTTOP', graphics)
            let scalePoint3 = this.drawRect(graphics.x + graphics.width - d, graphics.y + graphics.height - d, 6, 6, 'RIGHTBOTTOM', graphics)
            let scalePoint4 = this.drawRect(graphics.x - d, graphics.y + graphics.height - d, 6, 6, 'LEFTBOTTOM', graphics)
            borderArr.push(line1, line2, line3, line4, scalePoint1, scalePoint2, scalePoint3, scalePoint4)
            return borderArr
        },
        // 元素聚焦
        focusGraphics (graphics, type) {
            console.log('事件对象', graphics.name)
            let borderArr = []
            if (type === 'Circle') {
                borderArr = this.drawFocusBorderForCircle(graphics)
            } else if (type === 'Image') {
                borderArr = this.drawFocusBorderForRect(graphics)
            }
            // console.log('边框', borderArr)
            this.container.addChild(...borderArr)

            this.focusedGraphics = graphics
            console.log('focusBorder', this.focusBorder)
            for (let key of this.focusBorder.keys()) {
                console.log(key)
                for (let border of this.focusBorder.get(key)) {
                    border.destroy()
                }
                this.focusBorder.delete(key)
            }
            if (!this.focusBorder.has(graphics.name)) {
                this.focusBorder.set(graphics.name, borderArr)
                // console.log(this.focusBorder.get(graphics.name))
            }
        },
        // 缩放事件绑定
        bindScaleEvent (graphics, direction, parent) {
            console.log('注册监听', graphics)
            let dragStatus = false
            let startPoint
            graphics.on('pointerover', () => {
                if (direction === 'LEFTTOP' || direction === 'RIGHTBOTTOM') {
                    this.cursorStatus = 'cursor_nwse-resize'
                } else if (direction === 'RIGHTTOP' || direction === 'LEFTBOTTOM') {
                    this.cursorStatus = 'cursor_nesw-resize'
                }
            })
            graphics.on('pointerout', () => {
                this.cursorStatus = ''
            })
            graphics.on('pointerdown', (event) => {
                dragStatus = true
                startPoint = {
                    x: event.data.getLocalPosition(this.container).x,
                    y: event.data.getLocalPosition(this.container).y
                }
                console.log(startPoint)
            })
            graphics.on('pointerup', () => {
                dragStatus = false
            })
            graphics.on('pointermove', (event) => {
                if (dragStatus) {
                    let dx = event.data.getLocalPosition(this.container).x - startPoint.x
                    let dy = event.data.getLocalPosition(this.container).y - startPoint.y
                    console.log(parent)
                    switch (direction) {
                        case 'LEFTTOP':
                            // parent.pivot.x = parent.x + parent.width
                            // parent.pivot.y = parent.y + parent.height
                            dx = -dx
                            dy = -dy
                            parent.x = event.data.getLocalPosition(this.container).x
                            parent.y = event.data.getLocalPosition(this.container).y
                            parent.width += dx
                            parent.height += dy
                            break;
                        case 'RIGHTTOP':
                            // parent.pivot.x = parent.x
                            // parent.pivot.y = parent.y + parent.height
                            dy = -dy
                            break;
                        case 'RIGHTBOTTOM':
                            // parent.pivot.x = 0
                            // parent.pivot.y = 0
                            parent.width += dx
                            parent.height += dy
                            break;
                        case 'LEFTBOTTOM':
                            // parent.pivot.x = parent.x + parent.width
                            // parent.pivot.y = parent.y
                            dx = -dx
                            break;
                    }
                    console.log('缩放位移', dx, dy, 'pivot', parent.pivot, 'position', parent.position, direction)
                    // parent.scale.x = (parent.width + dx) / parent.width
                    // parent.scale.y = (parent.height + dy) / parent.height
                    // parent.scale.set(1.2, 1.2)
                    
                    startPoint = { x: event.data.getLocalPosition(this.container).x, y: event.data.getLocalPosition(this.container).y }
                    console.log('缩放后', startPoint)
                }
            })
        }
        
        // canvas全局事件监听
        // appStageEvent (appView) {
        //     appView.on('')
        // }
    }
}
</script>
<style scoped>
.container-view {
    position: absolute;
    inset: 40px 241px 0px 0px;
    width: auto;
    height: auto;
}
.canvas-view {
    position: absolute;
    inset: 0px 0px 0px 241px;
    width: auto;
    height: 100%;
}
.layer-view {
    position: absolute;
    inset: 0px auto 0px 0px;
    width: 240px;
    height: auto;
}
.propertiesPanelContainer {
    position: absolute;
    top: 0;
    right: 0;
    width: 246px;
    bottom: 0;
}

/* 指针 */
.cursor_default {
    cursor: default !important;
}
.cursor_pointer {
    cursor: pointer !important;
}
.cursor_crosshair {
    cursor: crosshair !important;
}
.cursor_text {
    cursor: text !important;
}
.cursor_ew-resize {
    cursor: ew-resize !important;
}
.cursor_ns-resize {
    cursor: ns-resize !important;
}
.cursor_s-resize {
    cursor: s-resize !important;
}
.cursor_e_resize {
    cursor: e-resize !important;
}
.cursor_nwse-resize {
    cursor: nwse-resize !important;
}
.cursor_nesw-resize {
    cursor: nesw-resize !important;
}
</style>