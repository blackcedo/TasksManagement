
<template>
    <AppLayout title="Dashboard">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                My Tasks
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <div class="p-6 lg:p-8 bg-white border-b border-gray-200">
                        <h3 class="mt-8 text-2xl font-medium text-gray-900" style="text-align: center;">
                            Add / Update Task ✨
                        </h3>
                        <br/>
                        <hr/>
                        <br/>
                        <form @submit.prevent="save">
                            <div class="form-group">
                                <input v-model="task.name" type="text" name="name"  class="form-control" placeholder="Task Name" style="border-radius: 10px; border-color: rgb(211, 211, 211);"/><br/>
                                <input v-model="task.date" type="date" name="date"  class="form-control" placeholder="Complete Date" style="border-radius: 10px; border-color: rgb(211, 211, 211);"/><br/>
                                <button type="submit" class="btn btn-success" style="width: 150px; background-color: green;">Save Task</button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
               
        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                    <div class="p-6 lg:p-8 bg-white border-b border-gray-200">
                        <div class="mt-8 text-2xl font-medium text-gray-900" style="text-align: center;">
                            My All Tasks 📝
                        </div>
                        <br/>
                        <hr/>
                        <br/>
                        <nav aria-label="Page navigation example">
                            <ul class="pagination">
                                <li v-bind:class="[{disabled: !result.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="loadTasks(result.prev_page_url)">Previous</a></li>
                                <li class="page-item disabled"><a class="page-link" href="#"> Task {{ result.from }} To {{result.to}} </a></li>
                                <li v-bind:class="[{disabled: !result.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="loadTasks(result.next_page_url)">Next</a></li>
                            </ul>
                        </nav>
                        <div class="mt-8 text-2xl font-medium text-gray-900">
                            <div v-for="task in result.data" v-bind:key="task.id" class="alert" v-bind:class="{'alert-success': task.complete==1, 'alert-info': task.complete==0}">
                                <div class="row">
                                    <div class="col-md-10">
                                        <h2><b>{{task.name}}</b></h2>
                                        <p style="font-size: 15px;" >{{task.date}}</p>
                                        <div v-if="task.complete==1">
                                            <label style="font-size: 15px; color:green;">Completed !</label>
                                        </div>
                                        <div v-else>
                                            <label style="font-size: 15px; color:rgb(128, 128, 128);">Not Completed !</label>
                                        </div>
                                    </div>
                                    <div class="col-md-2">
                                        <div class="btn-group-vertical">
                                            <div class="btn-group-vertical" v-if="task.complete==1">
                                                <button class="btn btn-danger" style="margin-bottom: 5px;" @click="remove(task)">Delete</button>
                                                <button class="btn btn-success" @click="notComplete(task)">Not Complete</button>
                                            </div>
                                            <div class="btn-group-vertical" v-else>
                                                <button class="btn btn-warning" style="margin-bottom: 5px; width: 130px;" @click="edit(task)">Edit</button>
                                                <button class="btn btn-danger" style="margin-bottom: 5px; width: 130px;" @click="remove(task)">Delete</button>
                                                <button class="btn btn-success" style="width: 130px;" @click="complete(task)">Complete</button>
                                            </div>     
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </AppLayout>
</template>

<script >
import AppLayout from '@/Layouts/AppLayout.vue';
import axios from 'axios';

export default {
    components: {
        AppLayout
    },

    data(){
        return{
            result: {},
            task:{id:"", name:"", date:"", complete:0}
        }
    },

    created(){
        this.loadTasks();
    },

    mounted(){

    },

    methods:{
        loadTasks(url){
            var page = url || "api/tasks";
            axios.get(page)
            .then(({data})=>{
                this.result = data;
            })
        },

        save(){
            if(this.task.id == '')
                this.saveData();
            else  
                this.updateData();
        },

        saveData(){
            axios.post('api/tasks', this.task)
            .then(()=>{
                Swal.fire({
                    position: 'center',
                    icon: 'success',
                    title: 'Task Saved Successfuly !',
                    showConfirmButton: false,
                    timer: 1500
                })
                this.task.name = '';
                this.task.date = '';
                this.task.complete = 0;
                this.loadTasks();
            })
            .catch((err)=>{
                console.log(err);
            })
        },

        updateData(){
            var url = 'api/tasks/'+this.task.id;
            axios.put(url, this.task)
            .then(()=>{
                this.task.name = '';
                this.task.date = '';
                this.task.complete = 0;
                this.task.id = '';
                this.loadTasks();
                Swal.fire({
                    position: 'center',
                    icon: 'success',
                    title: 'Task Updated Successfuly !',
                    showConfirmButton: false,
                    timer: 1500
                })
            })
            .catch((err)=>{
                console.log(err);
            })
        },

        edit(task){
            this.task = task;
        },

        remove(task){
            var url = 'api/tasks/'+task.id;
              
            Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
            }).then((result) => {
                if (result.isConfirmed) {
                    axios.delete(url)
                    .then(()=>{
                        Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                        )
                        this.loadTasks();
                    })
                    .catch((err)=>{
                        console.log(err);
                    })
                }
            })
        },

        complete(task){
            this.task = task;
            this.task.complete = 1;

            var url = 'api/tasks/'+this.task.id;
            axios.put(url, this.task)
            .then(()=>{
                this.task.name = '';
                this.task.date = '';
                this.task.complete = 0;
                this.task.id = '';
                this.loadTasks();
                Swal.fire({
                    position: 'center',
                    icon: 'success',
                    title: 'Task Marked As Completed !',
                    showConfirmButton: false,
                    timer: 1500
                })
            })
            .catch((err)=>{
                console.log(err);
            })

        },

        notComplete(task){
            this.task = task;
            this.task.complete = 0;

            var url = 'api/tasks/'+this.task.id;
            axios.put(url, this.task)
            .then(()=>{
                this.task.name = '';
                this.task.date = '';
                this.task.complete = 0;
                this.task.id = '';
                this.loadTasks();
                Swal.fire({
                    position: 'center',
                    icon: 'success',
                    title: 'Task Marked As Not Completed !',
                    showConfirmButton: false,
                    timer: 1500
                })
            })
            .catch((err)=>{
                console.log(err);
            })

        }
    }
}

</script>

