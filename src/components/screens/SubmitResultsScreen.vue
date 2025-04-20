<docs>
  Use this screen at the end of your experiment to submit the data to the server.
  You can provide the submission URL to the Experiment component.
</docs>

<template>
  <Screen v-if="!$magpie.debug" title="Submitting">
    <Slide>
      <p>{{ $t('screens.SubmitResultsScreen.waiting') }}</p>
      <Wait :time="0" @done="submit(() => $magpie.nextSlide())" />
    </Slide>
    <Slide>
      <p v-if="!error">
        {{ $t('screens.SubmitResultsScreen.done') }}
        <Wait :time="3000" @done="redirectToCompletionUrl" />
      </p>
      <div v-else>
        <p>{{ $t('screens.SubmitResultsScreen.error') }}</p>
        <p>{{ $t('screens.SubmitResultsScreen.manual') }}</p>
        <p>
          {{ $t('screens.SubmitResultsScreen.contact') }}
          <a :href="'mailto:' + $magpie.contactEmail" target="_blank" rel="noopener">
            {{ $magpie.contactEmail }}
          </a>
        </p>
        <p v-text="" />
        <button style= "bottom: 30%; transform: translate(-50%, -50%)" @click="downloadCsv">
          {{ $t('screens.SubmitResultsScreen.download') }}
        </button>
        <p>
          {{ $t('screens.SubmitResultsScreen.upload') }}
          <a :href="$magpie.uploadUrl" target="_blank" rel="noopener">
            {{ $magpie.uploadUrl }}
          </a>
        </p>
        <p>{{ $t('screens.SubmitResultsScreen.thanks') }}</p>
      </div>
    </Slide>
  </Screen>
  <DebugResultsScreen v-else />
</template>

<script>
import Screen from '../Screen';
import Wait from '../helpers/Wait';
import DebugResultsScreen from './DebugResultsScreen';
import Slide from '../Slide';
import stringify from 'csv-stringify/lib/sync';

export default {
  name: 'SubmitResultsScreen',
  components: { Slide, DebugResultsScreen, Wait, Screen },
  props: {},
  data() {
    return {
      error: null,
      results: [],
      csv: ''
    };
  },
  methods: {
    async submit(cb) {
      try {
        await this.$magpie.submit();
        cb();
      } catch (err) {
        this.results = this.$magpie.getAllData();
        if (this.results.length) {
          this.csv = stringify(this.results, {
            columns: Object.keys(this.results[0]),
            header: true
          });
        }
        this.error = err.message;
        cb();
      }
    },
    redirectToCompletionUrl() {
      if (this.$magpie.completionUrl && this.$magpie.mode === 'prolific') {
        window.location = this.$magpie.completionUrl;
      }
    },
    downloadCsv() {
      const blob = new Blob([this.csv], {
        type: 'text/plain',
        endings: 'native'
      });
      const element = document.createElement('a');
      const filename =
        'data-' +
        this.$magpie.id +
        '-' +
        new Date().toISOString().slice(0, 16).replace(/[:T]/g, '-') +
        '.csv';
      const objectUrl = URL.createObjectURL(blob);
      element.setAttribute('href', objectUrl);
      element.setAttribute('download', filename);
      element.style.display = 'none';
      document.body.appendChild(element);
      element.click();
      URL.revokeObjectURL(objectUrl);
      document.body.removeChild(element);
    }
  }
};
</script>
