<script>
/**
 * @fileoverview A textarea component with modes for standard input, read-only
 * display, and inline editing.
 *
 * @description This component wraps PrimeVue's Textarea, providing a consistent
 * look and feel. It includes a floating label and auto-resizing functionality.
 * It supports a `readOnly` mode that displays the text content, with an option
 * to expand and collapse long text. An `allowEdit` mode enables inline editing
 * of the content.
 *
 * @example
 * <!-- Basic usage with v-model -->
 * <Textarea
 *   id="description"
 *   title="Description"
 *   v-model="formData.description"
 *   :errors="formErrors.description"
 * />
 *
 * <!-- Read-only display with inline editing -->
 * <Textarea
 *   id="notes"
 *   title="Notes"
 *   v-model="userData.notes"
 *   :read-only="true"
 *   :allow-edit="true"
 * />
 */
import Textarea from 'primevue/textarea';
import FloatLabel from 'primevue/floatlabel';

import apiCall from "@/util/Fetch";
import {capitalize} from "vue";
import InputText from "primevue/inputtext";
import Check from "../icons/Check.vue";
import Edit from "../icons/Edit.vue";
import X from "../icons/X.vue";
import Search from "../icons/Search.vue";
import SearchOff from "../icons/SearchOff.vue";

export default {
  name: "Input",
  methods: {
    capitalize,
    toggleEdit() {
      this.isEditing = !this.isEditing;
    },
    dismissEdit() {
      this.tmpVal = this.modelValue;
      this.toggleEdit();
    },
    acceptEdit() {
      this.$emit('update:modelValue', this.tmpVal);
      this.toggleEdit();
    },
  },
  data() {
    return {
      isEditing: false,
      tmpVal: null,
      fullRead: false,
    }
  },
  /**
   * @property {String} id - The unique ID for the textarea element.
   * @property {String} title - The label text for the textarea, which will be translated.
   * @property {String} modelValue - The binding for the textarea's value, used with `v-model`.
   * @property {Object|Array|null} [errors] - A list of validation errors to display.
   * @property {Boolean} [readOnly=false] - If true, the component displays as text instead of a textarea.
   * @property {Boolean} [allowEdit=false] - If true and in read-only mode, enables inline editing.
   */
  props: {
    id: String,
    title: String,
    modelValue: String | Array(Number),
    errors: Object | Array | null,
    readOnly: {
      type: Boolean,
      default: false,
    },
    allowEdit: {
      type: Boolean,
      default: false,
    },
  },
  /**
   * @event update:modelValue
   * @description Emitted to update the bound `v-model` value.
   * @type {String}
   */
  emits: ['update:modelValue'],
  components: {
    SearchOff,
    Search,
    X,
    Edit,
    Check,
    InputText,
    Textarea,
    FloatLabel,
    apiCall,
  },
  computed: {
    textareaAsPlainText() {
      return this.modelValue ? this.modelValue.replace(/\n/g, '<br>') : '';
    }
  },
  watch: {
    modelValue: {
      handler(_new, _old) {
        this.$nextTick(
            () => {
              if (this.readOnly) {
                this.tmpVal = this.modelValue;
                return;
              }
              document.querySelector(`#${this.id}`).dispatchEvent(new Event('input'))
            }
        );
      },
      once: true
    }
  },
}
</script>

<template>
  <div v-if="!readOnly" class="row-element">
    <FloatLabel variant="on">
      <Textarea :id="id" :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" rows="5" cols="30"
                autoresize :invalid="!!errors" style="width: 100%"/>
      <label :for="id">{{ capitalize($t(title)) }}</label>
    </FloatLabel>
    <span class="error-text" v-if="errors">{{ $t(errors?.[0]) }}</span>
  </div>

  <div v-else class="readonly-element">
    <div>{{ capitalize($t(title)) }}</div>

    <div class="edit-row-element">
      <div v-if="!isEditing" class="readOnly-textarea" :class="{'full-read': fullRead}" v-html="textareaAsPlainText"/>

      <div v-else class="inline-edit-input">
        <Textarea :id="id" v-model="tmpVal" rows="5"
                  cols="30"
                  autoresize :invalid="!!errors" style="width: 100%"/>
        <span v-if="errors" class="error-text">{{ $t(errors?.[0]) }}</span>
      </div>

      <div v-if="allowEdit" class="edit-controls">
        <template v-if="!isEditing">
          <Search v-if="!fullRead" class="icon" @click="() => {fullRead=true}"/>
          <SearchOff v-if="fullRead" class="icon" @click="() => {fullRead=false}"/>
          <!-- TODO: Edit this to be an svg or a component with an edit icon for use -->
          <Edit class="icon" @click="toggleEdit"/>
        </template>
        <template v-else>
          <!-- TODO: Edit this to be an svg or a component with a check icon for use -->
          <Check class="icon" @click="acceptEdit"/>
          <!-- TODO: Edit this to be an svg or a component with a dismiss icon for use -->
          <X class="icon" @click="dismissEdit"/>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
.readonly-element {
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-column-gap: 0.25rem;
  grid-row-gap: 0;
  min-height: 3rem;
  align-items: center;
}

.edit-row-element {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.edit-controls {
  margin-left: 0.25rem;
  margin-right: 0.25rem;
  display: flex;
}

.readOnly-textarea {
  max-height: 100px;
  width: 100%;
  overflow-y: auto;
  transition: 0.5s ease-in-out;
}

.readOnly-textarea.full-read {
  max-height: 100vh;
}

.icon:hover {
  cursor: pointer;
}
</style>
