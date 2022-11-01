<template>
    <div class="container flex-grow-1 mt-5 mb-5 py-2">
        <div class="col-md-4 col-sm-12 mx-auto">
            <div class="card">
                <div class="card-header">
                    <h4>
                        Faça o upload dos arquivos aqui
                    </h4>
                </div>
                <div class="card-body">
                    <form>
                        <div class="buttons">
                            <label for="fileLabel">Label</label><br>
                            <input type="file" class="btn" id="fileLabel" name="fileLabel" @change="handleFileUpload($event)" accept=".txt" />
                            <br><br>
                            <label for="fileImage">Imagem</label><br>
                            <input type="file" class="btn" id="fileImage" name="fileImage" @change="handleFileUpload($event)" accept=".jpg, .png" />
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
    <div class="submit">
        <button v-on:click="submitFiles()" class="btn-primary">Submit</button>
    </div>
    <div class="canva">
        <canvas ref="canvas" width="2000" height="2000"></canvas>
    </div>
    <div class="zoom">
        <button v-on:click="imageZoomin()" id="zoomin" class="btn-primary">+</button>
        <button v-on:click="imageZoomout()" id="zoomout" class="btn-primary">-</button>
    </div>
</template>

<script>
import { fabric } from 'fabric'
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import json from '/public/classes.json'

