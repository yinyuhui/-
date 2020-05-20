<template lang="pug">
div
    .flex.flex-s
        el-button.mgl-8(type="primary" @click="add") add
    .flex.flex-s
        el-input.input.mgl-8(v-model.number="id")
        el-button.mgl-8(type="primary" @click="getItemById") getItemById
    .flex.flex-s
        el-input.input.mgl-8(v-model.number="age") 
        el-button.mgl-8(type="primary" @click="update") update
    .flex.flex-s
        el-input.input.mgl-8(v-model.number="deleteId") 
        el-button.mgl-8(type="primary" @click="deleteItem") deleteItem
    .flex.flex-s
        el-button.mgl-8(type="primary" @click="getAll") getAll
    .flex.flex-s
        el-input.input.mgl-8(v-model.number="lower") 
        el-input.input.mgl-8(v-model.number="upper") 
        el-button.mgl-8(type="primary" @click="filterByAge") filterByAge
</template>

<script>
	export default {
		name: 'indexedDB',
		data() {
			return {
				testDB: null,
				id: null,
				age: '',
				getResult: {},
				deleteId: null,
				lower: null,
				upper: null,
			}
		},
		mounted() {
			// 如果浏览器支持
			if ('indexedDB' in window) {
				// 打开数据库 [数据库名, 版本号]
				const openResult = indexedDB.open('testDB', 1)

				// 首次打开或版本号变化时进入执行
				openResult.onupgradeneeded = (e) => {
					let db = e.target.result
					// 如果不存在目标对象存储 就创建一个
					if (!db.objectStoreNames.contains('Person')) {
						let PersonOS = db.createObjectStore('Person', {
							keyPath: 'id', // 主键设置为id，也可以设置为一个值唯一的字段
							autoIncrement: true, // id是个自增值
						})
						// 创建一个age上的索引 [索引名, 数据属性, 索引设置]，索引不是必须设置的
						PersonOS.createIndex('age', 'age', { unique: false })
					}
				}
				openResult.onsuccess = (e) => {
					// 数据库成功打开后 存起来便于使用
					this.testDB = e.target.result
				}
			}
		},

		methods: {
			// 创建数据
			add() {
				if (this.testDB) {
					// 获取事务
					let transaction = this.testDB.transaction(
						'Person',
						'readwrite'
					)
					// 获取对象存储
					let store = transaction.objectStore('Person')

					// 要存储的数据，可以做个表单收集用户输入
					let person = {
						name: 'yyh',
						age: 18,
						email: 'veinyin@gmail.com',
					}

					// 操作都是异步的，所以有下面的两个钩子
					let request = store.add(person)

					request.onsuccess = (e) => {
						console.log('onsuccess', e)
					}
					request.onerror = (e) => {
						console.log('onerror', e)
					}
				}
			},

			// 获取数据
			getItemById() {
				// 所有操作都有这两步，获取事务、获取对象存储
				const transaction = this.testDB.transaction(
					'Person',
					'readonly'
				)
				const store = transaction.objectStore('Person')

				// 传入主键值，必传，只能获取一条数据
				const request = store.get(this.id)

				request.onsuccess = (e) => {
					this.getResult = e.target.result || {}
					console.log(this.getResult)
				}
				request.onerror = (e) => {
					console.log(e)
				}
			},

			// 更新
			update() {
				const transaction = this.testDB.transaction(
					'Person',
					'readwrite'
				)
				const store = transaction.objectStore('Person')
				const person = {
					...this.getResult,
					age: this.age,
				}
				const request = store.put(person)
				request.onsuccess = (e) => {
					console.log(e)
				}
				request.onerror = (e) => {
					console.log(e)
				}
			},

			// 删除
			deleteItem() {
				const transaction = this.testDB.transaction(
					'Person',
					'readwrite'
				)
				const store = transaction.objectStore('Person')
				const request = store.delete(this.deleteId)
				request.onsuccess = (e) => {
					console.log(e)
				}
				request.onerror = (e) => {
					console.log(e)
				}
			},

			// 获取所有数据
			getAll() {
				const transaction = this.testDB.transaction(
					'Person',
					'readonly'
				)
				const store = transaction.objectStore('Person')

				// 在对象存储上打开游标
				const cursor = store.openCursor()

				// 结果列表
				let resultList = []

				cursor.onsuccess = (e) => {
					// 一条一条查，如果没结束就会continue查下一条
					const cursor = e.target.result
					if (cursor) {
						resultList.push(cursor.value)
						// 下一循环
						cursor.continue()
					}
				}

				// 查询结束，可以拿着结果操作了
				transaction.oncomplete = () => {
					console.log(resultList)
				}
			},

			// 获取部分数据
			filterByAge() {
				if (!this.lower && !this.upper) return

				const transaction = this.testDB.transaction(
					'Person',
					'readonly'
				)
				const store = transaction.objectStore('Person')

				// 获取索引
				const index = store.index('age')

				let range
				// 根据数据设置范围
				if (this.lower && this.upper) {
					// 开区间
					range = IDBKeyRange.bound(this.lower, this.upper, true)
				} else if (this.lower) {
					// 默认是闭区间
					range = IDBKeyRange.lowerBound(this.lower)
				} else {
					range = IDBKeyRange.upperBound(this.upper)
				}

				// 结果列表
				let resultList = []

				// 在索引上打开游标 默认正序，加上'prev'参数就是逆序
				const cursor = index.openCursor(range, 'prev')
				cursor.onsuccess = (e) => {
					const cursor = e.target.result
					if (cursor) {
						resultList.push(cursor.value)
						cursor.continue()
					}
				}
				transaction.oncomplete = (e) => {
					console.log(resultList)
				}
			},
		},
	}
</script>

<style lang="stylus" scoped>
	.flex
	    display flex
	    align-items center
	    margin 8px
	.flex-s
	    justify-content flex-start
	.mgl-8
	    margin-left 8px
	.input
	    width 50px
</style>
