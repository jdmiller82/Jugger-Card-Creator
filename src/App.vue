/*
Version 1.3.2
Main features included:
- Vue 3 app with Tailwind CSS
- Fabric.js canvas rendering with structured layout
- Inputs for name, description, rarity, weapon, main image, team logo, card color
- Preview includes rarity gem, NJA logo, and weapon icon placeholders
- PNG export functionality
*/
<script setup>
import { ref, onMounted, watch } from 'vue'
import { fabric } from 'fabric'

const cardName = ref('')

watch(cardName, (val) => {
  if (val.length > 26) {
    cardName.value = val.slice(0, 26)
  }
})

const description = ref('')

watch(description, (val) => {
  if (val.length > 355) {
    description.value = val.slice(0, 355)
  }
})
const rarity = ref('Common')
const weapon = ref('Q-Tip')
const cardColor = ref('Blue')
const imageFile = ref(null)
const logoFile = ref(null)
const canvasRef = ref(null)
let canvas
let uploadedImage = null
let uploadedLogo = null
let backgroundImage = null

const rarities = ['Common', 'Rare', 'Epic', 'Legendary']
const weapons = ['Q-Tip', 'Staff', 'Longsword', 'Dual Short Swords', 'Sword and Shield', 'Chain']
const cardColors = ['Blue', 'Green', 'Yellow', 'Red']

onMounted(async () => {
  await document.fonts.load('1em Sriracha');
  await new Promise(resolve => setTimeout(resolve, 50));
  canvas = new fabric.Canvas(canvasRef.value, {
    width: 500,
    height: 750,
    backgroundColor: '',
    selection: false
  })
  canvas.on('object:moving', (e) => e.preventDefault())
  updateCard()
})

function updateCard() {
  if (!canvas) return
  const backgroundPath = `/assets/backgrounds/${cardColor.value.toLowerCase()}.png`
  fabric.Image.fromURL(backgroundPath, (bgImg) => {
    backgroundImage = bgImg
    drawCard()
  }, { crossOrigin: 'anonymous' })
}

function drawCard() {
  canvas.clear()

  const padding = 12
  const cardWidth = 500
  const cardHeight = 750
  const innerWidth = cardWidth - padding * 2

  const makeUnselectable = (obj) => {
    obj.selectable = false
    obj.evented = false
    return obj
  }

  // Card background
  if (backgroundImage) {
      backgroundImage.scaleToWidth(500);
      backgroundImage.scaleToHeight(750);
      backgroundImage.set({
      left: 0,
      top: 0,
      selectable: false,
      evented: false
    })
    canvas.add(backgroundImage)
    canvas.sendToBack(backgroundImage)
  }

  // User uploaded image
  if (uploadedImage) {
      uploadedImage.set({
      left: padding,
      top: padding,
      selectable: false,
      evented: false,
    })
    uploadedImage.clipPath = new fabric.Rect({
      width: cardWidth,
      height: cardWidth - 12,
      absolutePositioned: true
    })
    canvas.add(uploadedImage)
    canvas.sendToBack(uploadedImage)
  } else {
    const imageBox = makeUnselectable(new fabric.Rect({
      left: padding,
      top: padding - 1,
      width: innerWidth,
      height: cardWidth,
      fill: '#EEF2FF',
      absolutePositioned: true
    }))
    canvas.add(imageBox)
    canvas.sendToBack(imageBox)
  }

  // Calculate dynamic title width
  const textPadding = 32
  const ellipsizedTitle = cardName.value.length > 26 ? cardName.value.slice(0, 25) + '…' : cardName.value
  const textWidth = new fabric.Text(ellipsizedTitle, {
    fontSize: 24,
    fontFamily: 'Sriracha',
    fontWeight: 'bold'
  }).width || 100

  const barWidth = Math.min(textWidth + textPadding * 2, 476)

  const titleBarPath = new fabric.Path(`M0,12
    Q0,0 12,0
    H${barWidth}
    V26
    Q${barWidth},50 ${barWidth - 24},50
    H0
    V24
    Q0,12 0,12
    Z`, {
    left: padding - 1,
    top: padding -1,
    fill: getCardColor()
  })
  const titleBar = makeUnselectable(titleBarPath)
  const titleText = makeUnselectable(new fabric.Text(ellipsizedTitle, {
    left: padding + textPadding,
    top: padding + 10,
    fontSize: 24,
    fontFamily: 'Sriracha',
    fontWeight: 'bold',
    fill: 'white',
    textAlign: 'left'
  }))
  canvas.add(titleText, titleBar)
  canvas.bringToFront(titleBar)
  canvas.bringToFront(titleText)

  const descText = makeUnselectable(new fabric.Textbox(description.value, {
    left: padding + 6,
    top: 532,
    width: innerWidth - 12,
    fontSize: 16,
    fontFamily: 'Sriracha',
    fontStyle: 'italic',
    fill: '#000'
  }))
  canvas.add(descText)


  // RARITY
  fabric.Image.fromURL(`/assets/rarity/${rarity.value.toLowerCase()}.png`, (rarityImg) => {
    rarityImg.set({
      left: -4,
      top: 462,
      scaleX: 0.5,
      scaleY: 0.5,
      selectable: false,
      evented: false
    })
    canvas.add(rarityImg)
  })

  // NJA logo
  fabric.Image.fromURL('/assets/logo/nja.png', (njaImg) => {
    njaImg.set({
      left: 12,
      top: 674,
      scaleX: 0.5,
      scaleY: 0.5,
      selectable: false,
      evented: false
    })
    canvas.add(njaImg)
  })

  // WEAPONS
  // fabric.Image.fromURL(`/assets/weapons/${weapon.value.toLowerCase().replaceAll(' ', '-')}.png`, (weaponImg) => {
  //   weaponImg.set({
  //     left: 62,
  //     top: 674,
  //     scaleX: 0.5,
  //     scaleY: 0.5,
  //     selectable: false,
  //     evented: false
  //   })
  //   canvas.add(weaponImg)
  // })

  fabric.Image.fromURL(`/assets/weapons/${weapon.value.toLowerCase().replaceAll(' ', '-')}-${cardColor.value.toLowerCase()}.png`, (weaponImg) => {
    weaponImg.set({
      left: 62,
      top: 674,
      scaleX: 0.5,
      scaleY: 0.5,
      selectable: false,
      evented: false
    })
    canvas.add(weaponImg)
  })

  // TEAM LOGO
  if (uploadedLogo) {
    uploadedLogo.set({
      left: 366,
      top: 396,
      scaleX: 128 / uploadedLogo.width,
      scaleY: 128 / uploadedLogo.height,
      selectable: false,
      evented: false
    })

    uploadedLogo.clipPath = new fabric.Circle({
      radius: 500,
      originX: 'center',
      originY: 'center'
    })

    canvas.add(uploadedLogo)
    canvas.sendToBack(uploadedLogo)
    canvas.bringForward(uploadedLogo)
  } else {
    const teamLogo = makeUnselectable(new fabric.Circle({
      left: 366,
      top: 396,
      radius: 64,
      fill: '#EEF2FF',
    }))
    canvas.add(teamLogo)
    canvas.sendToBack(teamLogo)
    canvas.bringForward(teamLogo)
  }
  canvas.renderAll()
}

