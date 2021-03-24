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
                    circle.drawCircle(200, 200, 32)
                    circle.endFill()
                    // circle.x = 0
                    // circle.y = 300
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
            if (type === 'Circle') {
                let borderCir
                let startPoint
                let dragStatus = false
                graphics.on('pointerover', () => {
                    borderCir = new Graphics()
                    borderCir.beginFill(0x18a0fb)
                    borderCir.drawCircle(200, 200, 34)
                    borderCir.endFill()
                    borderCir.zIndex = 0
                    this.container.addChild(borderCir)
                })
                graphics.on('pointerout', () => {
                    borderCir.destroy()
                })
                graphics.on('pointerdown', (event) => {
                    dragStatus = true
                    startPoint = {
                        x: event.data.global.x,
                        y: event.data.global.y
                    }
                })
                graphics.on('pointerup', () => {
                    dragStatus = false
                    borderCir.destroy()
                })
                graphics.on('pointermove', (event) => {
                    if (dragStatus) {
                        const dx = event.data.global.x - startPoint.x
                        const dy = event.data.global.y - startPoint.y
                        console.log(dx, dy)
                        graphics.position.x += dx
                        graphics.position.y += dy
                        borderCir.position.x += dx
                        borderCir.position.y += dy
                        startPoint = { x: event.data.global.x, y: event.data.global.y }
                    }
                })
            }
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