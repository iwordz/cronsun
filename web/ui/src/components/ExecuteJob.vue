<template>
  <div class="ui modal">
    <i class="close icon"></i>
    <div class="header">执行任务 {{jobName}}</div>
    <div class="content">
      <Dropdown title="选择执行的节点" :items="nodes" v-on:change="changeNode"></Dropdown>
    </div>
    <div class="actions">
      <div class="ui deny button">取消</div>
      <div class="ui positive right labeled icon button">立刻执行任务 <i class="checkmark icon"></i></div>
    </div>
  </div>
</template>

<script>
import Dropdown from './basic/Dropdown.vue';

export default {
  name: 'execute-job',
  data(){
    return {
      jobGroup: '',
      jobId: '',
      jobName: '',
      nodes: [],
      selectedNode: '',
      loading: false
    }
  },
  
  mounted(){},

  methods: {
    show(jobName, jobGroup, jobId){
      this.jobName = jobName;
      this.jobGroup = jobGroup;
      this.jobId = jobId;
      this.fetchJobNodes();
      $(this.$el).modal({
        closable: false,
        onApprove: this.submit
      }).modal('show');
    },

    hide(){
      $(this.$el).modal('hide');
    },

    fetchJobNodes(){
      var vm = this;
      this.loading = true;
      this.$rest.GET('job/'+this.jobGroup+'-'+this.jobId+'/nodes').
        onsucceed(200, (resp)=>{
          resp.unshift('全部节点');
          vm.nodes = resp;
        }).
        onfailed((msg)=>{
          vm.$bus.$emit('error', msg);
          vm.hide();
        }).
        onend(()=>{vm.loading = false}).
        do();
    },

    submit(){
      var vm = this;
      this.loading = true;
      var node = this.selectedNode === '全部节点' ? '' : this.selectedNode;
      this.$rest.PUT('/job/'+this.jobGroup+'-'+this.jobId+'/execute?node='+node).
        onsucceed(204, ()=>{
          vm.$bus.$emit('success', '执行命令已发送，注意查看任务日志');
          vm.hide();
        }).
        onfailed((msg)=>{vm.$bus.$emit('error', msg)}).
        onend(()=>{vm.loading = false}).
        do();
      return false;
    },

    changeNode(val){
      this.selectedNode = val;
    }
  },

  components: {
    Dropdown
  }
}
</script>