<template>
  <div :class="$style['text-field__container']">
    <div
      :class="[
        $style['text-field'],
        { [$style['text-field--focused']]: isFocused },
        { [$style['text-field--invalid']]: hasError },
        { [$style['text-field--hovering']]: isHovering },
        { [$style['text-field--has-value']]: computedValue },
        { [$style['text-field--with-trailing-icon']]: trailingIcon },
      ]"
      @mouseout="isHovering = false"
      @mouseover="isHovering = true"
    >
      <UzoSprite
        v-if="trailingIcon && !hasError && computedValue && type !== 'password'"
        :class="$style['text-field__icon']"
        :height="20"
        :width="20"
        aria-hidden="true"
        name="cancel"
        @click="clearValue"
      />

      <UzoSprite
        v-if="hasError"
        :class="$style['text-field__icon']"
        :height="20"
        :width="20"
        aria-hidden="true"
        name="error"
      />

      <input
        :id="id"
        ref="input"
        v-model="computedValue"
        :autocomplete="autocomplete"
        :class="$style['text-field__input']"
        :name="name"
        :placeholder="placeholder"
        :type="computedType"
        :min="min"
        :max="max"
        :required="required"
        :maxlength="maxLength"
        :readonly="readonly"
        tabindex="0"
        @blur="onBlur"
        @focus="onFocus"
        v-on="{ ...listeners }"
      />
      <div
        :class="[
          $style['notched-outline'],
          { [$style['notched-outline--notched']]: isFocused || computedValue },
        ]"
      >
        <div :class="$style['notched-outline__leading']" />
        <div
          ref="inputLabel"
          :class="$style['notched-outline__notch']"
          :style="
            isFocused || computedValue
              ? { width: floatingLabelWidth + 'px' }
              : null
          "
        >
          <label
            :class="[
              $style['floating-label'],
              {
                [$style['floating-label--float-above']]:
                  isFocused || computedValue,
              },
            ]"
            :for="id"
            v-text="label"
          />
        </div>
        <div :class="$style['notched-outline__trailing']" />
      </div>
    </div>
    <div :class="$style['text-field__helper-line']">
      <div
        :class="[
          $style['text-field__helper-line__text'],
          $style['text-field__helper-line__text--persistent'],
          {
            [$style['text-field__helper-line__text--validation-msg']]: hasError,
          },
        ]"
      >
        <span v-text="hasError ? hasError : hint" />
      </div>
      <div
        v-if="maxLength"
        :class="[
          $style['text-field__character-counter'],
          $style['text-field__helper-line__text--persistent'],
        ]"
      >
        <span v-text="computedValue.length + ' / ' + maxLength" />
      </div>
    </div>
  </div>
</template>

<script>
import omit from 'lodash/omit'

export default {
  name: 'UzoInput',

  props: {
    // for v-model
    value: {
      type: String,
      default: '',
    },

    trailingIcon: {
      type: Boolean,
      default: true,
    },

    type: {
      type: String,
      default: 'text',
    },

    autocomplete: {
      type: String,
      default: undefined,
    },

    name: {
      type: String,
      default: '',
    },

    label: {
      type: String,
      required: true,
    },

    placeholder: {
      type: String,
      default: undefined,
    },

    id: {
      type: String,
      required: true,
    },

    required: {
      type: Boolean,
      default: undefined,
    },

    hint: {
      type: String,
      default: '',
    },

    errors: {
      type: Array,
      default: () => [],
    },

    readonly: {
      type: Boolean,
      default: undefined,
    },
    min: {
      type: Number,
      default: undefined,
    },
    max: {
      type: Number,
      default: undefined,
    },
    maxLength: {
      type: Number,
      default: undefined,
    },
  },

  data() {
    return {
      isFocused: false,
      isHovering: false,
      floatingLabelWidth: 0,
    }
  },

  computed: {
    listeners() {
      return omit(this.$listeners, 'input', 'focus', 'blur')
    },

    isValid() {
      if (this.computedType === 'number') {
        if (this.max && this.max < +this.computedValue) return false
        if (this.min && this.min > +this.computedValue) return false
      }
      return true
    },

    hasError() {
      if (!this.errors || !this.errors.length) return null

      return this.errors[0]
    },

    computedValue: {
      get() {
        return this.value
      },

      set(value) {
        let newValue = value
        if (this.maxLength) {
          if (newValue.length > this.maxLength) {
            newValue = newValue.slice(0, this.maxLength)
          }
        }
        if (this.computedType === 'number') {
          if (this.max && this.max < +newValue) newValue = `${this.max}`
          if (this.min && this.min > +newValue) newValue = `${this.min}`
        }
        this.$emit('input', newValue)
      },
    },

    computedType() {
      if (this.type === 'password' && this.passwordVisible) return 'text'

      return this.type
    },
  },

  watch: {
    isFocused: {
      handler(value) {
        this.$emit(value ? 'focus' : 'blur')
      },
    },

    computedValue(newVal) {
      if (this.maxLength && this.maxLength === newVal.length) this.$emit('full')
    },
  },

  mounted() {
    // SelectModalPicker.vue is making trouble because there is already a
    // value so it cannot compute the width.
    this.$nextTick(() =>
      setTimeout(() => {
        this.floatingLabelWidth =
          (this.$refs.inputLabel.clientWidth / 100) * 75 + 8
      }, 250)
    )
  },

  methods: {
    onFocus(event) {
      this.isFocused = true
      this.$emit('focus', event)
    },

    onBlur(event) {
      this.isFocused = false
      this.$emit('blur', event)
    },

    focus() {
      this.$refs.input.focus()
    },

    blur() {
      this.$refs.input.blur()
    },

    clearValue() {
      this.computedValue = ''
      return this.focus()
    },
  },
}
</script>

<style lang="scss" module>
@import './assets/input';
</style>
