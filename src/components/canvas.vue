<template>
    <div style="position: relative;flex: 1 1 auto;">
        <div class="container-view">
            <div class="canvas-view"></div>
            <div class="layer-view"></div>
        </div>
        <div class="propertiesPanelContainer"></div>
    </div>
</template>

<script>
import { Application, Container, Sprite, Graphics, Point } from 'pixi.js'

export default {
    name: 'custom-canvas',
    data () {
        return {
            app: {},
            container: {},
            canvasData: {
                id: 'blankPresets',
                x: 152,
                y: 50,
                width: 485,
                height: 975,
                type: 'Rectangle',
                scale: {
                    x: 0.8,
                    y: 0.8
                },
                zIndex: 0,
                children: [
                    {
                        id: 'image1',
                        x: 0,
                        y: 0,
                        width: 300,
                        height: 200,
                        type: 'Image',
                        url: 'http://anata.me/img/avatar.jpg',
                        zIndex: 1
                    },
                    {
                        id: 'circle1',
                        x: 400,
                        y: 400,
                        radius: 32,
                        type: 'Circle',
                        zIndex: 1
                    }
                ]
            },
            struct: new Map
        }
    },
    mounted () {
        this.app = new Application({
            resizeTo: document.getElementsByClassName('canvas-view')[0],
            backgroundColor: 0xe5e5e5,
            antialias: true
        })
        document.getElementsByClassName('canvas-view')[0].appendChild(this.app.view)
        this.parseCanvasData(this.canvasData)
    },
    methods: {
        parseCanvasData (canvasData) {
            
            // // 父容器
            this.container = new Container()
            this.container.x = canvasData.x
            this.container.y = canvasData.y
            // console.log(this.container.width, this.container.height)
            this.container.sortableChildren = true
            this.app.stage.addChild(this.container)
            
            let blankPresets = new Graphics()
            blankPresets.beginFill(0xffffff)
            blankPresets.drawRect(0, 0, canvasData.width, canvasData.height)
            blankPresets.endFill()
            blankPresets.zIndex = 0
            blankPresets.scale.set(canvasData.scale.x, canvasData.scale.y)
            this.container.addChild(blankPresets)
            // container.mask = new Graphics().beginFill(0xffffff).drawRect(20, 20, 487, 975).endFill()

            // this.struct.set()
            canvasData.children.forEach(element => {

                if (element.type === 'Image') {
                    let sprite = new Sprite.from(element.url)
                    sprite.width = 180
                    sprite.height = 120
                    sprite.interactive = true
                    sprite.buttonMode = true
                    
                    sprite.zIndex = 1
                    sprite.x = 20
                    sprite.y = 20
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
                    circle.x = 200
                    circle.y = 200
                    // circle.position.set(400, 400)
                    circle.zIndex = 1
                    circle.interactive = true
                    circle.buttonMode = true

                    this.container.addChild(circle)
                    this.bindEvent(circle, 'Circle')
                }
            })
            // this.app.stage.addChild(blankPresets)
        },
        // 绑定事件
        bindEvent (graphics, type) {
            if (type === 'Circle' || type === 'Image') {
                let borderCir
                let startPoint
                let dragStatus = false
                let line1, line2, line3, line4
                let scalePoint1, scalePoint2, scalePoint3, scalePoint4
                graphics.on('pointerover', () => {
                    if (!borderCir) {
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
                            borderCir.drawRect(0, 0, graphics.width + 4, graphics.height + 4)
                            borderCir.endFill()
                            borderCir.x = graphics.x - 2
                            borderCir.y = graphics.y - 2
                            borderCir.zIndex = 0
                            this.container.addChild(borderCir)
                        }
                        
                    }
                })
                graphics.on('pointerout', () => {
                    if (dragStatus && borderCir) return
                    dragStatus = false
                    borderCir && borderCir.destroy()
                    borderCir = null
                })
                graphics.on('pointerdown', (event) => {
                    dragStatus = true
                    startPoint = {
                        x: event.data.global.x,
                        y: event.data.global.y
                        // x: event.data.getLocalPosition(this.container).x,
                        // y: event.data.getLocalPosition(this.container).y
                    }
                    console.log(event.data.getLocalPosition(this.container))
                    console.log(graphics.position)
                })
                graphics.on('pointerup', () => {
                    dragStatus = false
                    borderCir && borderCir.destroy()
                    borderCir = null
                })
                graphics.on('pointertap', () => {
                    if (type === 'Circle') {
                        console.log('划线', graphics.width, graphics.x, graphics.y)
                        let radius = graphics.width / 2 - 2
                        line1 = this.drawDash({ x: graphics.x - radius, y: graphics.y - radius }, { x: graphics.x + radius, y: graphics.y - radius })
                        line2 = this.drawDash({ x: graphics.x + radius, y: graphics.y - radius }, { x: graphics.x + radius, y: graphics.y + radius })
                        line3 = this.drawDash({ x: graphics.x + radius, y: graphics.y + radius }, { x: graphics.x - radius, y: graphics.y + radius })
                        line4 = this.drawDash({ x: graphics.x - radius, y: graphics.y + radius }, { x: graphics.x - radius, y: graphics.y - radius })
                        
                        // scalePoint1 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y - graphics.width / 2 })
                        // scalePoint2 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y - graphics.width / 2 })
                        // scalePoint3 = this.drawCir({ x: graphics.x + graphics.width / 2, y: graphics.y + graphics.width / 2 })
                        // scalePoint4 = this.drawCir({ x: graphics.x - graphics.width / 2, y: graphics.y + graphics.width / 2 })
                        let scaleRadius1 = graphics.width / 2 - 3
                        let scaleRadius2 = graphics.width / 2 + 3
                        scalePoint1 = this.drawRect(graphics.x - scaleRadius2, graphics.y - scaleRadius2, 6, 6)
                        scalePoint2 = this.drawRect(graphics.x + scaleRadius1, graphics.y - scaleRadius2, 6, 6)
                        scalePoint3 = this.drawRect(graphics.x + scaleRadius1, graphics.y + scaleRadius1, 6, 6)
                        scalePoint4 = this.drawRect(graphics.x - scaleRadius2, graphics.y + scaleRadius1, 6, 6)
                    }
                })
                graphics.on('pointermove', (event) => {
                    if (dragStatus && borderCir) {
                        line1 && (line1.visible = false)
                        line2 && (line2.visible = false)
                        line3 && (line3.visible = false)
                        line4 && (line4.visible = false)
                        scalePoint1 && (scalePoint1.visible = false)
                        scalePoint2 && (scalePoint2.visible = false)
                        scalePoint3 && (scalePoint3.visible = false)
                        scalePoint4 && (scalePoint4.visible = false)
                        const dx = event.data.global.x - startPoint.x
                        const dy = event.data.global.y - startPoint.y
                        // const dx = event.data.getLocalPosition(this.container).x - startPoint.x
                        // const dy = event.data.getLocalPosition(this.container).y - startPoint.y
                        // console.log(dx, dy)
                        graphics.position.x += dx
                        graphics.position.y += dy
                        borderCir.position.x += dx
                        borderCir.position.y += dy
                        console.log(graphics.position.x, graphics.position.y)
                        // startPoint = { x: event.data.getLocalPosition(this.container).x, y: event.data.getLocalPosition(this.container).y }
                        startPoint = { x: event.data.global.x, y: event.data.global.y }
                    }
                })
            }
        },
        // 划线
        drawDash (startPoint, endPoint) {
            console.log(startPoint, endPoint)
            let line = new Graphics()
            line.lineStyle(1.5, 0x18a0fb, 1, 2)
            line.moveTo(startPoint.x, startPoint.y)
            line.lineTo(endPoint.x, endPoint.y)
            console.log(line.position)
            this.container.addChild(line)
            return line
        },
        // 画圆
        drawRect (x, y, width, height) {
            let rect = new Graphics()
            rect.beginFill(0xffffff)
            rect.lineStyle(1.5, 0x18a0fb, 1)
            rect.drawRect(0, 0, width, height)
            rect.endFill()
            rect.x = x
            rect.y = y
            this.container.addChild(rect)
            return rect
        }
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
</style>