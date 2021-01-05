<template>
  <div class="container">
    <div class="text-container">
      <CustomInput
        id="customInput"
        v-model="customInput1"
        label="Custom Label"
      />

      <CustomInput
        id="customInput"
        v-model="customInput2"
        label="Custom Label"
      />
    </div>
  </div>
</template>

<script>
import CustomInput from '~/components/CustomInput'

export default {
  components: {
    CustomInput,
  },

  data() {
    return {
      customInput1: 'Hello',
      customInput2: '',
    }
  },
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: gray;
}

.text-container {
  background-color: white;
}
</style>
