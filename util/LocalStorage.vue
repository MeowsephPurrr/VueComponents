<script>
/**
 * @fileoverview This component provides a mechanism to automatically save and
 * restore data to/from the browser's localStorage. It is designed to persist
 * data for a specific route, such as form data, across page reloads.
 *
 * @description When mounted, the component checks for saved data corresponding
 * to the current URL. If data is found, it prompts the user via a dialog to
 * either load or discard it. The data to be persisted is bound using the
 * `v-model:storedValue` directive. Any changes to the bound data are
 * automatically saved.
 *
 * @example
 * <!-- In your parent component -->
 * <template>
 *   <form>
 *     <input v-model="formData.name" placeholder="Name" />
 *     <input v-model="formData.email" placeholder="Email" />
 *   </form>
 *   <LocalStorage v-model:storedValue="formData" ref="localStorageRef" />
 * </template>
 *
 * <script>
 * import LocalStorage from './util/LocalStorage.vue';
 *
 * export default {
 *   components: { LocalStorage },
 *   data() {
 *     return {
 *       formData: { name: '', email: '' }
 *     }
 *   }
 * }
 * <'''>
 */
import Dialog from 'primevue/dialog';
import Button from 'primevue/button';
import {useRoute} from "vue-router";
import {capitalize} from "vue";
import Toaster from "./Toaster.vue";

export default {
  name: "LocalStorage",
  components: {
    Toaster,
    Dialog,
    Button,
  },
  /**
   * @property {Object} storedValue - The data object to be persisted in
   *   localStorage. This should be used with `v-model:storedValue`.
   */
  props: {
    storedValue: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      showLoadDialog: false,
      storageKey: null,
    }
  },
  /**
   * @event update:storedValue
   * @description Emitted when the user confirms to load the stored data.
   *   This event is used to update the parent component's data, enabling
   *   the `v-model` functionality.
   * @type {Object}
   */
  emits: ['update:storedValue'],
  watch: {
    storedValue: {
      handler(newValue) {
        if (newValue) {
          localStorage.setItem(this.storageKey, JSON.stringify(newValue));
        }
      },
      deep: true
    },
  },
  created() {
    const route = useRoute();
    this.storageKey = route.fullPath;
  },
  mounted() {
    if (localStorage.getItem(this.storageKey)) {
      this.showLoadDialog = true;
    }
  },
  methods: {
    capitalize,
    /**
     * Loads data from localStorage, emits an update event, and then clears
     * the stored data.
     */
    loadStoredData() {
      const storedData = localStorage.getItem(this.storageKey);
      if (storedData) {
        try {
          const parsedData = JSON.parse(storedData);
          this.$emit('update:storedValue', parsedData);
        } catch (e) {
          this.$refs.toasterRef.toast(
              "errors.storage.load",
              null,
              "warn",
              null,
              3000
          );
        } finally {
          // Clean up the storage after loading to prevent re-prompting
          // on the next page load unless the data changes again.
          localStorage.removeItem(this.storageKey);
        }
      }
      this.showLoadDialog = false;
    },
    /**
     * Manually removes the data from localStorage for the current route.
     * Can be called from a parent component via a ref.
     */
    removeStoredData() {
      localStorage.removeItem(this.storageKey);
    },
    /**
     * Rejects loading the stored data and removes it from localStorage.
     */
    rejectStoredData() {
      localStorage.removeItem(this.storageKey);
      this.showLoadDialog = false;
    },
  }
};
</script>

<template>
  <Dialog
      :visible="showLoadDialog"
      :modal="true"
      :closable="false"
      :style="{ width: '50vw' }"
      :draggable="false"
      :resizable="false"
  >
    <template #header>
      <div class="centered-header">
        <span class="p-dialog-title">{{ capitalize($t('load saved data for this form')) }}?</span>
      </div>
    </template>
    <template #footer>
      <div class="centered-footer">
        <Button severity="secondary" @click="rejectStoredData" autofocus> {{ capitalize($t('no')) }}</Button>
        <Button severity="success" @click="loadStoredData"> {{ capitalize($t('yes')) }}</Button>
      </div>
    </template>
  </Dialog>
  <Toaster ref="toasterRef"/>
</template>

<style scoped>
.centered-header {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.centered-footer {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
}

.centered-footer > .p-button {
  flex-grow: 1;
}
</style>
