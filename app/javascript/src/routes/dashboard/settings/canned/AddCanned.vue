<template>
  <modal :show.sync="show" :on-close="onClose">
    <div class="column content-box">
      <woot-modal-header
        :header-image="headerImage"
        :header-title="$t('CANNED_MGMT.ADD.TITLE')"
        :header-content="$t('CANNED_MGMT.ADD.DESC')"
      />
      <form class="row" v-on:submit.prevent="addAgent()">
        <div class="medium-12 columns">
          <label :class="{ 'error': $v.shortCode.$error }">
            {{ $t('CANNED_MGMT.ADD.FORM.SHORT_CODE.LABEL') }}
            <input type="text" v-model.trim="shortCode" @input="$v.shortCode.$touch" :placeholder="$t('CANNED_MGMT.ADD.FORM.SHORT_CODE.PLACEHOLDER')">
          </label>
        </div>

        <div class="medium-12 columns">
          <label :class="{ 'error': $v.content.$error }">
            {{ $t('CANNED_MGMT.ADD.FORM.CONTENT.LABEL') }}
            <input type="text" v-model.trim="content" @input="$v.content.$touch" :placeholder="$t('CANNED_MGMT.ADD.FORM.CONTENT.PLACEHOLDER')">
          </label>
        </div>
        <div class="modal-footer">
          <div class="medium-12 columns">
            <woot-submit-button
              :disabled="$v.content.$invalid || $v.shortCode.$invalid || addCanned.showLoading"
              :button-text="$t('CANNED_MGMT.ADD.FORM.SUBMIT')"
              :loading="addCanned.showLoading"
            />
            <a @click="onClose">Cancel</a>
          </div>
        </div>
      </form>
    </div>
  </modal>
</template>

<script>
/* global bus */
/* eslint no-console: 0 */
import { required, minLength } from 'vuelidate/lib/validators';

import PageHeader from '../SettingsSubPageHeader';
import WootSubmitButton from '../../../../components/buttons/FormSubmitButton';
import Modal from '../../../../components/Modal';

const cannedImg = require('assets/images/canned.svg');


export default {
  props: ['onClose'],
  components: {
    PageHeader,
    WootSubmitButton,
    Modal,
  },
  data() {
    return {
      shortCode: '',
      content: '',
      agentType: '',
      vertical: 'bottom',
      horizontal: 'center',
      addCanned: {
        showAlert: false,
        showLoading: false,
        message: '',
      },
      agentTypeList: this.$t('CANNED_MGMT.AGENT_TYPES'),
      show: true,
    };
  },
  computed: {
    headerImage() {
      return cannedImg;
    },
  },
  validations: {
    shortCode: {
      required,
      minLength: minLength(2),
    },
    content: {
      required,
    },
    agentType: {
      required,
    },
  },

  methods: {
    setPageName({ name }) {
      this.$v.agentType.$touch();
      this.agentType = name;
    },
    showAlert() {
      bus.$emit('newToastMessage', this.addCanned.message);
    },
    resetForm() {
      this.shortCode = this.content = '';
      this.$v.shortCode.$reset();
      this.$v.content.$reset();
    },
    addAgent() {
      // Show loading on button
      this.addCanned.showLoading = true;
      // Make API Calls
      this.$store.dispatch('addCannedResponse', {
        short_code: this.shortCode,
        content: this.content,
      })
      .then(() => {
        // Reset Form, Show success message
        this.addCanned.showLoading = false;
        this.addCanned.message = this.$t('CANNED_MGMT.ADD.API.SUCCESS_MESSAGE');
        this.showAlert();
        this.resetForm();
        this.onClose();
      })
      .catch(() => {
        this.addCanned.showLoading = false;
        this.addCanned.message = this.$t('CANNED_MGMT.ADD.API.ERROR_MESSAGE');
        this.showAlert();
      });
    },
  },
};
</script>
