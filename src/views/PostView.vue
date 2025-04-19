<template>
    <div>
        <form v-if="show_add" class="add">
            <input type="text" v-model="student.id" placeholder="学号" />
            <input type="text" v-model="student.name" placeholder="姓名" />
            <input type="text" v-model="student.age" placeholder="年龄" />
            <input type="text" v-model="student.gender" placeholder="性别" />
            <input type="text" v-model="student.birthday" placeholder="出生日期" />
            <input type="text" v-model="student.address" placeholder="地址" />
            <input type="text" v-model="student.phone" placeholder="电话" />
            <input type="text" v-model="student.email" placeholder="邮箱" />
            <button @click="addStudent()"> 添加 </button>
            <button @click="show_add = false"> 取消 </button>
        </form>
    </div>
</template>

<script>
</script>

<style scoped>
.add {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 20px;
}
</style>