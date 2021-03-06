<template>
  <div class="image-params-form">
    <QInput
      type="number"
      stack-label="Width"
      v-model="width"
    />
    <QInput
      type="number"
      stack-label="Height"
      v-model="height"
    />
    <ColorSelector @change="onColorChange" />
    <QInput
      type="text"
      stack-label="Caption"
      v-model="caption"
    />
    <FontFaceSelector v-model="fontFamily" />
    <QInput
      type="number"
      stack-label="Font size"
      v-model="fontSize"
    />
    <QSelect
      radio
      stack-label="Font weight"
      v-model="fontWeight"
      :options="[
        {label: '100', value: '100'},
        {label: '200', value: '200'},
        {label: '300', value: '300'},
        {label: '400', value: '400'},
        {label: '500', value: '500'},
        {label: '600', value: '600'},
        {label: '700', value: '700'},
        {label: '800', value: '800'},
        {label: '900', value: '900'}
      ]"
    />
    <QSelect
      radio
      stack-label="Filetype"
      v-model="filetype"
      :options="filetypeOptions"
    />
    <SliderField
      v-if="filetype === 'image/jpeg' || filetype === 'image/webp'"
      label="Image quality"
    >
      <QSlider
        v-model="imageQuality"
        :min="0"
        :max="1"
        :step="0.01"
        :decimals="2"
        label
        label-always
      />
    </SliderField>
    <div class="image-preview">
      <img :src="dataURI" alt="preview image">
    </div>
    <QInput
      prefix="&amp;lt;img&nbsp;src=&quot;"
      suffix="&quot;&gt;"
      stack-label="Data URI"
      v-model="dataURI"
      readonly
      @focus="selectDataURI($event)"
    />
    <a
    :href="dataURI"
    class="
      q-btn
      row
      inline
      flex-center
      q-focusable
      q-hoverable
      relative-position
      q-btn-rectangle
      q-btn-big
      bg-primary
      text-white
      full-width
      btn-download
    "
    :download="friendlyFilename"
    v-ripple
    >
      <div class="desktop-only q-focus-helper"></div>
      <span class="q-btn-inner row col flex-center">Download image</span>
    </a>
  </div>
</template>

<script>
import {
  QCard,
  QInput,
  QSelect,
  QSlider,
  debounce,
  Ripple
} from 'quasar'
import BitmapRenderer from '../lib/BitmapRenderer'
import SVGREnderer from '../lib/SVGRenderer'
import SliderField from './SliderField'
import FontFaceSelector from './FontFaceSelector'
import ColorSelector from './ColorSelector'

const extensions = {
  'image/png': 'png',
  'image/gif': 'gif',
  'image/jpeg': 'jpg',
  'image/x-icon': 'ico',
  'image/webp': 'webp',
  'image/svg+xml': 'svg'
}

function filetypeOptions () {
  const formats = [
    {label: 'png', value: 'image/png'},
    {label: 'gif', value: 'image/gif'},
    {label: 'jpg', value: 'image/jpeg'},
    {label: 'ico', value: 'image/x-icon'},
    {label: 'webp', value: 'image/webp'}
  ]

  const retval = []

  const canvas = document.createElement('canvas')

  formats.forEach((e) => {
    if (canvas.toDataURL(e.value).indexOf(e.value) !== -1) {
      retval.push(e)
    }
  })

  return retval
};

const renderers = {
  svg: new SVGREnderer(),
  bitmap: new BitmapRenderer()
}

export default {
  components: {
    ColorSelector,
    FontFaceSelector,
    SliderField,
    QCard,
    QInput,
    QSelect,
    QSlider
  },
  directives: {
    Ripple
  },
  data () {
    return {
      width: 600,
      height: 400,
      textColor: '#ffffff',
      backgroundColor: '#cccccc',
      caption: '',
      fontFamily: 'sans-serif',
      fontSize: 40,
      fontWeight: '400',
      fontAlign: 'center',
      filetype: 'image/png',
      dataURI: '',
      imageQuality: 0.92,
      filetypeOptions: [{label: 'svg', value: 'image/svg+xml'}]
    }
  },
  computed: {
    friendlyFilename () {
      const fg = this.textColor.replace(/#/, '')
      const bg = this.backgroundColor.replace(/#/, '')

      let retval = `dummy_${this.width}x${this.height}_${fg}_${bg}`

      if (this.caption !== '') {
        let sanitizedCaption = this.caption.replace(/\s/g, '-')
        sanitizedCaption = sanitizedCaption.replace(/[^A-Za-z0-9-]/g, '')
        sanitizedCaption = sanitizedCaption.toLowerCase()

        retval = `${retval}_${sanitizedCaption}`
      }

      retval = `${retval}.${extensions[this.filetype]}`

      return retval
    },
    rendererParams () {
      return {
        width: this.width,
        height: this.height,
        textColor: this.textColor,
        backgroundColor: this.backgroundColor,
        caption: this.caption !== '' ? this.caption : `${this.width} x ${this.height}`,
        fontFamily: this.fontFamily,
        fontSize: this.fontSize,
        fontWeight: this.fontWeight,
        filetype: this.filetype
      }
    }
  },
  watch: {
    rendererParams () {
      this.updateImage()
    },
    imageQuality () {
      this.updateImage()
    }
  },
  methods: {
    updateImage: debounce(function updateImageDebounced () {
      this.updateDataURI()
    }, 400),
    updateDataURI () {
      if (this.filetype === 'image/svg+xml') {
        this.dataURI = renderers.svg.render(this.rendererParams)
      }
      else if (this.filetype === 'image/jpeg' || this.filetype === 'image/webp') {
        this.dataURI = renderers.bitmap.render(this.rendererParams, this.filetype, this.imageQuality)
      }
      else {
        this.dataURI = renderers.bitmap.render(this.rendererParams, this.filetype)
      }
    },
    selectDataURI (event) {
      event.target.setSelectionRange(0, event.target.value.length)
    },
    onColorChange (event) {
      this.backgroundColor = event.backgroundColor
      this.textColor = event.textColor
    }
  },
  mounted () {
    this.filetypeOptions = this.filetypeOptions.concat(filetypeOptions())

    this.updateImage()
  }
}
</script>

<style scoped>
.image-params-form {
  margin: 0 auto;
  max-width: 90vw;
  padding-bottom: 50px;
  width: 500px;
}

.button-wrapper{
  display: flex;
  justify-content: center;
  padding: 20px 0;
}

.image-preview {
  align-items: center;
  display: flex;
  justify-content: center;
  height: 200px;
  margin-top: 50px;
}

.image-preview img {
  display: block;
  max-height: 100%;
  max-width: 100%;
}

.btn-download {
  appearance: none;
}
</style>
