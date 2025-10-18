<script>
/**
 * @fileoverview This component provides a global toast notification system.
 *
 * It wraps the PrimeVue Toast component and exposes a `toast` method
 * to allow other components to trigger toast notifications.
 *
 * @example
 *    How to use in another component:
 *    1. Import and register the Toaster component.
 *    2. Add the Toaster to your template with a ref:
 *       <Toaster ref="toaster" />
 *    3. Call the toast method using the ref:
 *       this.$refs.toaster.toast('Summary', 'Message', 'success');
 */
import {capitalize} from "vue";
import {useI18n} from "vue-i18n";
import {useToast} from "primevue/usetoast";
import Toast from 'primevue/toast';

export default {
  name: "Toaster",
  components: {
    Toast,
  },
  setup() {
    const {t} = useI18n();
    const primeToast = useToast();

    /**
     * Displays a toast notification.
     *
     * @param {string} summary - The summary text for the toast. This will be translated.
     * @param {string} message - The detail message for the toast. This will be translated.
     * @param {string} severity - The severity of the toast (e.g., 'success', 'info', 'warn', 'error').
     * @param {?string=} summaryPrefix - Optional prefix for the summary, which will be translated.
     * @param {?number=} life - The duration in milliseconds for which the toast should be displayed.
     */
    const toast = (summary, message, severity, summaryPrefix = null, life = null) => {
      primeToast.add({
        group: "genericToast",
        severity: severity,
        summary: summary ? capitalize(summaryPrefix ? t(summaryPrefix) : '' + ' ' + t(summary)) : "",
        detail: message ? capitalize(t(message)) : "",
        life: life
      });
    };

    return {
      toast,
    };
  },
  expose: ['toast'],
}
</script>

<template>
  <Toast group="genericToast"/>
</template>
