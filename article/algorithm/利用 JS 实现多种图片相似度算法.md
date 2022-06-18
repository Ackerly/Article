# 利用 JS 实现多种图片相似度算法
**特征提取算法**  
每种算法都会经过“特征提取”和“特征比对”两个步骤进行。  
**平均哈希算法**  
“平均哈希算法”主要由以下几步组成：  
第一步，缩小尺寸为8×8，以去除图片的细节，只保留结构、明暗等基本信息，摒弃不同尺寸、比例带来的图片差异。  
第二步，简化色彩。将缩小后的图片转为灰度图像。    
第三步，计算平均值。计算所有像素的灰度平均值。  
第四步，比较像素的灰度。将64个像素的灰度，与平均值进行比较。大于或等于平均值，记为1；小于平均值，记为0。  
第五步，计算哈希值。将上一步的比较结果，组合在一起，就构成了一个64位的整数，这就是这张图片的指纹。  
第六步，计算哈希值的差异，得出相似度（汉明距离或者余弦值）  

**图片压缩**  
采用 canvas 的 drawImage() 方法实现图片压缩，后使用 getImageData() 方法获取 ImageData 对象  
``` 
export function compressImg (imgSrc: string, imgWidth: number= 8): Promise<ImageData> {
return new Promise((resolve, reject) => {
if (!imgSrc) {
    reject('imgSrc can not be empty!')
}
const canvas = document.createElement('canvas')
const ctx = canvas.getContext('2d')
const img = new Image()
img.crossOrigin = 'Anonymous'
img.onload = function () {
```
为什么使用 canvas 可以实现图片压缩呢？简单来说，为了把“大图片”绘制到“小画布”上，一些相邻且颜色相近的像素往往会被删减掉，从而有效减少了图片的信息量，因此能够实现压缩的效果：  
在上面的 compressImg() 函数中，我们利用 new Image() 加载图片，然后设定一个预设的图片宽高值让图片压缩到指定的大小，最后获取到压缩后的图片的 ImageData 数据——这也意味着我们能获取到图片的每一个像素的信息。  
**图片灰度化**  
灰度图像:  
在计算机领域中，灰度（Gray scale）数字图像是每个像素只有一个采样颜色的图像。大部分情况下，任何的颜色都可以通过三种颜色通道（R, G, B）的亮度以及一个色彩空间（A）来组成，而一个像素只显示一种颜色，因此可以得到“像素 => RGBA”的对应关系。而“每个像素只有一个采样颜色”，则意味着组成这个像素的三原色通道亮度相等，因此只需要算出 RGB 的平均值即可：  
``` 
// 根据 RGBA 数组生成 ImageData
export function createImgData (dataDetail: number[]) {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  const imgWidth = Math.sqrt(dataDetail.length / 4)
  const newImageData = ctx?.createImageData(imgWidth, imgWidth) as ImageData
  for (let i = 0; i < dataDetail.length; i += 4) {
    let R = dataDetail[i]
    let G = dataDetail[i + 1]
    let B = dataDetail[i + 2]
    let Alpha = dataDetail[i + 3]
    newImageData.data[i] = R
    newImageData.data[i + 1] = G
    newImageData.data[i + 2] = B
    newImageData.data[i + 3] = Alpha
  }
  return newImageData
}
export function createGrayscale (imgData: ImageData) {
  const newData: number[] = Array(imgData.data.length)
  newData.fill(0)
  imgData.data.forEach((_data, index) => {
    if ((index + 1) % 4 === 0) {
      const R = imgData.data[index - 3]
      const G = imgData.data[index - 2]
      const B = imgData.data[index - 1]
      const gray = ~~((R + G + B) / 3)
      newData[index - 3] = gray
      newData[index - 2] = gray
      newData[index - 1] = gray
      newData[index] = 255 // Alpha 值固定为255
    }
  })
  return createImgData(newData)
}
```
mageData.data 是一个 Uint8ClampedArray 数组，可以理解为“RGBA数组”，数组中的每个数字取值为0~255，每4个数字为一组，表示一个像素的 RGBA 值。由于ImageData 为只读对象，所以要另外写一个 creaetImageData() 方法，利用 context.createImageData() 来创建新的 ImageData 对象。  
拿到灰度图像以后，就可以进行指纹提取的操作了。  
**指纹提取**  
在“平均哈希算法”中，若灰度图的某个像素的灰度值大于平均值，则视为1，否则为0。把这部分信息组合起来就是图片的指纹。由于我们已经拿到了灰度图的 ImageData 对象，要提取指纹也就变得很容易了：  
``` 
export function getHashFingerprint (imgData: ImageData) {
const grayList = imgData.data.reduce((pre: number[], cur, index) => {
    if ((index + 1) % 4 === 0) {
        pre.push(imgData.data[index - 1])
    }
    return pre
}, [])
const length = grayList.length
const grayAverage = grayList.reduce((pre, next) => (pre + next), 0) / length
return grayList.map(gray => (gray >= grayAverage ? 1 : 0)).join('')
```
**感知哈希算法**  
均值哈希虽然简单，但受均值的影响非常大。例如对图像进行伽马校正或直方图均衡就会影响均值，从而影响最终的hash值。存在一个更健壮的算法叫pHash。它将均值的方法发挥到极致。使用离散余弦变换(DCT)来获取图片的低频成分。  
离散余弦变换（DCT）是种图像压缩算法，它将图像从像素域变换到频率域。然后一般图像都存在很多冗余和相关性的，所以转换到频率域之后，只有很少的一部分频率分量的系数才不为0，大部分系数都为0（或者说接近于0）。下图的右图是对lena图进行离散余弦变换（DCT）得到的系数矩阵图。从左上角依次到右下角，频率越来越高，由图可以看到，左上角的值比较大，到右下角的值就很小很小了。换句话说，图像的能量几乎都集中在左上角这个地方的低频系数上面了。  
简单来说，该算法经过离散余弦变换以后，把图像从像素域转化到了频率域，而携带了有效信息的低频成分会集中在 DCT 矩阵的左上角，因此我们可以利用这个特性提取图片的特征。  
该算法的步骤如下：  
- 缩小尺寸：pHash以小图片开始，但图片大于88，3232是最好的。这样做的目的是简化了DCT的计算，而不是减小频率。
- 简化色彩：将图片转化成灰度图像，进一步简化计算量。
- 计算DCT：计算图片的DCT变换，得到32*32的DCT系数矩阵。
- 缩小DCT：虽然DCT的结果是3232大小的矩阵，但我们只要保留左上角的88的矩阵，这部分呈现了图片中的最低频率。
- 计算平均值：如同均值哈希一样，计算DCT的均值。
- 计算hash值：这是最主要的一步，根据8*8的DCT矩阵，设置0或1的64位的hash值，大于等于DCT均值的设为”1”，小于DCT均值的设为“0”。组合在一起，就构成了一个64位的整数，这就是这张图片的指纹。

