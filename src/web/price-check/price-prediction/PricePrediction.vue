<template>
  <div :style="{ 'min-height': !showContrib ? '4.375rem' : undefined }">
    <div v-if="loading" class="py-2 flex justify-center pr-4">
      <div>
        <i class="fas fa-dna fa-spin text-gray-600"></i>
      </div>
      <div class="pl-2 text-center">
        <div>Getting price prediction...</div>
        <div class="text-gray-600">Powered by poeprices.info</div>
      </div>
    </div>
    <div v-else-if="price">
      <div class="flex items-center pb-4">
        <item-quick-price class="flex-1"
          :min="price.min"
          :max="price.max"
          approx
          :item-img="item.icon"
          :currency="price.currency === 'exalt' ? 'exa' : 'chaos'"
        />
        <div class="text-center">
          <div class="leading-tight">
            <i v-if="price.confidence < 78" class="fas fa-exclamation-triangle pr-1 text-orange-400"></i>
            <span>{{ price.confidence }}{{ '\u2009' }}%</span>
          </div>
          <div class="text-xs text-gray-500 leading-none">Confidence</div>
        </div>
      </div>
      <div v-if="!showContrib" class="flex justify-between items-center">
        <button @click="showContrib = true" class="btn">Contribution to predicted price<i class="fas fa-chevron-down btn-icon ml-2"></i></button>
        <div class="flex" v-if="!feedbackSent && (price.confidence < 80)">
          <feedback-option :item="item" :prediction="price" @sent="feedbackSent = true" option="low" />
          <feedback-option :item="item" :prediction="price" @sent="feedbackSent = true" option="fair" />
          <feedback-option :item="item" :prediction="price" @sent="feedbackSent = true" option="high" />
        </div>
      </div>
      <table v-else>
        <thead>
          <th></th>
          <th class="text-gray-500 text-left font-normal pl-2">
            <button @click="showContrib = false">Contribution to predicted price<i class="fas fa-chevron-up btn-icon ml-2"></i></button>
          </th>
        </thead>
        <tbody class="align-top">
          <tr v-for="expl in price.explanation" :key="expl.name">
            <td class="text-right text-gray-500 whitespace-no-wrap">{{ expl.contrib }}&nbsp;%</td>
            <td class="pl-2 truncate w-full" style="max-width: 0;">{{ expl.name }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-else-if="error">
      <div class="text-red-400">Failed to get price prediction</div>
      <div>Error: {{ error }}</div>
    </div>
  </div>
</template>

<script>
import { requestPoeprices } from './poeprices'
import FeedbackOption from './FeedbackOption'
import ItemQuickPrice from '@/web/ui/ItemQuickPrice'

export default {
  name: 'PricePrediction',
  components: { FeedbackOption, ItemQuickPrice },
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      price: null,
      error: null,
      loading: false,
      showContrib: false,
      feedbackSent: false
    }
  },
  watch: {
    item: {
      immediate: true,
      async handler (item) {
        try {
          this.loading = true
          this.error = null
          this.price = null
          this.showContrib = false
          this.feedbackSent = false
          this.price = await requestPoeprices(this.item)
        } catch (err) {
          this.error = err.message
        } finally {
          this.loading = false
        }
      }
    }
  }
}
</script>
