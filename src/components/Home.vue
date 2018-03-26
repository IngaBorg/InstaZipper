<template>
    <div>
        <div class="header clearfix">
            <h1 class="display-1 mb-3">InstaZipper</h1>
            <div class="border-top divider float-left"></div>
            <div class="border-top divider float-right"></div>
        </div>

        <div class="row mt-3">
            <div class="col align-self-center">
                <div class="form-group">
                    <input
                        type="text"
                        class="form-control form-control-lg"
                        placeholder="@instagram"
                        v-on:keyup="usernameKeyUp" :value="result">
                </div>
            </div>
        </div>
        <process class="process" v-show="processing" :progress="progress" :total="igCount"></process>
    </div>
</template>

<script>
import Velocity from 'velocity-animate'
import Process from './Process'
import axios from 'axios'
import JSZip from 'jszip'
import FileSaver from 'file-saver'
export default {
  components: {
    Process
  },
  data: function () {
    return {
      result: null,
      processing: false,
      url: 'https://www.instagram.com/graphql/query/',
      igResponse: null,
      igCount: 0,
      queryHash: '472f257a40c653c64c666ce877d59d2b',
      variables: {
        'id': '5823115431',
        'first': 200
      },

      progress: 0,
      total: 0,
      zip: null
    }
  },
  methods: {
    usernameKeyUp: function (event) {
      if (event.keyCode === 13) {
        this.$data.processing = true
        this.$data.result = Math.random()

        Velocity(document.getElementsByClassName('divider'), {
          width: '50%'
        }, {
          duration: '1000'
        })

        Velocity(event.target, {
          transform: 'translateY(100px)',
          opacity: 0
        }, {
          complete: function (elements, activeCall) {
            elements[0].style = 'display:none'
          },
          queue: 'input'
        })
        Velocity(document.getElementsByClassName('process')[0], {
          transform: 'translateY(-40px)',
          opacity: 1
        }, {
          queue: 'input'
        })

        this.fetchData()
      }
    },
    fetchData: function () {
      let that = this
      axios.get(this.$data.url, {
        params: {
          'query_hash': this.$data.queryHash,
          'variables': JSON.stringify(this.$data.variables)
        }})
        .then(function (response) {
          that.$data.igResponse = response.data.data.user.edge_owner_to_timeline_media.edges
          that.$data.igCount = response.data.data.user.edge_owner_to_timeline_media.count
          that.archiver()
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    archiver: function () {
      this.$data.zip = new JSZip()
      let promises = []
      let that = this

      this.$data.igResponse.forEach(element => {
        if (element.node.__typename === 'GraphImage') {
          promises.push(
            axios.get(element.node.display_url, {
              responseType: 'arraybuffer'
            })
          )
        }
      })

      axios.all(promises)
        .then(axios.spread((...args) => {
          for (let i = 0; i < args.length; i++) {
            console.log(args[i].response.data)
            // if (args[i].status === 200 && args[i].hasOwnProperty('response')) {
            //   let arrayBuffer = args[i].response.data
            //   let url = args[i].request.responseURL
            //   let fileName = url.split('/').pop()
            //   console.log(arrayBuffer)
            //   that.$data.zip.file(fileName, arrayBuffer, {binary:true})
            // }
          }
        }))
        .then(function (data) {
          // that.$data.zip.generateAsync({type: 'blob'})
          //   .then(function (blob) {
          //     FileSaver.saveAs(blob, 'InstaZipper.zip')
          //   })
          console.log(data)
        })
        .catch(function (error) {
          console.log(error)
        })
    }
  }
}
</script>

<style>
input.form-control {
  z-index: -1;
}

.divider {
  width: 0%;
}

.process {
  opacity: 0;
}
</style>