function handleImageUpload(e) {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    fabric.Image.fromURL(reader.result, (img) => {
      const boxSize = 476
      const imgAspect = img.width / img.height

      if (imgAspect > 1) {
        img.scaleToHeight(boxSize)
      } else {
        img.scaleToWidth(boxSize)
      }

      img.set({
        left: 0,
        top: 0,
        originX: 'left',
        originY: 'top'
      })

      uploadedImage = img
      updateCard()
    })
  }
  reader.readAsDataURL(file)
}

function handleLogoUpload(e) {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    fabric.Image.fromURL(reader.result, (img) => {
      uploadedLogo = img
      updateCard()
    })
  }
  reader.readAsDataURL(file)
}

function getCardColor() {
  const colorMap = {
    Blue: '#2563EB',
    Green: '#16A34A',
    Yellow: '#EAB308',
    Red: '#DC2626'
  }
  return colorMap[cardColor.value] || '#2563EB'
}

function exportCard() {
  const dataURL = canvas.toDataURL({
    format: 'png',
    multiplier: 2 // or 3 for 1500x2250
  })
  const link = document.createElement('a')
  link.href = dataURL
  link.download = `${cardName.value || 'jugger-card'}.png`
  link.click()
}

</script>

<!-- TEMPLATE -->
<template>
  <div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6">
    <div>
      <h2 class="text-xl font-bold mb-4">Card Editor</h2>
      <div class="mb-4">
        <label>Name:</label>
        <input v-model="cardName" @input="updateCard" placeholder="Person or card name" class="border w-full p-2" />
        <div class="text-sm text-gray-500 text-right mt-1">{{ cardName.length }}/26</div>
      </div>
      <div class="mb-4">
        <label>Description:</label>
        <textarea v-model="description" @input="updateCard" placeholder="Add your description here" class="border w-full p-2"></textarea>
        <div class="text-sm text-gray-500 text-right mt-1">{{ description.length }}/355</div>
      </div>
      <div class="mb-4">
        <label>Rarity:</label>
        <select v-model="rarity" @change="updateCard" class="border w-full p-2">
          <option v-for="r in rarities" :key="r">{{ r }}</option>
        </select>
      </div>
      <div class="mb-4">
        <label>Weapon:</label>
        <select v-model="weapon" @change="updateCard" class="border w-full p-2">
          <option v-for="w in weapons" :key="w">{{ w }}</option>
        </select>
      </div>
      <div class="mb-4">
        <label>Card Color:</label>
        <select v-model="cardColor" @change="updateCard" class="border w-full p-2">
          <option v-for="color in cardColors" :key="color">{{ color }}</option>
        </select>
      </div>
      <div class="mb-4">
        <label>Main Image:</label>
        <input type="file" @change="handleImageUpload" />
      </div>
      <div class="mb-4">
        <label>Team Logo:</label>
        <input type="file" @change="handleLogoUpload" />
      </div>
      <button @click="exportCard" class="bg-blue-500 text-white px-4 py-2 rounded">Export as PNG</button>
    </div>
    <div>
      <h2 class="text-xl font-bold mb-4">Live Preview</h2>
      <canvas ref="canvasRef"></canvas>
    </div>
  </div>
  <div><p>Version 1.3.2</p></div>
  <br>
  <hr>
  <br>
  <div>
    <p>💖 Like this? Support me!</p>
    <a href="https://www.paypal.com/donate/?hosted_button_id=YFFSNNTG9259J" target="_blank">Donate here</a>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Sriracha&display=swap');

canvas {
  /* border: 1px solid #ccc; */
}

</style>
