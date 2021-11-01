<template>
  <h3>{{ status }}</h3>
  <div v-if="online" class="ipfs-info">
    <h3>
      ID: <span id="ipfs-info-id">{{ id }}</span>
    </h3>
    <h3>
      Agent version: <span id="ipfs-info-agent">{{ agentVersion }}</span>
    </h3>
QmT9qk3CRYbFDWpDFYeAv8T8H1gnongwKhh5J68NLkLir6<br>
QmWXdjNC362aPDtwHPUE9o2VMqPeNeCQuTBTv1NsKtwypg<br>
QmPChd2hVbrJ6bfo3WBcTW4iZnpHm8TEzWkLHmLpXhF68A<br>
Qmd7NtbnfwNF5wVcF9EiUA3MfC4Ns6FenSgyxnu7P5CWX2<br><br><br>
    filename: <input v-model="filename" /><br>
    textToSend: <input v-model="textToSend" /><br><br>
    <button @click="getFile">get data</button><br>
    <button @click="sendFile">sendFile</button>
    <button @click="getDir">getDir</button>
  </div>
</template>

<script>
import { CID } from 'multiformats/cid'
import { base64 } from "multiformats/bases/base64"

export default {
  name: "IpfsInfo",
  data: function() {
    return {
      filename: '',
      textToSend: '',
      //
      status: "Connecting to IPFS...",
      id: "",
      agentVersion: "",
      online: false
    };
  },
  setup(){

  },
  mounted: function() {
    this.getIpfsNodeInfo();
  },
  methods: {
    sendFile: async function(){

         const data = {
            path: '/tmp/my_test_data_file.txt',
            content: this.textToSend || 'Hello IPFS! ' + Math.random()
          }

         if(this.online) {
              const ipfs = await this.$ipfs;
              const result = await ipfs.add(data)

              console.info(result)

              console.info(result.cid.toString())

         }
    },
    getDir: async function(){
      if(this.online) {
        const ipfs = await this.$ipfs;

          for await (const file of ipfs.get(this.filename)) {
            // do something with buf
            console.log(file.toString())
          }
      }
    },
    getFile: async function(){
      
      if(this.online) {
        const ipfs = await this.$ipfs;
        const stream = ipfs.cat(this.filename)
        let data = ''

        for await (const chunk of stream) {
          // chunks of data are returned as a Buffer, convert it back to a string
          data += chunk.toString()
        }

        console.log(data)
      }
    },
    async getIpfsNodeInfo() {
      try {
        // Await for ipfs node instance.
        const ipfs = await this.$ipfs;
        // Call ipfs `id` method.
        // Returns the identity of the Peer.
        const { agentVersion, id } = await ipfs.id();
        this.agentVersion = agentVersion;
        this.id = id;
        // Set successful status text.
        this.status = "Connected to IPFS =)";
        this.online = ipfs.isOnline();
      } catch (err) {
        // Set error status text.
        this.status = `Error: ${err}`;
      }
    }
  }
};
</script>