回到代码中，首先添加一个 DCT 方法：  
``` 
function memoizeCosines (N: number, cosMap: any) {
    cosMap = cosMap || {}
    cosMap[N] = new Array(N * N)
    let PI_N = Math.PI / N
    for (let k = 0; k < N; k++) {
        for (let n = 0; n < N; n++) {
            cosMap[N][n + (k * N)] = Math.cos(PI_N * (n + 0.5) * k)
        }
    }
    return cosMap
}
function dct (signal: number[], scale: number = 2) {
    let L = signal.length
    let cosMap: any = null
    if (!cosMap || !cosMap[L]) {
        cosMap = memoizeCosines(L, cosMap)
    }
    let coefficients = signal.map(function () { return 0 })
    return coefficients.map(function (_, ix) {
        return scale * signal.reduce(function (prev, cur, index) {
            return prev + (cur * cosMap[L][index + (ix * L)])
        }, 0)
    })
}
```
然后添加两个矩阵处理方法，分别是把经过 DCT 方法生成的一维数组升维成二维数组（矩阵），以及从矩阵中获取其“左上角”内容。   
``` 

// 一维数组升维
function createMatrix (arr: number[]) {
    const length = arr.length
    const matrixWidth = Math.sqrt(length)
    const matrix = []
    for (let i = 0; i < matrixWidth; i++) {
        const _temp = arr.slice(i * matrixWidth, i * matrixWidth + matrixWidth)
        matrix.push(_temp)
    }
    return matrix
}
// 从矩阵中获取其“左上角”大小为 range × range 的内容
function getMatrixRange (matrix: number[][], range: number = 1) {
    const rangeMatrix = []
    for (let i = 0; i < range; i++) {
        for (let j = 0; j < range; j++) {
            rangeMatrix.push(matrix[i][j])
        }
    }
    return rangeMatrix
}
```
复用之前在“平均哈希算法”中所写的灰度图转化函数createGrayscale()，可以获取“感知哈希算法”的特征值：  
``` 
.
export function getPHashFingerprint (imgData: ImageData) {
    const dctData = dct(imgData.data as any)
    const dctMatrix = createMatrix(dctData)
    const rangeMatrix = getMatrixRange(dctMatrix, dctMatrix.length / 8)
    const rangeAve = rangeMatrix.reduce((pre, cur) => pre + cur, 0) / rangeMatrix.length
    return rangeMatrix.map(val => (val >= rangeAve ? 1 : 0)).join('')
} 
```
颜色分布法:  
每张图片都可以生成颜色分布的直方图（color histogram）。如果两张图片的直方图很接近，就可以认为它们很相似。
任何一种颜色都是由红绿蓝三原色（RGB）构成的，所以上图共有4张直方图（三原色直方图 + 最后合成的直方图）。  
如果每种原色都可以取256个值，那么整个颜色空间共有1600万种颜色（256的三次方）。针对这1600万种颜色比较直方图，计算量实在太大了，因此需要采用简化方法。可以将0～255分成四个区：0～63为第0区，64～127为第1区，128～191为第2区，192～255为第3区。这意味着红绿蓝分别有4个区，总共可以构成64种组合（4的3次方）。  
任何一种颜色必然属于这64种组合中的一种，这样就可以统计每一种组合包含的像素数量。  
基于这个原理，在进行颜色分布法的算法设计时，可以把这个区间的划分设置为可修改的，唯一的要求就是区间的数量必须能够被256整除。算法如下：  
``` 
// 划分颜色区间，默认区间数目为4个
// 把256种颜色取值简化为4种
export function simplifyColorData (imgData: ImageData, zoneAmount: number = 4) {
    const colorZoneDataList: number[] = []
    const zoneStep = 256 / zoneAmount
    const zoneBorder = [0] // 区间边界
    for (let i = 1; i <= zoneAmount; i++) {
        zoneBorder.push(zoneStep * i - 1)
    }
    imgData.data.forEach((data, index) => {
        if ((index + 1) % 4 !== 0) {
            for (let i = 0; i < zoneBorder.length; i++) {
                if (data > zoneBorder[i] && data <= zoneBorder[i + 1]) {
                    data = i
                }
            }
        }
        colorZoneDataList.push(data)
    })
    return colorZoneDataList
}
```
把颜色取值进行简化以后，就可以把它们归类到不同的分组里面去：  
``` 
export function seperateListToColorZone (simplifiedDataList: number[]) { 
    const zonedList: string[] = [] 
    let tempZone: number[] = [] 
    simplifiedDataList.forEach((data, index) => { 
        if ((index + 1) % 4 !== 0) { 
            tempZone.push(data) 
        } else { 
            zonedList.push(JSON.stringify(tempZone)) 
            tempZone = [] 
        } 
    }) 
    return zonedList 
} 
```
最后只需要统计每个相同的分组的总数即可：  
``` 
export function getFingerprint (zonedList: string[], zoneAmount: number = 16) { 
    const colorSeperateMap: { 
        [key: string]: number 
    } = {} 
    for (let i = 0; i < zoneAmount; i++) { 
        for (let j = 0; j < zoneAmount; j++) { 
            for (let k = 0; k < zoneAmount; k++) { 
                colorSeperateMap[JSON.stringify([i, j, k])] = 0 
            } 
        } 
    } 
    zonedList.forEach(zone => { 
        colorSeperateMap[zone]++ 
    }) 
    return Object.values(colorSeperateMap) 
} 
```
**内容特征法**  
”内容特征法“是指把图片转化为灰度图后再转化为”二值图“，然后根据像素的取值（黑或白）形成指纹后进行比对的方法。这种算法的核心是找到一个“阈值”去生成二值图。  
对于生成灰度图，有别于在“平均哈希算法”中提到的取 RGB 均值的办法，在这里我们使用加权的方式去实现。为什么要这么做呢？这里涉及到颜色学的一些概念  
采用 RGB 均值的灰度图是最简单的一种办法，但是它忽略了红、绿、蓝三种颜色的波长以及对整体图像的影响。如果直接取得 RGB 的均值作为灰度，那么处理后的灰度图整体来说会偏暗，对后续生成二值图会产生较大的干扰。  
那么怎么改善这种情况呢？答案就是为 RGB 三种颜色添加不同的权重。鉴于红光有着更长的波长，而绿光波长更短且对视觉的刺激相对更小，所以我们要有意地减小红光的权重而提升绿光的权重。经过统计，比较好的权重配比是 R:G:B = 0.299:0.587:0.114。  
可以得到灰度处理函数：  
``` 
enum GrayscaleWeight { 
    R = .299, 
    G = .587, 
    B = .114 
}
function toGray (imgData: ImageData) { 
    const grayData = [] 
    const data = imgData.data 
    for (let i = 0; i < data.length; i += 4) { 
        const gray = ~~(data[i] * GrayscaleWeight.R + data[i + 1] * GrayscaleWeight.G + data[i + 2] * GrayscaleWeight.B) 
        data[i] = data[i + 1] = data[i + 2] = gray 
        grayData.push(gray) 
    } 
    return grayData 
} 
```
上述函数返回一个 grayData 数组，里面每个元素代表一个像素的灰度值（因为 RBG 取值相同，所以只需要一个值即可）。接下来则使用“大津法”（Otsu's method）去计算二值图的阈值。  
``` 
/ OTSU algorithm 
// rewrite from http://www.labbookpages.co.uk/software/imgProc/otsuThreshold.html 
export function OTSUAlgorithm (imgData: ImageData) { 
    const grayData = toGray(imgData) 
    let ptr = 0 
    let histData = Array(256).fill(0) 
    let total = grayData.length 
    while (ptr < total) { 
        let h = 0xFF & grayData[ptr++] 
        histData[h]++ 
    } 
    let sum = 0 
    for (let i = 0; i < 256; i++) { 
        sum += i * histData[i] 
    } 
    let wB = 0 
    let wF = 0 
    let sumB = 0 
    let varMax = 0 
    let threshold = 0 
    for (let t = 0; t < 256; t++) { 
        wB += histData[t] 
        if (wB === 0) continue 
        wF = total - wB 
        if (wF === 0) break 
        sumB += t * histData[t] 
        let mB = sumB / wB 
        let mF = (sum - sumB) / wF 
        let varBetween = wB * wF * (mB - mF) ** 2 
        if (varBetween > varMax) { 
            varMax = varBetween 
            threshold = t 
        } 
    } 
    return threshold 
}
```
OTSUAlgorithm() 函数接收一个 ImageData 对象，经过上一步的 toGray() 方法获取到灰度值列表以后，根据“大津法”算出最佳阈值然后返回。接下来使用这个阈值对原图进行处理，即可获取二值图。  
``` 
export function binaryzation (imgData: ImageData, threshold: number) { 
    const canvas = document.createElement('canvas') 
    const ctx = canvas.getContext('2d') 
    const imgWidth = Math.sqrt(imgData.data.length / 4) 
    const newImageData = ctx?.createImageData(imgWidth, imgWidth) as ImageData 
    for (let i = 0; i < imgData.data.length; i += 4) { 
        let R = imgData.data[i] 
        let G = imgData.data[i + 1] 
        let B = imgData.data[i + 2] 
        let Alpha = imgData.data[i + 3] 
        let sum = (R + G + B) / 3 
        newImageData.data[i] = sum > threshold ? 255 : 0 
        newImageData.data[i + 1] = sum > threshold ? 255 : 0 
        newImageData.data[i + 2] = sum > threshold ? 255 : 0 
        newImageData.data[i + 3] = Alpha 
    } 
    return newImageData 
} 
```
若图片大小为 N×N，根据二值图“非黑即白”的特性，便可以得到一个 N×N 的 0-1 矩阵，也就是指纹   
## 特征比对算法 
**汉明距离**  
在信息论中，两个等长字符串之间的汉明距离（英语：Hamming distance）是两个字符串对应位置的不同字符的个数。换句话说，它就是将一个字符串变换成另外一个字符串所需要替换的字符个数。  
例如：  
- 1011101与1001001之间的汉明距离是2。
- 2143896与2233796之间的汉明距离是3。
- "toned"与"roses"之间的汉明距离是3。

