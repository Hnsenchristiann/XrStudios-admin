<template>
    <div>
        <svg ref="gantt" />
    </div>
</template>

<script>
import Gantt from 'frappe-gantt';
export default {
    name: 'GanttChart',
    props: {
        viewMode: {
            type: String,
            required: false,
            default: 'Month'
        },
        tasks: {
            type: Array,
            required: true,
            default: []
        }
    },
    data () {
        return {
            gantt: {},
        }
    },
}
</script>

<style>

</style>