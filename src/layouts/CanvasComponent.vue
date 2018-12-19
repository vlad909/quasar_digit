<template>
    <div>
        <canvas id="sheet"
                width="280px" height="280px" ref="saveCanvas"></canvas>
        <canvas width="280px" height="280px" ref="hiddenCanvas" id="hidden"></canvas>
        <canvas width="28px" height="28px" ref="smallCanvas" id="small"></canvas>
        <p>buffer: {{buffer}}</p>
        <button type="button" class="btn btn-danger" @click="knowingImageFromCanvas">wat?</button>
    </div>
</template>
<script>
    import 'fabric'
    import brain from 'brain.js'
    import trainData from '../assets/mnistTrain.json'

    export default {
        data() {
            return {
                canvas: {},
                n: new brain.NeuralNetwork(),
                buffer: ''
            }
        },
        methods: {
            getBoundingRectangle(img, threshold) {
                let rows = img.length;
                let columns = img[0].length;
                let minX = columns;
                let minY = rows;
                let maxX = -1;
                let maxY = -1;
                for (let y = 0; y < rows; y++) {
                    for (let x = 0; x < columns; x++) {
                        if (img[y][x] < threshold) {
                            if (minX > x) minX = x;
                            if (maxX < x) maxX = x;
                            if (minY > y) minY = y;
                            if (maxY < y) maxY = y;
                        }
                    }
                }
                return {minY: minY, minX: minX, maxY: maxY, maxX: maxX};
            },
            centerImage(img) {
                let meanX = 0;
                let meanY = 0;
                let rows = img.length;
                let columns = img[0].length;
                let sumPixels = 0;
                for (let y = 0; y < rows; y++) {
                    for (let x = 0; x < columns; x++) {
                        let pixel = (1 - img[y][x]);
                        sumPixels += pixel;
                        meanY += y * pixel;
                        meanX += x * pixel;
                    }
                }
                meanX /= sumPixels;
                meanY /= sumPixels;

                let dY = Math.round(rows / 2 - meanY);
                let dX = Math.round(columns / 2 - meanX);
                return {transX: dX, transY: dY};
            },
            toByteArrayIn28(grayscaleImg) {
                let nnInput = []
                for (let y = 0; y < 28; y++) {
                    for (let x = 0; x < 28; x++) {
                        let mean = 0;
                        for (let v = 0; v < 10; v++) {
                            for (let h = 0; h < 10; h++) {
                                mean += grayscaleImg[y * 10 + v][x * 10 + h];
                            }
                        }
                        mean = (1 - mean / 100); // average and invert
                        nnInput[x * 28 + y] = (mean - .5) / .5;
                    }
                }
                return nnInput
            },

            imageDataToGrayscale(imgData) {
                let grayscaleImg = [];
                for (let y = 0; y < imgData.height; y++) {
                    grayscaleImg[y] = [];
                    for (let x = 0; x < imgData.width; x++) {
                        let offset = y * 4 * imgData.width + 4 * x;
                        let alpha = imgData.data[offset + 3];
                        // weird: when painting with stroke, alpha == 0 means white;
                        // alpha > 0 is a grayscale value; in that case I simply take the R value
                        if (alpha === 0) {
                            imgData.data[offset] = 255;
                            imgData.data[offset + 1] = 255;
                            imgData.data[offset + 2] = 255;
                        }
                        imgData.data[offset + 3] = 255;
                        // simply take red channel value. Not correct, but works for
                        // black or white images.
                        grayscaleImg[y][x] = imgData.data[y * 4 * imgData.width + x * 4] / 255;
                    }
                }
                return grayscaleImg;
            },
            $28ByteCodeToImage(context /* контекст главного*/, thumbnail /* 28*28 image canv*/, nnInput) {
                let nnInput2 = []
                for (let y = 0; y < 28; y++) {
                    for (let x = 0; x < 28; x++) {
                        let block = context.getImageData(x * 10, y * 10, 10, 10);
                        let newVal = 255 * (0.5 - nnInput[x * 28 + y] / 2);
                        nnInput2.push(Math.round((255 - newVal) / 255 * 100) / 100);
                        for (let i = 0; i < 4 * 10 * 10; i += 4) {
                            block.data[i] = newVal;
                            block.data[i + 1] = newVal;
                            block.data[i + 2] = newVal;
                            block.data[i + 3] = 255;
                        }
                        context.putImageData(block, x * 10, y * 10);

                        thumbnail.data[(y * 28 + x) * 4] = newVal;
                        thumbnail.data[(y * 28 + x) * 4 + 1] = newVal;
                        thumbnail.data[(y * 28 + x) * 4 + 2] = newVal;
                        thumbnail.data[(y * 28 + x) * 4 + 3] = 255;
                    }
                }
                return {
                    thumbnail: thumbnail,
                    nnInput2: nnInput2
                }
            },
            knowingImageFromCanvas() {
                let context = this.$refs['saveCanvas'].getContext('2d')
                let image = context.getImageData(0, 0, context.canvas.height, context.canvas.width)
                let grayscaleImg = this.imageDataToGrayscale(image) //перевод полученного изображения в сервый
                let boundingRectangle = this.getBoundingRectangle(grayscaleImg, 0.01); //серое изображение ограничивается квадратом
                let trans = this.centerImage(grayscaleImg) //поиск центра массы для центрирования

                this.$refs['hiddenCanvas'].width = image.width
                this.$refs['hiddenCanvas'].height = image.height
                let copyCtx = this.$refs['hiddenCanvas'].getContext('2d') // её контекст
                let brW = boundingRectangle.maxX + 1 - boundingRectangle.minX; //ширина по центрированным значениям
                let brH = boundingRectangle.maxY + 1 - boundingRectangle.minY; //высота по центрированным значениям
                let scaling = 190 / (brW > brH ? brW : brH); // масштаб
                // масштабьирование и перенос
                copyCtx.translate(context.canvas.width / 2, context.canvas.height / 2);
                copyCtx.scale(scaling, scaling);
                copyCtx.translate(-context.canvas.width / 2, -context.canvas.height / 2);
                // translate to center of mass
                copyCtx.translate(trans.transX, trans.transY);

                copyCtx.drawImage(context.canvas, 0, 0); //орисовка изображения центрировааного в невидмой копии
                image = copyCtx.getImageData(0, 0, 280, 280);  // присвоение побитового массива сс невидимой канвы основной канве
                grayscaleImg = this.imageDataToGrayscale(image); //перевод в 255.255.255

                let nnInput = this.toByteArrayIn28(grayscaleImg) // перегон значений в 28х28

                let thumbnailCtx = this.$refs['smallCanvas'].getContext('2d') // контекст маленькой канвы
                let thumbnail = thumbnailCtx.getImageData(0, 0, 28, 28); //битовый массив маленькой канвы
                context.clearRect(0, 0, image.width, image.height) //очистка основной канвы
                context.drawImage(this.$refs['hiddenCanvas'].getContext('2d').canvas, 0, 0); //отрисовка со скрытой канвы в основной

                let resultBy28Byte = this.$28ByteCodeToImage(context, thumbnail, nnInput)
                thumbnail = resultBy28Byte.thumbnail
                let nnInput2 = resultBy28Byte.nnInput2 //получение побитвого массива 28*28
                thumbnailCtx.putImageData(thumbnail, 0, 0); //отрисовка 28*28 канвы
                console.log(this.n.run(nnInput2))
                // this.buffer = context
            }
        },
        mounted() {
            this.canvas = new fabric.Canvas('sheet')
            this.canvas.isDrawingMode = true
            this.canvas.freeDrawingBrush.width = 10
            this.canvas.freeDrawingBrush.color = "#000000";
            this.n.fromJSON(trainData)
        }
    }
</script>
<style>
    #sheet {
        border: 1px black solid !important;
    }

    #hidden, #small {
        border: 1px red solid;
    }
</style>
