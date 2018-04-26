<template>
    <div ref="gantt"></div>
</template>

<script>
    import 'dhtmlx-gantt'
    import axios from 'axios'

    export default {
        name: 'gantt',
        props: {
            tasks: {
                type: Object,
                default () {
                    return {data: [], links: []}
                }
            }
        },

        methods: {
            $_initGanttEvents: function () {
                // scheduler.config.time_step = 15;
                gantt.config.scale_unit = "hour";
                gantt.config.multiselect = true;
                // gantt.config.duration_step = 15;
                gantt.config.date_scale = "%H:00";
                gantt.config.duration_unit = "minute";
                gantt.config.columns = [
                    {name:"wbs", label:"WBS", width:30, template:gantt.getWBSCode },
                    {name:"text",       label:"タスク名",  width:"*", tree:true, width: 150},
                    {name:"start_date", label:"開始時間", align:"center", resize: true, width: 100},
                    {name:"duration",   label:"工数",   align:"center", resize: true },
                    {name:"add",        label:"",           width:44 }
                ];
                // gantt.config.duration_step = 1;
                gantt.config.fit_tasks = true;
                gantt.config.scale_height = 70;
                gantt.config.min_column_width = 50;
                gantt.config.scale_offset_minimal = false;
                gantt.config.subscales = [
                  {unit:"minute", step:15, date : "%i"}
                ];
                // gantt.config.lightbox.sections=[
                //     {name:"description", height:38, map_to:"text", type:"textarea", focus:true},
                //     {name: "time", height: 72, type: "duration", map_to: "auto"}
                // ];
                // gantt.config.time_step = 15;
                if(gantt.$_eventsInitialized)
                    return;
                gantt.attachEvent('onTaskSelected', (id) => {
                    let task = gantt.getTask(id)
                    this.$emit('task-selected', task)
                })

                gantt.attachEvent('onAfterTaskAdd', (id, task) => {
                    this.$emit('task-updated', id, 'inserted', task)
                    task.progress = task.progress || 0
                    if (task.parent == 0) {
                      task.color = "#0CD329"
                    } else {
                        task.color = "#3DB9D3"
                    }
                    if(gantt.getSelectedId() == id) {
                        this.$emit('task-selected', task)
                    }
                    var s = task.start_date
                    var start_date = s.getDate()+"-"+(s.getMonth()+1)+"-"+s.getFullYear()+" "+s.toLocaleTimeString()
                    axios.post("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/data", {
                        id: task.id,
                        text: task.text,
                        start_date: start_date,
                        duration: task.duration,
                        progress: task.progress,
                        color: task.color,
                        parent: task.parent
                    })
                })

                gantt.attachEvent('onAfterTaskUpdate', (id, task) => {
                    console.log("hoge")
                     if (task.parent == 0) {
                         var s = task.start_date
                         var start_date = s.getDate()+"-"+(s.getMonth()+1)+"-"+s.getFullYear()+" "+s.toLocaleTimeString()
                         axios.post("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/data/"+task.id, {
                             text: task.text,
                             start_date: start_date,
                             duration: task.duration,
                             progress: task.progress,
                         })
                       return;
                    }
                    var parentTask = gantt.getTask(task.parent);
                    var children = gantt.getChildren(parentTask.id);
                    var totProgress = 0;
                    var tempTask;
                    children.forEach(function (child) {
                      tempTask = gantt.getTask(child);
                      totProgress += parseFloat(tempTask.progress);
                    });
                    parentTask.progress = (totProgress / children.length).toFixed(2);
                    gantt.updateTask(parentTask.id);
                    this.$emit('task-updated', id, 'updated', task)
                    var s = task.start_date
                    var start_date = s.getDate()+"-"+(s.getMonth()+1)+"-"+s.getFullYear()+" "+s.toLocaleTimeString()
                    axios.post("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/data/"+task.id, {
                        text: task.text,
                        start_date: start_date,
                        duration: task.duration,
                        progress: task.progress,
                    })
                })

                gantt.attachEvent('onAfterTaskDelete', (id) => {
                    this.$emit('task-updated', id, 'deleted')
                    if(!gantt.getSelectedId()) {
                        this.$emit('task-selected', null)
                    }
                    axios.delete("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/data/"+id)
                })

                gantt.attachEvent('onAfterLinkAdd', (id, link) => {
                    this.$emit('link-updated', id, 'inserted', link)
                    axios.post("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/links", {
                        id: id,
                        source: link.source,
                        target: link.target,
                        type: link.type

                    })
                })

                gantt.attachEvent('onAfterLinkUpdate', (id, link) => {
                    this.$emit('link-updated', id, 'updated', link)
                    axios.post("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/links"+id, {
                        source: link.source,
                        target: link.target,
                        type: link.type
                    })
                })

                gantt.attachEvent('onAfterLinkDelete', (id, link) => {
                    this.$emit('link-updated', id, 'deleted')
                    axios.delete("http://wbsserver.3epfniimky.ap-northeast-1.elasticbeanstalk.com/api/links/"+id)
                })
                gantt.$_eventsInitialized = true;
            }
        },

        mounted () {
            this.$_initGanttEvents();

            gantt.init(this.$refs.gantt)
            // gantt.init("gantt_here")
            gantt.parse(this.$props.tasks)
        }
    }
</script>

<style>
    @import "~dhtmlx-gantt/codebase/dhtmlxgantt.css";
</style>
