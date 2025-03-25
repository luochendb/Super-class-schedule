<template>
    <view>
        <SqTimetable @click="getCourse" :time="time" :image="image" :timeColor="timeColor" :courseList="courseList" :showNotWeek="true"
            :week="12"></SqTimetable>
    </view>
</template>
<script>
    import SqTimetable from '../../components/sq-timetable/sq-timetable.vue'
    export default {
        components: {
            SqTimetable
        },
        data() {
            return {
                time: [
                    ['8:00', '8:45'],
                    ['8:50', '9:35'],
                    ['9:55', '10:40'],
                    ['10:45', '11:30'],
                    ['14:00', '14:45'],
                    ['14:50', '15:35'],
                    ['15:55', '16:40'],
                    ['16:45', '17:30'],
                    ['19:00', '19:45'],
                    ['19:50', '20:35']
                ],
                image: '',
                colorArrays: ['#70eb55', '#ffc450', '#ff9bd7', '#86affe', '#58eea4', '#ffa17d', '#1cbbb4', '#6964ad',
                    '#d42245', '#218285'
                ],
                timeColor: '#000000',
                courseList: [{
                    id: 1000,
                    begin: 3,
                    end: 4,
                    week: 2,
                    name: '面向对象-JAVA实践教学',
                    week_begin: 8,
                    week_end: 8,//只在第8周有课
                    place: '理泽楼314',
                    teacher: '陆师'
                }, {
                    id: 1001,
                    begin: 1,
                    end: 2,
                    week: 4,
                    name: '计算机网络（双语）',
                    week_begin: 1,
                    week_end: 16,//在第1-到第16周有课
                    place: '理泽楼315',
                    teacher: '刘师'
                }]
            }
        },
        onLoad() {

        },
        methods: {
            getCourse(item) {
                console.log(item)
            }
        }
    }
</script>
<style>
</style>