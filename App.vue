<template>
    <div>
       <DatePicker v-model="now"></DatePicker>
       <!-- <DatePicker :value="now" @input="v=>now=v"></DatePicker> -->
    </div>
</template>
<script>
import DatePicker from './date-picker.vue'
export default {
   components:{
       DatePicker
   },
   data () {
       return {
           now: new Date()
       }
   },
   mounted () {
       this.now = new Date
   }
}
</script>