export default {
    data() {
        return {
            canvasOriginalHeight: 1080,
            canvasOriginalWidth: 1920,
            canvasWidth: 1920,
            canvasHeight: 1080,
            canvasScale: 1,
            classFile: json,
            canvas: null,
            canvasObj: null,
            seletedImage: '',
            componentKey: 0,
            images: [],
            fileConfig: null,
            imageFile: null,
            imageFileName: null,
            configFileName: null,
            bgImageOrig: null,
            currentImageIndex: null,
        };
    },
    mounted() {
        this.canvas = this.$refs.canvas;
        this.canvasObj = new fabric.Canvas(this.canvas);

        this.setCanvasZoom();
        this.setCanvasSize({ height: this.canvasHeight, width: this.canvasWidth });
    },
    methods: {
        async handleFileUpload(event) {
            const file = event.target.files[0];

            console.log(this.classFile[10]);

            if (file) {
                let fileName = file.name;
                let fileType = file.type;

                if (fileType === "text/plain") {
                    this.fileConfig = await file.text();
                    this.configFileName = fileName;
                    console.log(this.fileConfig);
                }
                else if (fileType === "image/jpeg" || fileType === "image/png") {
                    this.imageFileName = fileName;
                    // Process image file
                    this.imageFile = URL.createObjectURL(file);

                }
            }
        },

        handleImageConfigs() {
            // Process file with points
            const indexLastImg = this.images.length - 1;
            var lines = this.fileConfig.split('\n');

            lines.forEach(l => {
                if (l != '') {
                    var values = l.split(' ');
                    if (values.length != 6) {
                        console.log("Line error: " + l);
                    } else {
                        console.log(values);
                        var point = {};
                        point.anotation = this.classFile[values[0]].classe;
                        point.color = '#' + this.classFile[values[0]].cor;
                        point.anotationCode = values[0];
                        point.x1 = parseInt(values[1]);
                        point.y1 = parseInt(values[2]);
                        point.x2 = parseInt(values[3]);
                        point.y2 = parseInt(values[4]);
                        this.images[indexLastImg].points.push(point);
                    }
                }
            });
 
        },

        removeCanvasObjects(canvas) {
            var objects = canvas.getObjects();
            for (var i = 0; i < objects.length; i++) {
                canvas.remove(objects[i]);
            }
        },

        setCanvasSize(canvasSizeObject) {
            let canvas = this.canvasObj;
            canvas.setWidth(canvasSizeObject.width);
            canvas.setHeight(canvasSizeObject.height);
        },

        setCanvasZoom() {
            let canvas = this.canvasObj;
            this.canvasWidth = this.canvasOriginalWidth * this.canvasScale;
            this.canvasHeight = this.canvasOriginalHeight * this.canvasScale;

            canvas.setWidth(this.canvasWidth);
            canvas.setHeight(this.canvasHeight);
        },

        setCanvasBackgroundImageUrl(url) {
            let canvas = this.canvasObj;
            if (url && url.length > 0) {
                fabric.Image.fromURL(url, function (img) {
                    bgImage = img;
                    scaleAndPositionImage();
                });
            } else {
                canvas.backgroundImage = 0;
                canvas.setBackgroundImage('', canvas.renderAll.bind(canvas));

                canvas.renderAll();
            }
        },

        scaleAndPositionImage(bgImage) {
            let canvas = this.canvasObj;
            this.setCanvasZoom();

            var canvasAspect = this.canvasWidth / this.canvasHeight;
            var imgAspect = bgImage.width / bgImage.height;
            var left, top, scaleFactor;

            if (canvasAspect >= imgAspect) {
                var scaleFactor = this.canvasWidth / bgImage.width;
                left = 0;
                top = -((bgImage.height * scaleFactor) - this.canvasHeight) / 2;
            } else {
                var scaleFactor = this.canvasHeight / bgImage.height;
                top = 0;
                left = -((bgImage.width * scaleFactor) - this.canvasWidth) / 2;
            }

            canvas.setBackgroundImage(bgImage, canvas.renderAll.bind(canvas), {
                top: top,
                left: left,
                originX: 'left',
                originY: 'top',
                scaleX: scaleFactor,
                scaleY: scaleFactor
            });
            canvas.renderAll();
        },

        handleImgSelection(imageIndex) {
            this.componentKey += 1;
            let canvas = this.canvasObj;
            let points = this.images[imageIndex].points;
            this.currentImageIndex = imageIndex;
            let scaleAndPositionImageRef = this.scaleAndPositionImage;

            this.removeCanvasObjects(canvas);

            let bgImage = Object.assign({}, this.bgImageOrig);
            let canvasScale = this.canvasScale;

            fabric.Image.fromURL(this.images[imageIndex].file, function (img) {
                bgImage = img;
                scaleAndPositionImageRef(img);

                for (var i = 0; i < points.length; i++) {
                    var point = points[i];

                    var ret = new fabric.Rect({
                        width: (point.x2 - point.x1) * canvasScale,
                        height: (point.y2 - point.y1) * canvasScale,
                        fill: '',
                        left: point.x1 * canvasScale,
                        top: point.y1 * canvasScale,
                        stroke: point.color,
                        strokeWidth: 4,
                        hoverCursor: 'default'
                    });
                    canvas.add(ret);

                    // Create a new Textbox instance
                    var text = new fabric.Text(point.anotation, {
                        fontSize: 20,
                        fill: 'white',
                        fontFamily: 'Arial',
                        fontWeight: 'bold',
                        left: (point.x1) * canvasScale,
                        top: (point.y1 - 20) * canvasScale,
                    });
                    canvas.add(text);
                }

            });

            this.canvasObj.renderAll();
        },

        submitFiles() {
            if (this.fileConfig == null) {
                alert("Selecione arquivo");
                return;
            }
            if (this.imageFile == null) {
                alert("Selecione imagem");
                return;
            }
            if (this.imageFileName.substr(0, this.imageFileName.lastIndexOf(".")) !=
                this.configFileName.substr(0, this.configFileName.lastIndexOf("."))) {
                alert("O arquivo selecionado e imagem nao correspondem");
                return;
            }

            this.images.push({
                file: this.imageFile,
                name: this.imageFileName,
                points: [],
            });

            this.handleImageConfigs();

            this.fileConfig = null;
            this.imageFile = null;

            this.handleImgSelection(0);
        },

        imageZoomin() {
            this.canvasScale *= 1.25;
            this.handleImgSelection(this.currentImageIndex);
        },

        imageZoomout() {
            this.canvasScale /= 1.25;
            this.handleImgSelection(this.currentImageIndex);
        },
    }
}
</script>