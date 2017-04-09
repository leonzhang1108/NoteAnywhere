<style lang="stylus" rel="stylesheet/stylus">
    @import 'common/stylus/index'
</style>

<template>
	<div>
		<todolist></todolist>
	</div>
</template>

<script>
import todolist from 'components/TodoList/TodoList'
import axios from 'axios'
import $ from 'webpack-zepto'
import Fastclick from 'fastclick'
window.$ = $

Fastclick.attach(document.body)

const ERR_OK = 0

export default {
	components: {
		'todolist': todolist
	}
}

</script>
