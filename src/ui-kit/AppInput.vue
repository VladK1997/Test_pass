<script setup lang="ts">
import { ref, watch, computed, onMounted } from "vue";
import type { Ref } from "vue";
import debounce from "@/infrastructure/utils/debounce";
import AppIcon from "@/ui-kit/AppIcon.vue";

const emit = defineEmits([
  "handleInputValue",
  "clearSearch",
  "saveValue",
  "focusInput",
  "blurInput",
]);

const props = defineProps<{
  type?: string;
  isSearch?: boolean;
  isEdit?: boolean;
  label?: string;
  placeholder?: string;
  min?: number;
  max?: number;
  error?: boolean;
  errorMessage?: string[];
  default?: string | number | Date | null;
  disabled?: boolean;
  locked?: boolean;
  color?: string;
  noCross?: boolean;
  passwordReveal?: boolean;
}>();
watch(
  () => props.default,
  () => {
    if (props.default !== undefined) {
      inputValue.value = props.default;
    }
  }
);

const inputType = computed<"number" | "text" | "password">(() => {
  if (props.type === "password") return "password";
  if (["integer", "float", "decimal", "number"].includes(props.type)) {
    return "number";
  } else {
    return "text";
  }
});

const inputValue: Ref<string | number | Date | null> = ref(null);
const checkedValue = ref<boolean>(false);
const isShowValue = ref<boolean>(false);

watch(
  inputValue,
  debounce(
    () => {
      if (
        props.type === "integer" &&
        inputValue.value !== "" &&
        inputValue.value !== null
      ) {
        inputValue.value = Math.floor(+inputValue.value);
      } else if (
        ["float", "decimal"].includes(props.type) &&
        (inputValue.value as number) % 1 === 0 &&
        inputValue.value !== "" &&
        inputValue.value !== null
      ) {
        inputValue.value = (+(inputValue.value as number)).toFixed(1);
      }
      emit("handleInputValue", inputValue.value);
    },
    ["integer", "float", "decimal", "number"].includes(props.type) ? 1000 : 200
  )
);

function toggleReveal() {
  isShowValue.value = !isShowValue.value;
}

function clearSearch() {
  inputValue.value = "";
  checkedValue.value = false;
  emit("clearSearch");
}
function saveValue() {
  checkedValue.value = true;
  emit("saveValue", inputValue.value);
}

onMounted(() => {
  if (props.default || props.default === 0) {
    inputValue.value = props.default;
    if (props.isEdit) {
      checkedValue.value = true;
    }
  }
});
</script>

<template>
  <div class="input" :class="{ error: props.error }">
    <div class="input__label" v-if="label?.length">
      <p>{{ label }}</p>
      <div><slot name="label-append" /></div>
    </div>
    <div class="input__content">
      <svg class="input__search" v-if="props.isSearch">
        <use href="@/assets/icons/sprite.svg#search"></use>
      </svg>
      <input
        class="input__input"
        :class="{ search: props.isSearch, reveal: props.passwordReveal }"
        :style="{ backgroundColor: props.color }"
        :type="!isShowValue ? inputType : 'text'"
        :min="props.min"
        :max="props.max"
        :step="props.step"
        :placeholder="props.placeholder"
        :readonly="checkedValue"
        :disabled="props.disabled || props.locked"
        v-model="inputValue"
        @focus="$emit('focus')"
        @blur="$emit('blur', ref)"
      />
      <div class="input__icon">
        <AppIcon name="lock" size="xl" v-if="locked" />
      </div>
    </div>
    <div class="input__error" v-if="props.error">
      <p v-for="error in props.errorMessage">
        {{ error.$message }}
      </p>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import "src/assets/styles/base/typography";
.input {
  position: relative;
  width: 100%;
  height: 100%;
  // width: 35rem;
  // height: 4rem;
  &__search {
    position: absolute;
    top: 1.1rem;
    left: 1.2rem;
    width: 1.8rem;
    height: 1.8rem;
    color: $color-grey-200;
  }

  &__input {
    width: 100%;
    height: 100%;
    background: none;
    border: none;
    color: $color-grey-100;
    @extend .body-16;
    appearance: textfield;
    -moz-appearance: textfield;
    &.reveal {
      padding: 0 6.5rem 0 1.5rem;
    }
    &.search {
      padding: 0 4.3rem;
    }
    &:placeholder {
      color: $color-grey-700;
      opacity: 1;
    }
    &:focus-visible {
      outline: none;
    }
    &::-webkit-outer-spin-button,
    &::-webkit-inner-spin-button {
      -webkit-appearance: none;
    }
  }

  &__label {
    color: $color-grey-200;
    display: flex;
    gap: 8px;
    align-items: center;
    @extend .body-12 !optional;
    margin-bottom: 4px;
  }

  &.error &__label {
    color: $color-red-base;
  }

  &__icon {
    position: absolute;
    right: 0;
    top: 1rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: $color-grey-50;
  }
  &__edit,
  &__check,
  &__clear {
    width: 2rem;
    height: 2rem;
  }
  &__clear {
    cursor: pointer;
  }
  &__edit {
    fill: $color-white;
  }
  &__check {
    fill: $color-white;
    cursor: pointer;
  }
  &__error {
    top: 100%;
    position: absolute;
    font-size: rem(12);
    right: 0;
    color: $color-red-base;
    margin-top: rem(20);
  }
}
</style>