可以写出计算汉明距离的方法：  
``` 
export function hammingDistance (str1: string, str2: string) {
    let distance = 0
    const str1Arr = str1.split('')
    const str2Arr = str2.split('')
    str1Arr.forEach((letter, index) => {
        if (letter !== str2Arr[index]) {
            distance++
        }
    })
    return distance
}
```
知道了汉明距离，也就可以知道两个等长字符串之间的相似度了（汉明距离越小，相似度越大）：  
似度 = (字符串长度 - 汉明距离) / 字符串长度
**余弦相似度**  
余弦相似度的定义：  
余弦相似性通过测量两个向量的夹角的余弦值来度量它们之间的相似性。0度角的余弦值是1，而其他任何角度的余弦值都不大于1；并且其最小值是-1。从而两个向量之间的角度的余弦值确定两个向量是否大致指向相同的方向。两个向量有相同的指向时，余弦相似度的值为1；两个向量夹角为90°时，余弦相似度的值为0；两个向量指向完全相反的方向时，余弦相似度的值为-1。这结果是与向量的长度无关的，仅仅与向量的指向方向相关。余弦相似度通常用于正空间，因此给出的值为0到1之间。  
注意这上下界对任何维度的向量空间中都适用，而且余弦相似性最常用于高维正空间。  
余弦相似度可以计算出两个向量之间的夹角，从而很直观地表示两个向量在方向上是否相似，这对于计算两个 N×N 的 0-1 矩阵的相似度来说非常有用。根据余弦相似度的公式，可以把它的 js 实现写出来：  
``` 
export function cosineSimilarity (sampleFingerprint: number[], targetFingerprint: number[]) { 
    // cosθ = ∑n, i=1(Ai × Bi) / (√∑n, i=1(Ai)^2) × (√∑n, i=1(Bi)^2) = A · B / |A| × |B| 
    const length = sampleFingerprint.length 
    let innerProduct = 0 
    for (let i = 0; i < length; i++) { 
        innerProduct += sampleFingerprint[i] * targetFingerprint[i] 
    } 
    let vecA = 0 
    let vecB = 0 
    for (let i = 0; i < length; i++) { 
        vecA += sampleFingerprint[i] ** 2 
        vecB += targetFingerprint[i] ** 2 
    } 
    const outerProduct = Math.sqrt(vecA) * Math.sqrt(vecB) 
    return innerProduct / outerProduct 
} 
```
**两种比对算法的适用场景**  
明白了“汉明距离”和“余弦相似度”这两种特征比对算法以后，就要去看看它们分别适用于哪些特征提取算法的场景  
首先来看“颜色分布法”。在“颜色分布法”里面，把一张图的颜色进行区间划分，通过统计不同颜色区间的数量来获取特征，那么这里的特征值就和“数量”有关，也就是非 0-1 矩阵  
要比较两个“颜色分布法”特征的相似度，“汉明距离”是不适用的，只能通过“余弦相似度”来进行计算  
**计算精度**  
经过对不同素材的多方比对，得出了下列结论:  
对于两张颜色较为丰富，细节较多的图片来说，“颜色分布法”的计算结果是最符合直觉的。  
对于两张内容相近但颜色差异较大的图片来说，“内容特征法”和“平均/感知哈希算法”都能得到符合直觉的结果  
针对“颜色分布法“，区间的划分数量对计算结果影响较大，选择合适的区间很重要

参考:
[利用 JS 实现多种图片相似度算法](https://mp.weixin.qq.com/s/4hu9NOx7yw7Kwzubbsv3lQ)
