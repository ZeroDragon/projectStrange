<template lang="pug">
  .home
    .left-column
      h1 Proyecto Dr. Strange
      .roms-container(v-show="!hideUploadSection")
        .rom-section
          h2 BIOS
          input(type="file" ref="biosInput" @change="handleBiosUpload" accept=".csv" :disabled="roms.bios" class="hidden-input")
          button.upload-btn(@click="biosInput.click()" :disabled="roms.bios") {{ roms.bios ? 'BIOS cargada' : 'Cargar BIOS' }}
          .rom-info(v-if="roms.bios")
            p Cargado: {{ roms.bios.length }} registros
            p Columnas: 6
        .rom-section
          h2 ROM
          input(type="file" ref="romInput" @change="handleRomUpload" accept=".csv" :disabled="roms.rom" class="hidden-input")
          button.upload-btn(@click="romInput.click()" :disabled="roms.rom") {{ roms.rom ? 'ROM cargada' : 'Cargar ROM' }}
          .rom-info(v-if="roms.rom")
            p Cargado: {{ roms.rom.length }} registros
            p Columnas: 6
      .status
        .status-content(v-if="hasRoms")
          p ✅ BIOS y ROM cargadas en memoria
          .status-actions
            button.download-btn(@click="downloadRom" aria-label="Descargar ROM")
              svg(width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg")
                path(d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
                path(d="M7 10l5 5 5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
                path(d="M12 15V3" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
            button.clear-btn(@click="clearMemory" aria-label="Limpiar memoria")
              svg(width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg")
                path(d="M3 6h18" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
                path(d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
                path(d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round")
        p(v-else) ⚠️ Carga BIOS y ROM para comenzar
      .ram-section(v-show="hasRoms && !hasCalculated")
        h2 RAM
        .ram-content
          .ram-column(v-for="(value, index) in ram" :key="index")
            .ram-value {{ value }}
            input.ram-input(type="number" v-model="newRamValues[index]")
        .ram-buttons
          button.load-ram-btn(@click="loadRam") Cargar
          button.calculate-btn(@click="calculate") Calcular
      .ram-section(v-show="hasRoms")
        .universes-section(v-if="universes.length > 0")
          .universes-list
            .universe-item(
              v-for="(universe, universeIndex) in universes"
              :key="universeIndex"
              :class="{ 'consensus': universe.name.includes('Consensus') }"
              @mouseenter="hoveredUniverse = universeIndex"
              @mouseleave="hoveredUniverse = null"
            )
              .universe-content
                .universe-column(v-for="(item, colIndex) in universe.items" :key="colIndex")
                  .universe-number {{ item.number }}
                  .universe-count {{ item.count }}
          .manual-universe-section
            .manual-universe-inputs
              .manual-input(v-for="(value, index) in manualUniverse" :key="index")
                input(type="number" v-model="manualUniverse[index]" min="1" max="56" placeholder="0")
            button.add-manual-btn(@click="addManualUniverse") Insertar Universo
    .right-column
      .results-section(v-if="calculatedResults.length > 0")
        .results-grid
          .result-column(v-for="(result, index) in calculatedResults" :key="index")
            .result-item(v-for="(item, itemIndex) in result" :key="itemIndex" :class="{ 'bios-item': item.isBios, 'zero-count': item.count === 0, 'highlighted': isHighlighted(index, item.number) }")
              .result-number {{ item.number }}
              .result-arrow →
              .result-count {{ item.count }}
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const roms = ref({
  bios: null,
  rom: null
})

const biosInput = ref(null)
const romInput = ref(null)

const newRamValues = ref([0, 0, 0, 0, 0, 0])

const calculatedResults = ref([])
const biosResults = ref([])
const universes = ref([])
const hoveredUniverse = ref(null)
const hasCalculated = ref(false)
const manualUniverse = ref([0, 0, 0, 0, 0, 0])

const hasRoms = computed(() => roms.value.bios && roms.value.rom)

const hideUploadSection = computed(() => hasRoms.value)

const ram = computed(() => {
  if (!roms.value.rom || roms.value.rom.length === 0) return []
  return roms.value.rom[roms.value.rom.length - 1]
})

const STORAGE_KEY = 'doctorstrange_roms'

const parseCSV = (content) => {
  const lines = content.trim().split('\n')
  const data = []

  for (const line of lines) {
    const cells = line.trim().split(',')
    if (cells.length !== 6) continue

    const row = cells.map(cell => {
      const num = parseInt(cell.trim())
      return isNaN(num) ? 0 : num
    })

    data.push(row)
  }

  return data
}

const handleBiosUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const content = e.target.result
    roms.value.bios = parseCSV(content)
    saveToStorage()
  }
  reader.readAsText(file)
}

const handleRomUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const content = e.target.result
    roms.value.rom = parseCSV(content)
    saveToStorage()
  }
  reader.readAsText(file)
}

const saveToStorage = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(roms.value))
}

const loadFromStorage = () => {
  const stored = localStorage.getItem(STORAGE_KEY)
  if (stored) {
    roms.value = JSON.parse(stored)
  }
}

const clearMemory = () => {
  roms.value = {
    bios: null,
    rom: null
  }
  if (biosInput.value) {
    biosInput.value.value = ''
  }
  if (romInput.value) {
    romInput.value.value = ''
  }
  calculatedResults.value = []
  biosResults.value = []
  universes.value = []
  hoveredUniverse.value = null
  hasCalculated.value = false
  localStorage.removeItem(STORAGE_KEY)
}

const downloadRom = () => {
  const csvContent = roms.value.rom.map(row => row.join(',')).join('\n')
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  const url = URL.createObjectURL(blob)

  link.setAttribute('href', url)
  link.setAttribute('download', 'rom.csv')
  link.style.visibility = 'hidden'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  URL.revokeObjectURL(url)
}

const loadRam = () => {
  const newRow = newRamValues.value.map(v => parseInt(v) || 0)
  if (newRow.includes(0)) {
    alert('No se puede cargar RAM con valores en 0. Por favor verifica todos los campos.')
    return
  }
  roms.value.rom.push(newRow)
  newRamValues.value = [0, 0, 0, 0, 0, 0]
  calculatedResults.value = []
  biosResults.value = []
  universes.value = []
  hoveredUniverse.value = null
  hasCalculated.value = false
  saveToStorage()
}

const calculateOccurrences = (data) => {
  const results = []

  for (let col = 0; col < 6; col++) {
    const ramValue = ram.value[col]
    const occurrences = {}

    const columnData = data.map(row => row[col])

    for (let i = 0; i < columnData.length; i++) {
      if (columnData[i] === ramValue && i + 1 < columnData.length) {
        const nextNumber = columnData[i + 1]
        if (!occurrences[nextNumber]) {
          occurrences[nextNumber] = 0
        }
        occurrences[nextNumber]++
      }
    }

    const sortedResults = Object.entries(occurrences)
      .map(([number, count]) => ({ number: parseInt(number), count }))
      .sort((a, b) => a.number - b.number)

    results.push(sortedResults)
  }

  return results
}

const calculate = () => {
  const romResults = calculateOccurrences(roms.value.rom)
  const biosResultsData = calculateOccurrences(roms.value.bios)

  const combinedResults = []

  for (let col = 0; col < 6; col++) {
    const romNumbers = new Set(romResults[col].map(item => item.number))
    const combined = [...romResults[col]]

    for (const biosItem of biosResultsData[col]) {
      if (!romNumbers.has(biosItem.number)) {
        combined.push({ ...biosItem, isBios: true })
      }
    }

    combined.sort((a, b) => a.number - b.number)

    const existingNumbers = new Set(combined.map(item => item.number))

    const numbersWithCount = combined.filter(item => item.count > 0).map(item => item.number)
    const minNum = numbersWithCount.length > 0 ? Math.min(...numbersWithCount) : 1
    const maxNum = numbersWithCount.length > 0 ? Math.max(...numbersWithCount) : 56

    for (let num = minNum - 1; num <= maxNum + 1; num++) {
      if (num >= 1 && num <= 56 && !existingNumbers.has(num)) {
        combined.push({ number: num, count: 0 })
      }
    }

    combined.sort((a, b) => a.number - b.number)
    combinedResults.push(combined)
  }

  calculatedResults.value = combinedResults
  biosResults.value = biosResultsData

  calculateUniverses(combinedResults)
  hasCalculated.value = true
}

const calculateUniverses = (results) => {
  const generateUniverse = (initialIndex, universeName) => {
    const universe = []
    const chosenIndices = []
    const usedNumbers = new Set()
    const minCounts = []

    const backtrack = (col) => {
      if (col === 6) return true

      const colResults = results[col]
      const previousCount = col > 0 ? universe[col - 1].count : -Infinity

      let candidates = colResults
        .map((item, index) => ({ ...item, index }))
        .map(item => ({
          ...item,
          score: col > 0 ? item.count - Math.abs(item.index - chosenIndices[col - 1]) : item.count
        }))

      candidates.sort((a, b) => b.score - a.score)

      for (const candidate of candidates) {
        if (usedNumbers.has(candidate.number)) continue
        if (candidate.count <= previousCount) continue

        chosenIndices.push(candidate.index)
        usedNumbers.add(candidate.number)
        universe.push({
          column: col,
          index: candidate.index,
          number: candidate.number,
          count: candidate.count
        })

        if (backtrack(col + 1)) return true

        universe.pop()
        chosenIndices.pop()
        usedNumbers.delete(candidate.number)
      }

      return false
    }

    if (initialIndex === null) return null

    const baseItem = results[0][initialIndex]
    usedNumbers.add(baseItem.number)
    universe.push({
      column: 0,
      index: initialIndex,
      number: baseItem.number,
      count: baseItem.count
    })
    chosenIndices.push(initialIndex)

    if (backtrack(1)) {
      return universe
    }

    return null
  }

  const colResults = results[0]
  const zeroCountItems = colResults.filter(item => item.count === 0)
  const firstZeroIndex = zeroCountItems.length > 0 ? colResults.indexOf(zeroCountItems[0]) : null

  const maxCountItem = colResults.reduce((max, item) => {
    return item.count > max.count ? item : max
  }, colResults[0])
  const maxCountIndex = colResults.indexOf(maxCountItem)

  const universesList = []

  const universe1 = generateUniverse(firstZeroIndex, 'Semilla Zero Count')
  if (universe1) universesList.push({ items: universe1, name: 'Semilla Zero Count' })

  const generateUniverse2 = () => {
    const colResults = results[0]

    let sortedCandidates = colResults
      .map((item, index) => ({ ...item, index }))
      .sort((a, b) => b.count - a.count)

    let bestUniverse = null
    let bestStartCount = -1

    for (const startCandidate of sortedCandidates) {
      if (bestUniverse !== null && startCandidate.count < bestStartCount) {
        break
      }

      const universe = []
      const chosenIndices = []
      const usedNumbers = new Set()

      const backtrack = (col) => {
        if (col === 6) return true

        const colResults = results[col]
        const previousCount = col > 0 ? universe[col - 1].count : -Infinity

        let candidates = colResults
          .map((item, index) => ({ ...item, index }))
          .map(item => ({
            ...item,
            score: col > 0 ? item.count - Math.abs(item.index - chosenIndices[col - 1]) : item.count
          }))

        candidates.sort((a, b) => b.score - a.score)

        for (const candidate of candidates) {
          if (usedNumbers.has(candidate.number)) continue
          if (candidate.count <= previousCount) continue

          chosenIndices.push(candidate.index)
          usedNumbers.add(candidate.number)
          universe.push({
            column: col,
            index: candidate.index,
            number: candidate.number,
            count: candidate.count
          })

          if (backtrack(col + 1)) return true

          universe.pop()
          chosenIndices.pop()
          usedNumbers.delete(candidate.number)
        }

        return false
      }

      chosenIndices.push(startCandidate.index)
      usedNumbers.add(startCandidate.number)
      universe.push({
        column: 0,
        index: startCandidate.index,
        number: startCandidate.number,
        count: startCandidate.count
      })

      if (backtrack(1)) {
        return universe
      }
    }

    return null
  }

  const universe2 = generateUniverse2()
  if (universe2) universesList.push({ items: universe2, name: 'Semilla Max Count' })

  const generateUniverse3 = () => {
    const universe = []
    const chosenIndices = []
    const usedNumbers = new Set()

    const backtrack = (col) => {
      if (col === 6) return true

      const colResults = results[col]
      const previousCount = col > 0 ? universe[col - 1].count : -Infinity

      let candidates = colResults
        .map((item, index) => ({ ...item, index }))
        .filter(item => item.count > previousCount && !usedNumbers.has(item.number))

      candidates.sort((a, b) => b.count - a.count)

      for (const candidate of candidates) {
        chosenIndices.push(candidate.index)
        usedNumbers.add(candidate.number)
        universe.push({
          column: col,
          index: candidate.index,
          number: candidate.number,
          count: candidate.count
        })

        if (backtrack(col + 1)) return true

        universe.pop()
        chosenIndices.pop()
        usedNumbers.delete(candidate.number)
      }

      return false
    }

    if (backtrack(0)) {
      return universe
    }

    return null
  }

  const universe3 = generateUniverse3()
  if (universe3) universesList.push({ items: universe3, name: 'Max Count Creciente' })

  const generateTwinUniverse = (results, useUpper) => {
    const universe = []
    const chosenIndices = []
    const usedNumbers = new Set()

    const getCenterIndex = (colResults) => {
      const length = colResults.length
      if (length % 2 === 0) {
        return useUpper ? (length / 2 - 1) : (length / 2)
      } else {
        return Math.floor(length / 2)
      }
    }

    const getStartingIndex = (colResults) => {
      const centerIndex = getCenterIndex(colResults)
      const centerItem = colResults[centerIndex]

      if (useUpper) {
        if (colResults.length % 2 === 0) {
          return centerIndex
        } else {
          for (let i = centerIndex - 1; i >= 0; i--) {
            if (colResults[i].count !== centerItem.count) {
              return i
            }
          }
          return 0
        }
      } else {
        if (colResults.length % 2 === 0) {
          return centerIndex
        } else {
          for (let i = centerIndex + 1; i < colResults.length; i++) {
            if (colResults[i].count !== centerItem.count) {
              return i
            }
          }
          return colResults.length - 1
        }
      }
    }

    const backtrack = (col) => {
      if (col === 6) return true

      const colResults = results[col]
      const previousCount = col > 0 ? universe[col - 1].count : -Infinity
      const centerIndex = getCenterIndex(colResults)
      const startingIndex = getStartingIndex(colResults)

      let candidates = colResults
        .map((item, index) => ({ ...item, index }))
        .filter(item => item.count > previousCount && !usedNumbers.has(item.number))

      candidates.sort((a, b) => {
        const distanceA = Math.abs(a.index - startingIndex)
        const distanceB = Math.abs(b.index - startingIndex)
        return distanceA - distanceB
      })

      for (const candidate of candidates) {
        chosenIndices.push(candidate.index)
        usedNumbers.add(candidate.number)
        universe.push({
          column: col,
          index: candidate.index,
          number: candidate.number,
          count: candidate.count
        })

        if (backtrack(col + 1)) return true

        universe.pop()
        chosenIndices.pop()
        usedNumbers.delete(candidate.number)
      }

      return false
    }

    if (backtrack(0)) {
      return universe
    }

    return null
  }

  const universe4 = generateTwinUniverse(results, true)
  if (universe4) universesList.push({ items: universe4, name: 'Centro Arriba' })

  const universe5 = generateTwinUniverse(results, false)
  if (universe5) universesList.push({ items: universe5, name: 'Centro Abajo' })

  const generateUniverse6 = () => {
    const universe = []
    const chosenIndices = []
    const usedNumbers = new Set()

    const backtrack = (col) => {
      if (col === 6) return true

      const colResults = results[col]
      const previousCount = col > 0 ? universe[col - 1].count : -Infinity

      let candidates = colResults
        .map((item, index) => ({ ...item, index }))
        .filter(item => item.count > previousCount && !usedNumbers.has(item.number))

      candidates.sort((a, b) => a.count - b.count)

      for (const candidate of candidates) {
        chosenIndices.push(candidate.index)
        usedNumbers.add(candidate.number)
        universe.push({
          column: col,
          index: candidate.index,
          number: candidate.number,
          count: candidate.count
        })

        if (backtrack(col + 1)) return true

        universe.pop()
        chosenIndices.pop()
        usedNumbers.delete(candidate.number)
      }

      return false
    }

    if (backtrack(0)) {
      return universe
    }

    return null
  }

  const universe6 = generateUniverse6()
  if (universe6) universesList.push({ items: universe6, name: 'Min Count Creciente' })

  const removeDuplicateUniverses = () => {
    const seen = new Set()
    const uniqueUniverses = []

    for (const universe of universesList) {
      const signature = universe.items.map(item => item.number).join(',')
      if (!seen.has(signature)) {
        seen.add(signature)
        uniqueUniverses.push(universe)
      }
    }

    universesList.length = 0
    universesList.push(...uniqueUniverses)

  }

  removeDuplicateUniverses()

  const generateConsensusUniverse = () => {
    const columnStats = []

    for (let col = 0; col < 6; col++) {
      const numberCounts = new Map()

      for (const universe of universesList) {
        const item = universe.items.find(u => u.column === col)
        if (item) {
          const count = numberCounts.get(item.number) || 0
          numberCounts.set(item.number, count + 1)
        }
      }

      const maxCount = Math.max(...numberCounts.values())
      const winners = Array.from(numberCounts.entries())
        .filter(([_, count]) => count === maxCount)
        .map(([number, _]) => number)

      columnStats.push({
        winners,
        maxCount,
        numberCounts
      })
    }

    const generateConsensusWithBacktrack = (initialWinners, name) => {
      const universe = []
      const chosenIndices = []
      const usedNumbers = new Set()

      const backtrack = (col) => {
        if (col === 6) return true

        const colResults = results[col]
        const winners = columnStats[col].winners
        const previousCount = col > 0 ? universe[col - 1].count : -Infinity

        let candidates = colResults
          .map((item, index) => ({ ...item, index }))
          .filter(item => winners.includes(item.number))
          .filter(item => item.count > previousCount && !usedNumbers.has(item.number))

        candidates.sort((a, b) => {
          if (b.count !== a.count) {
            return b.count - a.count
          }
          return winners.indexOf(a.number) - winners.indexOf(b.number)
        })

        for (const candidate of candidates) {
          chosenIndices.push(candidate.index)
          usedNumbers.add(candidate.number)
          universe.push({
            column: col,
            index: candidate.index,
            number: candidate.number,
            count: candidate.count
          })

          if (backtrack(col + 1)) return true

          universe.pop()
          chosenIndices.pop()
          usedNumbers.delete(candidate.number)
        }

        return false
      }

      if (backtrack(0)) {
        return universe
      }

      return null
    }

    const mainUniverse = generateConsensusWithBacktrack(null, 'Consensus Principal')
    let generatedUniverses = []

    if (mainUniverse) {
      generatedUniverses.push({ items: mainUniverse, name: 'Consensus Principal' })
    }

    const tieColumns = columnStats
      .map((stat, col) => ({ col, stat }))
      .filter(({ stat }) => stat.winners.length > 1)

    if (tieColumns.length > 0) {

      const { col: firstTieCol } = tieColumns[0]
      const winnersCount = columnStats[firstTieCol].winners.length

      for (let i = 1; i < winnersCount; i++) {
        const generateAlternative = () => {
          const universe = []
          const chosenIndices = []
          const usedNumbers = new Set()

          const backtrack = (col) => {
            if (col === 6) return true

            const colResults = results[col]
            const previousCount = col > 0 ? universe[col - 1].count : -Infinity

            let candidates

            if (col === firstTieCol && columnStats[firstTieCol].winners.length > 1) {
              candidates = colResults
                .map((item, index) => ({ ...item, index }))
                .filter(item => item.number === columnStats[firstTieCol].winners[i])
                .filter(item => item.count > previousCount && !usedNumbers.has(item.number))
            } else {
              const winners = columnStats[col].winners
              candidates = colResults
                .map((item, index) => ({ ...item, index }))
                .filter(item => winners.includes(item.number))
                .filter(item => item.count > previousCount && !usedNumbers.has(item.number))
            }

            candidates.sort((a, b) => {
              if (b.count !== a.count) {
                return b.count - a.count
              }
              const winners = columnStats[col].winners
              return winners.indexOf(a.number) - winners.indexOf(b.number)
            })

            for (const candidate of candidates) {
              chosenIndices.push(candidate.index)
              usedNumbers.add(candidate.number)
              universe.push({
                column: col,
                index: candidate.index,
                number: candidate.number,
                count: candidate.count
              })

              if (backtrack(col + 1)) return true

              universe.pop()
              chosenIndices.pop()
              usedNumbers.delete(candidate.number)
            }

            return false
          }

          if (backtrack(0)) {
            return universe
          }

          return null
        }

        const altUniverse = generateAlternative()
        if (altUniverse) {
          generatedUniverses.push({ items: altUniverse, name: `Consensus Alternativo ${i}` })
        }
      }
    }

    return generatedUniverses
  }

  const consensusUniverses = generateConsensusUniverse()
  consensusUniverses.forEach(u => universesList.push(u))

  universes.value = universesList
}

const isHighlighted = (col, number) => {
  if (hoveredUniverse.value === null) return false
  const universe = universes.value[hoveredUniverse.value]
  if (!universe) return false
  const universeItem = universe.items.find(item => item.column === col)
  return universeItem && universeItem.number === number
}

const addManualUniverse = () => {
  const values = manualUniverse.value.map(v => parseInt(v))

  if (values.some(v => v === 0 || isNaN(v))) {
    alert('Todos los campos deben tener un valor numérico')
    return
  }

  const uniqueNumbers = new Set(values)
  if (uniqueNumbers.size !== 6) {
    alert('Los números deben ser únicos')
    return
  }

  if (values.some(v => v < 1 || v > 56)) {
    alert('Los números deben estar entre 1 y 56')
    return
  }

  const items = values.map((number, index) => {
    const colResult = calculatedResults.value[index]
    const item = colResult.find(r => r.number === number)
    return {
      column: index,
      index: colResult.findIndex(r => r.number === number),
      number,
      count: item ? item.count : 0
    }
  })

  const manualUniverseObj = {
    items,
    name: 'Universo Manual'
  }

  universes.value.push(manualUniverseObj)
  manualUniverse.value = [0, 0, 0, 0, 0, 0]
}

onMounted(() => {
  loadFromStorage()
})
</script>

<style lang="stylus">
.home
  padding 2rem
  max-width 1400px
  margin 0 auto
  display flex
  gap 2rem
  flex-wrap wrap

  .left-column
    flex 1
    min-width 400px

    h1
      color #42b883
      margin-bottom 2rem
      text-align center

    .roms-container
      display flex
      gap 2rem
      margin-bottom 2rem
      flex-wrap wrap

      .rom-section
        flex 1
        min-width 300px
        padding 1.5rem
        background #f5f5f5
        border-radius 8px

        h2
          color #35495e
          margin-bottom 1rem
          font-size 1.2rem

        input
          width 100%
          padding 0.5rem
          margin-bottom 1rem

        .upload-btn
          width 100%
          padding 0.75rem
          background #42b883
          color white
          border none
          border-radius 4px
          cursor pointer
          font-size 1rem
          transition background 0.3s
          margin-bottom 1rem

          &:hover:not(:disabled)
            background #3aa876

          &:disabled
            background #7cb89e
            cursor not-allowed

        .rom-info
          p
            margin 0.25rem 0
            color #666
            font-size 0.9rem

    .status
      text-align center
      padding 1rem
      background #f0f0f0
      border-radius 4px

      .status-content
        display flex
        align-items center
        justify-content center
        gap 1rem

        p
          margin 0
          color #35495e
          font-weight bold

        .status-actions
          display flex
          gap 0.5rem

          button
            padding 0.5rem
            color white
            border none
            border-radius 4px
            cursor pointer
            transition background 0.3s

          .download-btn
            background #42b883

            &:hover
              background #3aa876

          .clear-btn
            background #ff6b6b

            &:hover
              background #e55a5a

    .ram-section
      margin-top 2rem
      padding 1.5rem
      background #f5f5f5
      border-radius 8px

      h2
        color #35495e
        margin-bottom 1.5rem
        text-align center
        font-size 1.5rem

      .ram-content
        display grid
        grid-template-columns repeat(6, 1fr)
        gap 1rem

        .ram-column
          text-align center
          padding 1.5rem 1rem
          background white
          border-radius 8px
          border 2px solid #e0e0e0

          .ram-value
            color #42b883
            font-size 1.5rem
            font-weight bold
            margin-bottom 1rem

          .ram-input
            width 100%
            padding 0.5rem
            text-align center
            border 1px solid #ccc
            border-radius 4px
            font-size 1.2rem
            font-weight bold
            color #35495e

            -moz-appearance textfield

            &::-webkit-outer-spin-button,
            &::-webkit-inner-spin-button
              -webkit-appearance none
              margin 0

            &:focus
              outline none
              border-color #42b883

      .ram-buttons
        display flex
        justify-content center
        gap 1rem
        margin 2rem auto 0
        max-width 600px

        .load-ram-btn,
        .calculate-btn
          flex 1
          max-width 300px
          padding 1rem 2rem
          color white
          border none
          border-radius 8px
          cursor pointer
          font-size 1.1rem
          font-weight bold
          transition background 0.3s
          box-shadow 0 2px 4px rgba(0, 0, 0, 0.1)

        .load-ram-btn
          background #42b883

          &:hover
            background #3aa876
            box-shadow 0 4px 6px rgba(0, 0, 0, 0.15)

        .calculate-btn
          background #35495e

          &:hover
            background #2c3e50
            box-shadow 0 4px 6px rgba(0, 0, 0, 0.15)

        .load-ram-btn:active,
        .calculate-btn:active
          transform translateY(1px)
          box-shadow 0 1px 2px rgba(0, 0, 0, 0.1)

      .universes-section
        padding 1.5rem
        background white
        border-radius 8px
        border 2px solid #e0e0e0

        h2
          color #35495e
          margin-bottom 1.5rem
          text-align center
          font-size 1.3rem

        .universes-list
          .universe-item
            padding 1rem
            background #f9f9f9
            border-radius 6px
            margin-bottom 1rem
            cursor pointer
            transition background 0.3s

            &:hover
              background #f0f0f0

            h3
              color #42b883
              margin-bottom 1rem
              text-align center
              font-size 1.1rem

            .universe-content
              display grid
              grid-template-columns repeat(6, 1fr)
              gap 1rem

              .universe-column
                text-align center
                padding 0.1rem 0.75rem
                background white
                border-radius 6px
                border 2px solid #42b883

                .universe-number
                  color #35495e
                  font-weight bold
                  font-size 1.2rem
                  margin-bottom 0.25rem

                .universe-count
                  color #42b883
                  font-size 1rem

            &.consensus
              h3
                color #9b59b6

              .universe-content
                .universe-column
                  border 2px solid #9b59b6

                  .universe-count
                    color #9b59b6

        .manual-universe-section
          margin-top 2rem
          padding 1.5rem
          background #f5f5f5
          border-radius 8px
          border 2px solid #e0e0e0

          .manual-universe-inputs
            display grid
            grid-template-columns repeat(6, 1fr)
            gap 1rem
            margin-bottom 1.5rem

            .manual-input
              input
                width 100%
                padding 0.5rem
                text-align center
                border 1px solid #ccc
                border-radius 4px
                font-size 1.1rem
                font-weight bold
                color #35495e
                -moz-appearance textfield

                &::-webkit-outer-spin-button,
                &::-webkit-inner-spin-button
                  -webkit-appearance none
                  margin 0

                &:focus
                  outline none
                  border-color #42b883

          .add-manual-btn
            width 100%
            padding 1rem
            background #42b883
            color white
            border none
            border-radius 6px
            cursor pointer
            font-size 1.1rem
            font-weight bold
            transition background 0.3s

            &:hover
              background #3aa876

            &:active
              transform translateY(1px)

  .right-column
    flex 1
    min-width 400px

    .results-section
      padding 1.5rem
      background white
      border-radius 8px
      border 2px solid #e0e0e0

      .results-grid
        display grid
        grid-template-columns repeat(6, 1fr)
        gap 1rem

        .result-column
          background #f9f9f9
          border-radius 6px
          padding 0

          .result-item
            display flex
            align-items center
            justify-content space-between
            padding 0.4rem 0
            font-size 0.9rem
            color #35495e
            padding 0 0.75rem

            &.highlighted
              background #fffacd
              border-radius 4px

            &.zero-count
              color #3498db

              &.highlighted
                background #fffacd

              .result-number,
              .result-arrow,
              .result-count
                color #3498db

            &.bios-item
              color #e74c3c

              &.highlighted
                background #fffacd
                color #e74c3c

              .result-number,
              .result-arrow,
              .result-count
                color #e74c3c

            .result-number
              font-weight bold

            .result-arrow
              color #666
              margin 0 0.25rem

            .result-count
              color #666

  .hidden-input
    display none
</style>
