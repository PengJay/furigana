<template>
  <div class="furigana-converter">
    <div v-if="result" class="result" v-html="result"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { bitable } from '@lark-base-open/js-sdk'

const result = ref('')
const kuroshiro = ref(null)

onMounted(async () => {
  try {

    console.log('Kuroshiro initializing...')

    kuroshiro.value = new window.Kuroshiro.default()

    console.log('window.Kuroshiro', window.Kuroshiro)


    console.log('Kuroshiro', kuroshiro.value)


    console.log('KuromojiAnalyzer', window.KuromojiAnalyzer)



    // 修改初始化方式
    await kuroshiro.value.init(new window.KuromojiAnalyzer({
      dicPath: '/libs/dict'
    }))

    console.log('Kuroshiro initialized')

    // 添加选择事件监听
    bitable.base.onSelectionChange(async () => {
      await convertSelectedCell()
    })
  } catch (error) {
    console.error('Kuroshiro initialization failed:', error)
  }
})

const convertSelectedCell = async () => {
  if (!kuroshiro.value) {
    console.error('Kuroshiro not initialized')
    return
  }

  try {
    const selection = await bitable.base.getSelection()
    const table = await bitable.base.getTableById(selection.tableId)
    const cellValue = await table.getCellValue(selection.fieldId, selection.recordId)

    console.log('cellValue', cellValue);


    if (cellValue) {
      const converted = await kuroshiro.value.convert(cellValue.toString(), {
        mode: 'furigana',
        to: 'hiragana'
      })

      result.value = converted
      await table.setCellValue(selection.fieldId, selection.recordId, converted)
    }
  } catch (error) {
    console.error('Conversion failed:', error)
  }
}
</script>

<style scoped>
.furigana-converter {
  padding: 16px;
}

.result {
  margin-top: 16px;
  padding: 8px;
  border: 1px solid #eee;
  border-radius: 4px;
}
</style>