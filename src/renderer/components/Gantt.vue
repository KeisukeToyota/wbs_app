<template>
    <div ref="gantt"></div>
</template>

<script>
    import 'dhtmlx-gantt'

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
                gantt.config.scale_unit = "hour";
                gantt.config.multiselect = true;
                gantt.config.duration_step = 0;
                gantt.config.date_scale = "%H:00";
                gantt.config.duration_unit = "minute";
                gantt.config.columns = [
                    {name:"wbs", label:"WBS", width:40, template:gantt.getWBSCode },
                    {name:"text",       label:"タスク名",  width:"*", tree:true },
                    {name:"start_date", label:"開始時間", align:"center" },
                    {name:"duration",   label:"工数",   align:"center" },
                    {name:"add",        label:"",           width:44 }
                ];
                gantt.config.duration_step = 1;
                gantt.config.scale
                gantt.config.fit_tasks = true;
                gantt.config.scale_height = 70;
                gantt.config.lightbox.sections=[
                    {name:"description", height:38, map_to:"text", type:"textarea", focus:true},
                    {name: "time", height: 72, type: "duration", map_to: "auto"}
                ];
                gantt.config.time_step = 15;
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
                    }
                    if(gantt.getSelectedId() == id) {
                        this.$emit('task-selected', task)
                    }
                })

                gantt.attachEvent('onAfterTaskUpdate', (id, task) => {
                     if (task.parent == 0) {
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
                })

                gantt.attachEvent('onAfterTaskDelete', (id) => {
                    this.$emit('task-updated', id, 'deleted')
                    if(!gantt.getSelectedId()) {
                        this.$emit('task-selected', null)
                    }
                })

                gantt.attachEvent('onAfterLinkAdd', (id, link) => {
                    this.$emit('link-updated', id, 'inserted', link)
                })

                gantt.attachEvent('onAfterLinkUpdate', (id, link) => {
                    this.$emit('link-updated', id, 'updated', link)
                })

                gantt.attachEvent('onAfterLinkDelete', (id, link) => {
                    this.$emit('link-updated', id, 'deleted')
                })
                gantt.$_eventsInitialized = true;
            }
        },

        mounted () {
            this.$_initGanttEvents();

            gantt.init(this.$refs.gantt)
            gantt.parse(this.$props.tasks)
        }
    }
</script>

<style>
    @import "~dhtmlx-gantt/codebase/dhtmlxgantt.css";
</style>
