<template lang="pug">
div
	h2 Timeline

	input-timeinterval(v-model="daterange", v-bind:duration="timeintervalDefaultDuration")

	table
		tr
			th.pr-2
				label(for="mode") Filter:
			td
				input(type="text", v-model="filterText", placeholder="Filter event titles…")

	div(v-show="buckets !== null")
		div
			div(style="float: left")
				| Events shown:  {{ num_events }}
			div(style="float: right; color: #999")
				| Drag to pan and scroll to zoom.
		div(style="clear: both")
		vis-timeline(:buckets="filteredBuckets", showRowLabels=true, :queriedInterval="daterange")
	div(v-show="!(buckets !== null && num_events)")
		h1 Loading...
</template>

<script>
import moment from 'moment';
import _ from 'lodash';

export default {
  name: 'Timeline',
  data: () => {
    return {
      buckets: null,
      daterange: [moment().subtract(1, 'hour'), moment()],
      timeintervalDefaultDuration: localStorage.durationDefault,
      filterText: '',
    };
  },
  computed: {
    num_events() {
      return _.sumBy(this.filteredBuckets, 'events.length');
    },
    filteredBuckets() {
      console.warn(this.buckets);
      if (this.filterText.length)
        return this.buckets.map(b => {
          return {
            ...b,
            // events: b.events.filter(e => !e.data.title || e.data.title.toLowerCase().includes(this.filterText.toLowerCase())),
            events: b.events.filter(e => e.data.title && e.data.title.toLowerCase().includes(this.filterText.toLowerCase())),
          };
        });
      else return this.buckets;
    }
  },
  watch: {
    daterange() {
      this.getBuckets();
    },
  },
  mounted: function() {
    this.getBuckets();
  },
  methods: {
    getBuckets: async function() {
      this.buckets = await this.$store.dispatch('buckets/getBucketsWithEvents', {
        start: this.daterange[0].format(),
        end: this.daterange[1].format(),
      });
    },
  },
};
</script>
