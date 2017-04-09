<style>

</style>

<template>

    <label class="weui-cell weui-check__label" for="s11">
        <div class="weui-cell__hd">
            <input type="checkbox" class="weui-check" :id="'list-'+index">
            <i class="weui-icon-checked"></i>
        </div>
        <div class="weui-cell__bd">
            <p>{{content}}</p>
        </div>
    </label>
</template>
<script>
export default {
    name: 'todoItem',
    props: {
        content: String,
        index: Number
    },
    created() {

    },
    methods: {

    }
}
</script>