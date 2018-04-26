<template>
  <div class="container">
    <div>
        <h1> {{ date.getFullYear()+"年"+(date.getMonth()+1)+"月"+date.getDate()+"日" }}</h1>
        <button type="button" class="btn btn-success" v-on:click="backPage()">＜</button>
        <button type="button" class="btn btn-success" v-on:click="nextPage()">＞</button>
        <!-- <button type="button" class="btn btn-primary" v-on:click="">Day</button>
        <button type="button" class="btn btn-primary" v-on:click="changeWeekScale()">Week</button>
        <button type="button" class="btn btn-primary" v-on:click="">Month</button> -->
    </div>
    <div class="right-container">
      <div class="gantt-selected-info">
        <div v-if="selectedTask">
          <h2>{{selectedTask.text}}</h2>
          <span><b>ID: </b>{{selectedTask.id}}</span><br/>
          <span><b>進捗: </b>{{selectedTask.progress|toPercent}}%</span><br/>
          <span><b>開始時間: </b>{{selectedTask.start_date|niceDate}}</span><br/>
          <span><b>終了時間: </b>{{selectedTask.end_date|niceDate}}</span><br/>
        </div>
        <div v-else class="select-task-prompt">
          <h2>Click any task</h2>
        </div>
      </div>
      <ul class="gantt-messages">
        <li class="gantt-message" v-for="message in messages">{{message}}</li>
      </ul>
    </div>
    <gantt class="left-container" :tasks="tasks" @task-updated="logTaskUpdate" @link-updated="logLinkUpdate" @task-selected="selectTask"></gantt>
  </div>
</template>

<script>
    import Gantt from './components/Gantt.vue'
    import axios from 'axios'
    import BootstrapVue from 'bootstrap-vue'
    import 'bootstrap/dist/css/bootstrap.css'
    import 'bootstrap-vue/dist/bootstrap-vue.css'

    export default {
        name: 'wbs_app',
        components: {Gantt},
        data () {
            return {
                tasks: {
                    data: [
                        // {id: 1, text: '日報', start_date: '21-04-2018 12:00', duration: 120, progress: 0.0, color: "#0CD329"},
                        // {id: 2, text: '1', start_date: '21-04-2018 12:00', duration: 60, progress: 0.0, parent: 1},
                        // {id: 3, text: '2', start_date: '21-04-2018 13:00', duration: 60, progress: 0.0, parent: 1}
                    ],
                    links: [
                        // {id: 1, source: 2, target: 3, type: '0'}
                    ]
                },
                selectedTask: null,
                messages: [],
                date: new Date(),
            }
        },
        filters: {
            toPercent (val) {
                if(!val) return '0'
                return Math.round((+val) * 100)
            },
            niceDate (obj){
                return `${obj.getFullYear()}年${obj.getMonth()+1}月${obj.getDate()}日 ${obj.getHours()}時${obj.getMinutes()}分`
            }
        },
        mounted:function() {
            var d = new Date()
            this.date = new Date(d.getFullYear(), d.getMonth(), d.getDate())
            var today = this.date.getDate()+"-"+(this.date.getMonth()+1)+"-"+this.date.getFullYear()
            this.getTasks(today)
        },
        methods: {
            selectTask (task) {
              // console.log(this.tasks)
                this.selectedTask = task
            },

            addMessage (message) {
                this.messages.unshift(message)
                if(this.messages.length > 40) {
                    this.messages.pop()
                }
            },

            logTaskUpdate (id, mode, task) {
                let text = (task && task.text ? ` (${task.text})`: '')
                let message = `タスク ${mode}: ${id} ${text}`
                this.addMessage(message)
            },

            logLinkUpdate (id, mode, link) {
                let message = `リンク ${mode}: ${id}`
                if(link){
                    message += ` ( ソース: ${link.source}, ターゲット: ${link.target} )`
                }
                this.addMessage(message)
            },

            async getTasks (date) {
              var tasks = {}
              gantt.clearAll();
              // var s = new Date()
              // var today = s.getDate()+"-"+(s.getMonth()+1)+"-"+s.getFullYear()
              await axios.get("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/data?start_date="+date)
                  .then(function (response){
                      tasks.data = response.data.data
                  })
                  .catch(function (error) {
                      console.log(error)
                  });
              await axios.get("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/links")
                .then(function (response){
                    tasks.links = response.data.links
                })
                .catch(function (error) {
                    console.log(error)
                })
                gantt.parse({
                    data: tasks.data,
                    links: tasks.links
                })
            },

            taskClear() {
              gantt.clearAll()
            },

            backPage () {
                // this.date.setDate(this.date.getDate() - 1);
                this.date = new Date(this.date.getFullYear(), this.date.getMonth(), this.date.getDate()-1)
                var back_date = this.date.getDate()+"-"+(this.date.getMonth()+1)+"-"+this.date.getFullYear()
                this.getTasks(back_date)
            },

            nextPage () {
                // this.date.setDate(this.date.getDate() + 1);
                this.date = new Date(this.date.getFullYear(), this.date.getMonth(), this.date.getDate()+1)
                var next_date = this.date.getDate()+"-"+(this.date.getMonth()+1)+"-"+this.date.getFullYear()
                this.getTasks(next_date)
            },

            changeWeekScale () {
                gantt.config.scale_unit = "week"
                gantt.config.date_scale = "%m月%d日";
                gantt.config.subscales = [
                ];

            },
        }
    }
</script>

<style>
  html, body {
    height: 100%;
    margin: 0;
    padding: 0;
    font-family: Arial;
    font-size: 10px;
  }

  .container {
    height: 100%;
    width: 100%;
  }


  .left-container {
    overflow: hidden;
    position: relative;
    height: 100%;
  }

  .right-container {
    border-right: 1px solid #cecece;
    float: right;
    height: 100%;
    width: 200px;
    box-shadow: 0 0 5px 2px #aaa;
    position: relative;
    z-index:2;
    font-size: 10px;
  }

  .gantt-messages {
    list-style-type: none;
    height: 50%;
    margin: 0;
    overflow-x: hidden;
    overflow-y: auto;
    padding-left: 5px;
  }

  .gantt-messages > .gantt-message {
    background-color: #f4f4f4;
    box-shadow:inset 5px 0 #d69000;
    font-family: Geneva, Arial, Helvetica, sans-serif;
    font-size: 9px;
    margin: 5px 0;
    padding: 8px 0 8px 10px;
  }

  .gantt-selected-info {
    border-bottom: 1px solid #cecece;
    box-sizing: border-box;
    font-family: Geneva, Arial, Helvetica, sans-serif;
    height: 50%;
    line-height: 28px;
    padding: 10px;
  }

  .gantt-selected-info h2 {
    border-bottom: 1px solid #cecece;
    font-size: 13px;
  }

  .select-task-prompt h2{
    color: #d9d9d9;
  }
</style>
