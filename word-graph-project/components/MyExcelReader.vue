<template>
  <input type="file" @change="onFileChange" />
</template>
<script>
import * as xlsx from 'xlsx';

export default {
  data() {
    return {
    }
  },
  methods: {
    onFileChange(event) {
      this.file = event.target.files ? event.target.files[0] : null;
      if (this.file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const bstr = e.target.result;
          const wb = xlsx.read(bstr, {type: 'binary'});
          const wsname = wb.SheetNames[0];
          const ws = wb.Sheets[wsname];
          const data = xlsx.utils.sheet_to_json(ws, {header: 1}); 
          this.$emit('submit', Object.values(data)); //2次元配列になっているindex0が発言,index1が発言者
        }
        reader.readAsBinaryString(this.file);
      }
    },
  }
}
</script>
