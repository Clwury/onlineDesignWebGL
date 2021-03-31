<template>
    <div style="position: relative;flex: 1 1 auto;">
        <div class="container-view">
            <div class="canvas-view" :class="cursorStatus"></div>
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
                        stage: true,
                        x: 0,
                        y: 0,
                        width: 485,
                        height: 975,
                        type: 'Rectangle',
                        color: 0xffffff,
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
                        color: 0xc4c4c4,
                        scale: {
                            x: 1,
                            y: 1
                        },
                        zIndex: 1
                    },
                    {
                        id: 'rect1',
                        x: 16.5,
                        y: 16.5,
                        width: 10,
                        height: 10,
                        type: 'Rectangle',
                        color: 0xc4c4c4,
                        scale: {
                            x: 1,
                            y: 1
                        },
                        zIndex: 3
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

        document.onkeydown = (event) => {
            if (event.key === ' ') {
                this.cursorStatus = 'cursor_pointer'
            }
        }
        document.onkeyup = (event) => {
            if (event.key === ' ') {
                this.cursorStatus = ''
            }
        }
    },
    methods: {
        stagePointer () {
            console.log('按下空格')
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
                    blankPresets.beginFill(element.color)
                    blankPresets.drawRect(0, 0, element.width, element.height)
                    blankPresets.endFill()
                    blankPresets.x = element.x
                    blankPresets.y = element.y
                    blankPresets.zIndex = element.zIndex
                    blankPresets.scale.set(element.scale.x, element.scale.y)

                    if (!element.stage) {
                        blankPresets.interactive = true
                        blankPresets.buttonMode = true
                    }

                    blankPresets.name = element.id
                    blankPresets.type = element.type
                    console.log(blankPresets)
                    this.container.addChild(blankPresets)
                    this.bindEvent(blankPresets)
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
                    sprite.type = element.type
                    this.container.addChild(sprite)
                    this.bindEvent(sprite)
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
                    circle.type = element.type
                    this.container.addChild(circle)
                    this.bindEvent(circle)
                }
            })
            // this.app.stage.addChild(blankPresets)
        },
        // 绑定事件
        bindEvent (graphics) {
            if (graphics.type === 'Circle' || graphics.type === 'Image' || graphics.type === 'Rectangle') {
                let borderCir
                let startPoint
                let dragStatus = false
                console.log('注册组件id', graphics.name)
                // let line1, line2, line3, line4
                // let scalePoint1, scalePoint2, scalePoint3, scalePoint4
                graphics.on('pointerover', () => {
                    console.log('指针移入')
                    if (!borderCir && this.focusedGraphics.name !== graphics.name) {
                        if (graphics.type === 'Circle') {
                            borderCir = new Graphics()
                            borderCir.beginFill(0x18a0fb)
                            borderCir.drawCircle(0, 0, 33.5)
                            borderCir.endFill()
                            borderCir.x = graphics.x
                            borderCir.y = graphics.y
                            borderCir.zIndex = 0
                            this.container.addChild(borderCir)
                        } else if (graphics.type === 'Image' || graphics.type === 'Rectangle') {
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
                    this.focusGraphics(graphics)
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
                        this.focusGraphics(graphics)
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
        drawDash (startPoint, endPoint, direction) {
            // console.log(startPoint, endPoint)
            let line = new Graphics()
            line.lineStyle(1, 0x18a0fb, 1, 1)
            line.moveTo(startPoint.x, startPoint.y)
            line.lineTo(endPoint.x, endPoint.y)
            // line.x = 0
            // line.y = 0
            line.start = startPoint
            line.end = endPoint
            line.direction = direction
            console.log(line.position)
            // console.log(line.position, line.width, line.height)
            // this.container.addChild(line)
            return line
        },
        // 画矩形
        drawRect (x, y, width, height, direction, parent) {
            let rect = new Graphics()
            rect.beginFill(0xffffff)
            rect.lineStyle(1, 0x18a0fb, 1)
            rect.drawRect(0, 0, width, height)
            rect.endFill()
            rect.x = x
            rect.y = y
            rect.direction = direction
            rect.zIndex = 2
            // rect.centerPoint = { x: rect.x + rect.width / 2, y: rect.y + rect.height }
            console.log('矩形宽高', rect.width, rect.height, rect.x, rect.y)
            // console.log('中心点', rect.x + rect.width / 2, rect.y + rect.height / 2, '父中心点', parent.x, parent.y)
            switch (direction) {
                case 'LEFTTOP':
                    rect.symmetryDirection = 'RIGHTBOTTOM'
                    break;
                case 'RIGHTTOP':
                    rect.symmetryDirection = 'LEFTBOTTOM'
                    break;
                case 'RIGHTBOTTOM':
                    rect.symmetryDirection = 'LEFTTOP'
                    break;
                case 'LEFTBOTTOM':
                    rect.symmetryDirection = 'RIGHTTOP'
                    break;
            }
            rect.interactive = true
            this.bindScaleEvent(rect, direction, parent)
            // this.container.addChild(rect)
            return rect
        },
        // 绘制可拖动区域
        // drawWhirlArea (x, y, direction) {
        //     let container1 = new Container()

        // },
        // 绘制聚焦边框(圆形)
        drawFocusBorderForCircle (graphics) {
            let borderArr = []
            // console.log('绘制边框', graphics.width, graphics.height, graphics.x, graphics.y)
            let dx = graphics.width / 2
            let dy = graphics.height / 2
            let line1 = this.drawDash({ x: graphics.x - dx, y: graphics.y - dy }, { x: graphics.x + dx, y: graphics.y - dy }, 'TOP')
            let line2 = this.drawDash({ x: graphics.x + dx, y: graphics.y - dy }, { x: graphics.x + dx, y: graphics.y + dy }, 'RIGHT')
            let line3 = this.drawDash({ x: graphics.x + dx, y: graphics.y + dy }, { x: graphics.x - dx, y: graphics.y + dy }, 'BOTTOM')
            let line4 = this.drawDash({ x: graphics.x - dx, y: graphics.y + dy }, { x: graphics.x - dx, y: graphics.y - dy }, 'LEFT')
            
            // scalePoint1 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y - graphics.width / 2 })
            // scalePoint2 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y - graphics.width / 2 })
            // scalePoint3 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y + graphics.width / 2 })
            // scalePoint4 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y + graphics.width / 2 })
            dx = graphics.width / 2 - 3.5 + 1
            dy = graphics.height / 2 + 3.5
            let dm = graphics.width / 2 + 3.5
            let dn = graphics.height / 2 - 3.5 + 1
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
            // let d = 2
            let line1 = this.drawDash({ x: graphics.x, y: graphics.y }, { x: graphics.x + graphics.width, y: graphics.y },'TOP')
            let line2 = this.drawDash({ x: graphics.x + graphics.width, y: graphics.y }, { x: graphics.x + graphics.width, y: graphics.y + graphics.height }, 'RIGHT')
            let line3 = this.drawDash({ x: graphics.x + graphics.width, y: graphics.y + graphics.height }, { x: graphics.x, y: graphics.y + graphics.height }, 'BOTTOM')
            let line4 = this.drawDash({ x: graphics.x, y: graphics.y + graphics.height }, { x: graphics.x, y: graphics.y }, 'LEFT')
            
            let d = 3.5
            let scalePoint1 = this.drawRect(graphics.x - d, graphics.y - d, 6, 6, 'LEFTTOP', graphics)
            let scalePoint2 = this.drawRect(graphics.x + graphics.width - d + 1, graphics.y - d, 6, 6, 'RIGHTTOP', graphics)
            let scalePoint3 = this.drawRect(graphics.x + graphics.width - d + 1, graphics.y + graphics.height - d + 1, 6, 6, 'RIGHTBOTTOM', graphics)
            let scalePoint4 = this.drawRect(graphics.x - d, graphics.y + graphics.height - d + 1, 6, 6, 'LEFTBOTTOM', graphics)
            borderArr.push(line1, line2, line3, line4, scalePoint1, scalePoint2, scalePoint3, scalePoint4)
            return borderArr
        },
        // 元素聚焦
        focusGraphics (graphics) {
            console.log('事件对象', graphics.name)
            let borderArr = []
            if (graphics.type === 'Circle') {
                borderArr = this.drawFocusBorderForCircle(graphics)
            } else if (graphics.type === 'Image' || graphics.type === 'Rectangle') {
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
            graphics.on('pointerdown', () => {
                dragStatus = true
                // startPoint = {
                    // x: event.data.getLocalPosition(this.container).x,
                    // y: event.data.getLocalPosition(this.container).y
                    // x: graphics.centerPoint.x,
                    // y: graphics.centerPoint.y
                // }
                switch (direction) {
                    case 'LEFTTOP':
                        if (parent.type === 'Rectangle' || parent.type === 'Image') {
                            startPoint = {
                                x: parent.x,
                                y: parent.y
                            }
                        } else if (parent.type === 'Circle') {
                            startPoint = {
                                x: parent.x - parent.width / 2,
                                y: parent.y - parent.height / 2
                            }
                        }

                        break;
                    case 'RIGHTTOP':
                        if (parent.type === 'Rectangle' || parent.type === 'Image') {
                            startPoint = {
                                x: parent.x + parent.width,
                                y: parent.y
                            }
                        } else if (parent.type === 'Circle') {
                            startPoint = {
                                x: parent.x + parent.width / 2,
                                y: parent.y - parent.height / 2
                            }
                        }
                        
                        break;
                    case 'RIGHTBOTTOM':
                        if (parent.type === 'Rectangle' || parent.type === 'Image') {
                            startPoint = {
                                x: parent.x + parent.width,
                                y: parent.y + parent.height
                            }
                        } else if (parent.type === 'Circle') {
                            startPoint = {
                                x: parent.x + parent.width / 2,
                                y: parent.y + parent.height / 2
                            }
                        }
                        
                        break;
                    case 'LEFTBOTTOM':
                        if (parent.type === 'Rectangle' || parent.type === 'Image') {
                            startPoint = {
                                x: parent.x,
                                y: parent.y + parent.height
                            }
                        } else if (parent.type === 'Circle') {
                            startPoint = {
                                x: parent.x - parent.width / 2,
                                y: parent.y + parent.height / 2
                            }
                        }
                        
                        break;
                
                }
                console.log(startPoint)
            })
            graphics.on('pointerup', () => {
                dragStatus = false
            })
            graphics.on('pointermove', (event) => {
                if (dragStatus) {
                    let currentPoint = {
                        x: event.data.getLocalPosition(this.container).x,
                        y: event.data.getLocalPosition(this.container).y
                    }
                    // console.log(currentPoint)
                    let dx = currentPoint.x - startPoint.x
                    let dy = currentPoint.y - startPoint.y
                    console.log(parent)
                    switch (direction) {
                        case 'LEFTTOP':
                            // parent.pivot.x = parent.x + parent.width
                            // parent.pivot.y = parent.y + parent.height
                            // dx = -dx
                            // dy = -dy
                            parent.width -= dx
                            parent.height -= dy
                            break;
                        case 'RIGHTTOP':
                            // parent.pivot.x = parent.x
                            // parent.pivot.y = parent.y + parent.height
                            parent.width += dx
                            parent.height -= dy
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
                            parent.width -= dx
                            parent.height += dy
                            break;
                    }
                    this.computeGraphicsScaledPosition(parent, currentPoint, direction)
                    this.computeBorderScaledPosition(parent, direction, dx, dy)
                    console.log('缩放位移', dx, dy, 'pivot', parent.pivot, 'position', parent.position, direction)
                    // parent.scale.x = (parent.width + dx) / parent.width
                    // parent.scale.y = (parent.height + dy) / parent.height
                    // parent.scale.set(1.2, 1.2)

                    startPoint = currentPoint
                    console.log('缩放后', startPoint)
                }
            })
        },
        // 计算缩放后位置
        computeGraphicsScaledPosition (graphics, currentPoint, direction) {
            if (graphics.type === 'Rectangle' || graphics.type === 'Image') {
                switch (direction) {
                    case 'LEFTTOP':
                        graphics.x = currentPoint.x
                        graphics.y = currentPoint.y
                        break;
                    case 'RIGHTTOP':
                        graphics.x = currentPoint.x - graphics.width
                        graphics.y = currentPoint.y
                        break;
                    case 'RIGHTBOTTOM':
                        graphics.x = currentPoint.x - graphics.width
                        graphics.y = currentPoint.y - graphics.height
                        break;
                    case 'LEFTBOTTOM':
                        graphics.x = currentPoint.x
                        graphics.y = currentPoint.y - graphics.height
                        break;
                }
            } else if (graphics.type === 'Circle') {
                switch (direction) {
                    case 'LEFTTOP':
                        graphics.x = currentPoint.x + graphics.width / 2
                        graphics.y = currentPoint.y + graphics.height / 2
                        break;
                    case 'RIGHTTOP':
                        graphics.x = currentPoint.x - graphics.width / 2
                        graphics.y = currentPoint.y + graphics.height / 2
                        break;
                    case 'RIGHTBOTTOM':
                        graphics.x = currentPoint.x - graphics.width / 2
                        graphics.y = currentPoint.y - graphics.height / 2
                        break;
                    case 'LEFTBOTTOM':
                        graphics.x = currentPoint.x + graphics.width / 2
                        graphics.y = currentPoint.y - graphics.height / 2
                        break;
                }
            }
        },
        // 计算边框缩放位置
        computeBorderScaledPosition (graphics, direction, dx, dy) {
            let alignment = 1
            let d = this.focusBorder.get(graphics.name)
            for (let index = 4; index < d.length; index++) {
                const e = d[index]
                if (e.symmetryDirection === direction) {
                    continue
                }
                // console.log('需要更新的触点', e.symmetryDirection, direction)
                this.computePoint(e, direction, dx, dy)
            }
            for (let i = 0; i < 4; i++) {
                let l = d[i]
                const start = l.start
                const end = l.end
                const lineDirection = l.direction
                l.destroy() && (l = null)
                d[i] = this.computeLine(start, end, lineDirection, direction, dx, dy, alignment)
                this.container.addChild(d[i])
            }
        },
        // 线段
        computeLine (beforeStart, beforeEnd, lineDirection, direction, dx, dy, alignment) {
            switch (direction) {
                case 'LEFTTOP':
                    console.log('线', beforeStart, beforeEnd)
                    if (lineDirection === 'RIGHT') {
                        let line2 = new Graphics()
                        line2.lineStyle(1, 0x18a0fb, 1, alignment)
                        line2.start = { x: beforeStart.x, y: beforeStart.y + dy }
                        line2.end = { x: beforeEnd.x, y: beforeEnd.y }
                        line2.direction = lineDirection
                        line2.moveTo(line2.start.x, line2.start.y)
                        line2.lineTo(line2.end.x, line2.end.y)
                        return line2
                    }
                    if (lineDirection === 'BOTTOM') {
                        let line3 = new Graphics()
                        line3.lineStyle(1, 0x18a0fb, 1, alignment)
                        line3.start = { x: beforeStart.x, y: beforeStart.y }
                        line3.end = { x: beforeEnd.x + dx, y: beforeEnd.y }
                        line3.direction = lineDirection
                        line3.moveTo(line3.start.x, line3.start.y)
                        line3.lineTo(line3.end.x, line3.end.y)
                        return line3
                    }
                    if (lineDirection === 'LEFT') {
                        let line4 = new Graphics()
                        line4.lineStyle(1, 0x18a0fb, 1, alignment)
                        line4.start = { x: beforeStart.x + dx, y: beforeStart.y }
                        line4.end = { x: beforeEnd.x + dx, y: beforeEnd.y + dy }
                        line4.direction = lineDirection
                        line4.moveTo(line4.start.x, line4.start.y)
                        line4.lineTo(line4.end.x, line4.end.y)
                        return line4
                    }
                    if (lineDirection === 'TOP') {
                        let line1 = new Graphics()
                        line1.lineStyle(1, 0x18a0fb, 1, alignment)
                        line1.start = { x: beforeStart.x + dx, y: beforeStart.y + dy }
                        line1.end = { x: beforeEnd.x, y: beforeEnd.y + dy }
                        line1.direction = lineDirection
                        line1.moveTo(line1.start.x, line1.start.y)
                        line1.lineTo(line1.end.x, line1.end.y)
                        return line1
                    }
                    break;
                case 'RIGHTTOP':
                    if (lineDirection === 'LEFT') {
                        let line4 = new Graphics()
                        line4.lineStyle(1, 0x18a0fb, 1, alignment)
                        line4.start = { x: beforeStart.x, y: beforeStart.y }
                        line4.end = { x: beforeEnd.x, y: beforeEnd.y + dy }
                        line4.direction = lineDirection
                        line4.moveTo(line4.start.x, line4.start.y)
                        line4.lineTo(line4.end.x, line4.end.y)
                        return line4
                    }
                    if (lineDirection === 'BOTTOM') {
                        let line3 = new Graphics()
                        line3.lineStyle(1, 0x18a0fb, 1, alignment)
                        line3.start = { x: beforeStart.x + dx, y: beforeStart.y }
                        line3.end = { x: beforeEnd.x, y: beforeEnd.y }
                        line3.direction = lineDirection
                        line3.moveTo(line3.start.x, line3.start.y)
                        line3.lineTo(line3.end.x, line3.end.y)
                        return line3
                    }
                    if (lineDirection === 'RIGHT') {
                        let line2 = new Graphics()
                        line2.lineStyle(1, 0x18a0fb, 1, alignment)
                        line2.start = { x: beforeStart.x + dx, y: beforeStart.y + dy }
                        line2.end = { x: beforeEnd.x + dx, y: beforeEnd.y }
                        line2.direction = lineDirection
                        line2.moveTo(line2.start.x, line2.start.y)
                        line2.lineTo(line2.end.x, line2.end.y)
                        return line2
                    }
                    if (lineDirection === 'TOP') {
                        let line1 = new Graphics()
                        line1.lineStyle(1, 0x18a0fb, 1, alignment)
                        line1.start = { x: beforeStart.x, y: beforeStart.y + dy }
                        line1.end = { x: beforeEnd.x + dx, y: beforeEnd.y + dy }
                        line1.direction = lineDirection
                        line1.moveTo(line1.start.x, line1.start.y)
                        line1.lineTo(line1.end.x, line1.end.y)
                        return line1
                    }
                    break;
                case 'RIGHTBOTTOM':
                    if (lineDirection === 'LEFT') {
                        let line4 = new Graphics()
                        line4.lineStyle(1, 0x18a0fb, 1, alignment)
                        line4.start = { x: beforeStart.x, y: beforeStart.y + dy }
                        line4.end = { x: beforeEnd.x, y: beforeEnd.y }
                        line4.direction = lineDirection
                        line4.moveTo(line4.start.x, line4.start.y)
                        line4.lineTo(line4.end.x, line4.end.y)
                        return line4
                    }
                    if (lineDirection === 'TOP') {
                        let line1 = new Graphics()
                        line1.lineStyle(1, 0x18a0fb, 1, alignment)
                        line1.start = { x: beforeStart.x, y: beforeStart.y }
                        line1.end = { x: beforeEnd.x + dx, y: beforeEnd.y }
                        line1.direction = lineDirection
                        line1.moveTo(line1.start.x, line1.start.y)
                        line1.lineTo(line1.end.x, line1.end.y)
                        return line1
                    }
                    if (lineDirection === 'RIGHT') {
                        let line2 = new Graphics()
                        line2.lineStyle(1, 0x18a0fb, 1, alignment)
                        line2.start = { x: beforeStart.x + dx, y: beforeStart.y }
                        line2.end = { x: beforeEnd.x + dx, y: beforeEnd.y + dy }
                        line2.direction = lineDirection
                        line2.moveTo(line2.start.x, line2.start.y)
                        line2.lineTo(line2.end.x, line2.end.y)
                        return line2
                    }
                    if (lineDirection === 'BOTTOM') {
                        let line3 = new Graphics()
                        line3.lineStyle(1, 0x18a0fb, 1, alignment)
                        line3.start = { x: beforeStart.x + dx, y: beforeStart.y + dy }
                        line3.end = { x: beforeEnd.x, y: beforeEnd.y + dy }
                        line3.direction = lineDirection
                        line3.moveTo(line3.start.x, line3.start.y)
                        line3.lineTo(line3.end.x, line3.end.y)
                        return line3
                    }
                    break;
                case 'LEFTBOTTOM':
                    if (lineDirection === 'RIGHT') {
                        let line2 = new Graphics()
                        line2.lineStyle(1, 0x18a0fb, 1, alignment)
                        line2.start = { x: beforeStart.x, y: beforeStart.y }
                        line2.end = { x: beforeEnd.x, y: beforeEnd.y + dy }
                        line2.direction = lineDirection
                        line2.moveTo(line2.start.x, line2.start.y)
                        line2.lineTo(line2.end.x, line2.end.y)
                        return line2
                    }
                    if (lineDirection === 'TOP') {
                        let line1 = new Graphics()
                        line1.lineStyle(1, 0x18a0fb, 1, alignment)
                        line1.start = { x: beforeStart.x + dx, y: beforeStart.y }
                        line1.end = { x: beforeEnd.x, y: beforeEnd.y }
                        line1.direction = lineDirection
                        line1.moveTo(line1.start.x, line1.start.y)
                        line1.lineTo(line1.end.x, line1.end.y)
                        return line1
                    }
                    if (lineDirection === 'LEFT') {
                        let line4 = new Graphics()
                        line4.lineStyle(1, 0x18a0fb, 1, alignment)
                        line4.start = { x: beforeStart.x + dx, y: beforeStart.y + dy }
                        line4.end = { x: beforeEnd.x + dx, y: beforeEnd.y }
                        line4.direction = lineDirection
                        line4.moveTo(line4.start.x, line4.start.y)
                        line4.lineTo(line4.end.x, line4.end.y)
                        return line4
                    }
                    if (lineDirection === 'BOTTOM') {
                        let line3 = new Graphics()
                        line3.lineStyle(1, 0x18a0fb, 1, alignment)
                        line3.start = { x: beforeStart.x, y: beforeStart.y + dy }
                        line3.end = { x: beforeEnd.x + dx, y: beforeEnd.y + dy }
                        line3.direction = lineDirection
                        line3.moveTo(line3.start.x, line3.start.y)
                        line3.lineTo(line3.end.x, line3.end.y)
                        return line3
                    }
                    break;
            }
        },
        // 触点
        computePoint (graphics, direction, dx, dy) {
            switch (direction) {
                case 'LEFTTOP':
                    if (graphics.direction === 'LEFTTOP') {
                        graphics.x += dx
                        graphics.y += dy
                    }
                    if (graphics.direction === 'RIGHTTOP') {
                        graphics.y += dy
                    }
                    if (graphics.direction === 'LEFTBOTTOM') {
                        graphics.x += dx
                    }
                    break;
                case 'RIGHTTOP':
                    if (graphics.direction === 'RIGHTTOP') {
                        graphics.x += dx
                        graphics.y += dy
                    }
                    if (graphics.direction === 'LEFTTOP') {
                        graphics.y += dy
                    }
                    if (graphics.direction === 'RIGHTBOTTOM') {
                        graphics.x += dx
                    }
                    break;
                case 'RIGHTBOTTOM':
                    if (graphics.direction === 'RIGHTBOTTOM') {
                        graphics.x += dx
                        graphics.y += dy
                    }
                    if (graphics.direction === 'RIGHTTOP') {
                        graphics.x += dx
                    }
                    if (graphics.direction === 'LEFTBOTTOM') {
                        graphics.y += dy
                    }
                    break;
                case 'LEFTBOTTOM':
                    if (graphics.direction === 'LEFTBOTTOM') {
                        graphics.x += dx
                        graphics.y += dy
                    }
                    if (graphics.direction === 'LEFTTOP') {
                        graphics.x += dx
                    }
                    if (graphics.direction === 'RIGHTBOTTOM') {
                        graphics.y += dy
                    }
                    break;
            }
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
.curson_rt-scale {
    cursor:url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAEJElEQVRYR+2Wb0hbZxTGn0Rr7J9ETNdcMsUMw/yDk2lkiJC4DAY6FZ2MyWDOeTdB7GTCpkNwI7FTYTAUNYQxtkCjDKfSD04NhoFK/BAWiJ1mDiWwaOJsu8YuumprbDqO3EJLTaLbFb/0wv2S9715f/c5z3nOFeCUL8Epn49nAM8UOIoCtCfG5/NlJScn/w7gAXfz4t9oALQuqqmpuWgymX7d2dmZkEgkjQB2AezzQRAJQEiHT0xMqEtKSqx1dXU3GhsbxVKp9HuFQqEHcJeDCP0fkHAAB4c7nc63c3Nzr5aXl28uLS2J3W73GTosEAjM+v3+sZGRkdHW1tZbAIL/tSzhAM5MTU29otForlZUVFzyeDznJRJJMD4+/iHDMHtZWVlBrVYbr1Kp7oVCoYmxsbEulmXXAOwBeHgcRcIBiABI5+fnryQlJb2rVCpF29vbwt7e3k2n0xnyeDznVldXz+7u7j5gWfZme3s7XC7Xp3l5eT8BuH8cNcIqACARQPLMzEyLRCIprqysvFBcXBwwGAxir9e7wjBMitFovKvX6+WpqambCwsLUq/X25GSkvIVZ1LqlqhXJA+cA/AcgKTZ2dmP9/f3X8vPz49RqVSfrKysBGpraxXV1dWatLS0V1mWFdvt9ji/3y9wOBzvazSaawB2AEQ1aKQuiAVAEBcByBwOx2Wz2Tzd39//G4B7AGg9Qa/Xq3U63ZdlZWV3FhcXxdPT07Fut7u8qKjoZ25fRE9Ey4EYAOcBiDkYkpXejOpMa/S7rLS09KXx8fHvGIbZz8zMvG+xWG51dXW90dHR8QdnzLCliAZAD9JBcdwbk6TUchRC9Gw85xW5xWL5aG1trby+vj5xYGDgdnZ29tc5OTlGAP9E6oyjABAE7aOb5HxcUsoLgiCvpHi9XmNnZ+fzPp9PaDKZrstksncA3OGgD1XhqACR3EwQFwDIR0dHP/T7/e/pdDrp+vp6THd39+stLS2/cGU7MQD6Y8qNSwBeWF5enkxPTxcHAgFsbGx8k5GR8QWAv8PNDj4UOPBCQ0PDiz09PVaRSMQIBALMzc39FRsbG5LL5T8qFAqCoNnxVDbwARBjtVpz1Wr1QHNzc6JQKBQZjcaEoaGh221tbQeGVCgUP8jl8s8Pa0s+AKhDEl0u1xWBQFBVVVUl3tvbC8XFxQlUKtWW2WyWFhYWvmyz2agln+oIPgBoQiZQbNtsts+USuWbSqXyrFar3ZqcnJQ0NTV90NfXNwvgJmfGJ4KJDwDqgkeJmUyxXVBQ8FYwGNwaHh7+lmXZIQB/ckakafnExQfAo7CixKQ8YOx2++XBwcEZg8FwHcANAJvhYpkvgMchqBykCDmear7FHX7oYOITgCBoQJEn6KZak+QU3WGnIt8AkWL7RJMw6odHuA0nocCxYE4d4F/thnAw5DSHgQAAAABJRU5ErkJggg==")15 15,auto !important;
}

</style>