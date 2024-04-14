<template>
  <div class="combo-box-container">
    <div class="combo-box" :class="{ 'overlay-on': isDropdownOpen }">
      <div class="input-container">
        <input
          v-model="inputValue"
          type="text"
          :class="{ 'no-match': isNoMatchFound }"
          @input="onInput"
          @keydown="onKeyDown"
          @keydown.enter="onEnter"
          @focus="onFocus"
        />
        <button :class="{ 'button-active': isDropdownOpenByButton }" @click="onButtonClick">▼</button>
      </div>
      <ul v-show="isDropdownOpen" :id="dropdownId" class="options-container">
        <li
          v-for="(option, index) in displayedOptions"
          :key="index"
          :class="{ highlighted: index === highlightedIndex }"
          @click="selectOption(option)"
          @mouseover="highlightOption(index)"
          @mouseleave="unhighlightOption"
        >
          <span
            v-for="(part, partIndex) in splitText(option.label)"
            :key="partIndex"
            :class="{ 'matched-text': isMatchedText(part) }"
          >
            {{ part }}
          </span>
        </li>
        <li v-if="isNoMatchFound" class="no-match-message">該当するものがありません</li>
      </ul>
    </div>
    <div v-if="isDropdownOpen" class="overlay" @click="closeDropdown"></div>
  </div>
</template>

<script lang="ts" setup>
import { computed, defineEmits, defineProps, ref } from 'vue';

interface Option {
  value?: string | number;
  label?: string;
}

interface Props {
  options: Option[];
  modelValue: Option;
  noMatchFound: boolean;
}

const props = defineProps<Props>();

const emit = defineEmits<{
  (event: 'update:modelValue', value: Option): void;
  (event: 'update:noMatchFound', value: boolean): void;
}>();

const highlightedIndex = ref<number>(-1); // 現在ハイライトされているオプションのインデックス、-1 はハイライトされていない状態
const isDropdownOpen = ref(false); // ドロップダウンが開いているかどうかのフラグ
const dropdownId = 'combo-box-dropdown'; // ドロップダウンのID
const isFullOptionsDisplayed = ref(false); // 全てのオプションを表示するかどうかのフラグ
const isDropdownOpenByButton = ref(false); // ボタンによってドロップダウンが開かれたかどうかのフラグ

const inputValue = computed({
  get: () => props.modelValue.label || '',
  set: (value) => {
    const matchedOption = props.options.find((option) => option.label?.toLowerCase() === value.toLowerCase());

    if (matchedOption) {
      emit('update:modelValue', matchedOption);
      emit('update:noMatchFound', false);
    } else {
      emit('update:modelValue', { label: value });
      emit('update:noMatchFound', true);
    }
  }
});

const isNoMatchFound = computed(() => {
  // 入力値と一致するオプションが見つからないかどうかの算出プロパティ
  return (
    inputValue.value !== '' && !props.options.some((option) => option.label?.toLowerCase() === inputValue.value.toLowerCase())
  );
});

const displayedOptions = computed(() => {
  // 表示されるオプションのリストを返す算出プロパティ
  if (isFullOptionsDisplayed.value) {
    return props.options;
  } else if (inputValue.value === '') {
    return props.options;
  } else {
    return props.options.filter((option) => option.label?.toLowerCase().includes(inputValue.value.toLowerCase()));
  }
});

const onInput = () => {
  isFullOptionsDisplayed.value = false;
};

const onKeyDown = (event: KeyboardEvent) => {
  if (event.key === 'ArrowUp') {
    event.preventDefault();
    const newIndex = Math.max(highlightedIndex.value - 1, 0);

    highlightOption(newIndex);
    scrollToOption(newIndex);
  } else if (event.key === 'ArrowDown') {
    event.preventDefault();
    const newIndex = Math.min(highlightedIndex.value + 1, displayedOptions.value.length - 1);

    highlightOption(newIndex);
    scrollToOption(newIndex);
  } else {
    isDropdownOpen.value = true;
  }
};

const onEnter = () => {
  if (highlightedIndex.value >= 0) {
    emit('update:modelValue', displayedOptions.value[highlightedIndex.value]);
  } else {
    emit('update:modelValue', { label: inputValue.value });
  }
  closeDropdown();
};

const onFocus = () => {
  isDropdownOpen.value = true;
};

const onButtonClick = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
  isDropdownOpenByButton.value = isDropdownOpen.value;
  isFullOptionsDisplayed.value = true;
};

const selectOption = (option: Option) => {
  inputValue.value = option.label || '';
  highlightedIndex.value = -1;
  closeDropdown();
  emit('update:modelValue', option);
};

const highlightOption = (index: number) => {
  highlightedIndex.value = index;
  scrollToOption(index);
};

const unhighlightOption = () => {
  highlightedIndex.value = -1;
};

const scrollToOption = (index: number) => {
  const optionsList = document.getElementById(dropdownId);
  const option = optionsList?.children[index] as HTMLElement;

  if (option) {
    const optionsTop = optionsList?.scrollTop || 0;
    const optionsBottom = optionsTop + (optionsList?.clientHeight || 0);
    const optionTop = option.offsetTop;
    const optionBottom = optionTop + option.clientHeight;

    if (optionTop < optionsTop) {
      optionsList?.scrollTo({ top: optionTop, behavior: 'smooth' });
    } else if (optionBottom > optionsBottom) {
      optionsList?.scrollTo({
        top: optionBottom - (optionsList?.clientHeight || 0),
        behavior: 'smooth'
      });
    }
  }
};

const closeDropdown = () => {
  isDropdownOpen.value = false;
  isDropdownOpenByButton.value = false;
  isFullOptionsDisplayed.value = false;
};

const splitText = (text: string | undefined) => {
  if (!text) return [];
  const searchTerm = inputValue.value.toLowerCase();

  return text.split(new RegExp(`(${searchTerm})`, 'gi'));
};

const isMatchedText = (text: string) => {
  const searchTerm = inputValue.value.toLowerCase();

  return text.toLowerCase() === searchTerm;
};
</script>

<style scoped>
.combo-box-container {
  position: relative;
}

.combo-box {
  position: relative;
  width: 500px;
}

.input-container {
  display: flex;
}

.input-container input {
  flex: 1;
  padding: 5px;
  font-size: 16px;
}

.input-container button {
  padding: 5px 10px;
  font-size: 16px;
  background-color: #f0f0f0;
  border: none;
  cursor: pointer;
}

.input-container button.button-active {
  background-color: #e0e0e0;
}

.options-container {
  position: absolute;
  font-size: 14px;
  top: 100%;
  left: 0;
  width: 100%;
  max-height: 200px;
  overflow-y: auto;
  background-color: #fff;
  border: 1px solid #ccc;
  border-top: none;
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.options-container li {
  padding: 5px;
  cursor: pointer;
}

.options-container li.highlighted {
  background-color: #f0f0f0;
}

.matched-text {
  font-weight: bold;
  text-decoration: underline;
}

.input-container input.no-match {
  color: #999;
}

.no-match-message {
  padding: 5px;
  color: #999;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: transparent;
  z-index: 10;
}

.overlay-on {
  z-index: 11;
}
</style>
