<template>
  <div class="test">
    <h1>Import data</h1>
    <input type="url" v-on:keyup.enter="enterHit" placeholder="URL" />
    <p>
      {{rowCount}} rows processed
    </p>
    <p v-if="error" v-html="error"></p>
    <div class="table-responsive">
      <table class="table table-bordered table-striped">
        <tbody>
          <tr v-for="row in rows">
            <td v-for="value in row">{{value}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import * as _ from 'lodash'
  import {genUUID} from '../mixins/helpers'

  export default {
    name: 'DataImport',
    data() {
      return {
        rowCount: 0,
        badrowcount: 0,
        error: null,
        rows: [],
        state: 'pending',
        steps: [
          {
            verb: 'source',
            uuid: genUUID(),
            options: {
              revision: 0
            }
          }
        ]
      }
    },
    methods: {
      enterHit: function(e) {
        const urlToData = e.target.value
        // Generate pipeline desctiptor using url
        this.steps[0].options.revision = 1
        this.steps[0].options.url = urlToData
        let body = {
          actions: this.steps
        }
        this.$http.post('http://localhost:8000/config', body).then(response => {
          let source = new EventSource(`http://localhost:8000/events/${response.body.id}`)
          source.onmessage = (event) => {
            let rt = {}
            if (event.data.startsWith('{')) {
              rt = JSON.parse(event.data)
            }
            if (rt.e == 'start') {
              this.state = 'in-progress';
              this.rows = [];
              this.rowCount = 0;
              this.badrowcount = 0;
              this.error = null;
            } else if (rt.e == 'err') {
              if (!this.error) {
                this.error = '';
              }
              this.error += rt.msg + '<br/>';
            } else if (rt.e == 'done') {
              this.state = 'done'
            } else if (rt.e == 'r') {
              if (rt.idx >= this.rowCount) {
                let data = Object.values(rt.data)
                this.rows.push(data)
                this.rowCount = rt.idx + 1;
              }
            } else if (rt.e == 've') {
              this.badrowcount += 1;
              if (rt.idx >= this.rowCount) {
                let data = _.mapValues(rt.data, (v_) => {return {v: v_};});
                if (data[rt.field]) {
                  data[rt.field].klass = 'error';
                }
                this.rows.push({idx: rt.idx + 1, data: data})
                this.rowCount = rt.idx + 1;
              }
            } else if (rt.e == 'rs') {
              this.schema = rt.data;
            }
            // Close connection when all data is received:
            if (event.data === 'close') {
              source.close()
            }
          }
        })
      }
    }
  }
</script>

<style scoped>

</style>
