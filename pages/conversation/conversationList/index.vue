<template>
	<view class="conversation_container">
		<chat-header ref="chatHeaderRef" />
		<view class="conversation_search">
			<u-search disabled :showAction="false" shape="square"></u-search>
		</view>
		<z-paging ref="paging" :fixed="false" :auto="false" default-page-size="20" :show-loading-more-no-more-view="false" :refresher-enabled="!storeIsSyncing" @query="queryList">
			<u-swipe-action ref="swipeWrapperRef" class="swipe_wrapper" v-show="!storeIsSyncing">
				<conversation-item v-for="item in storeConversationList" :key="item.conversationID"
					@closeAllSwipe="closeAllSwipe" :source="item" />
			</u-swipe-action>
		</z-paging>
				
		<view class="loading_wrap" v-if="storeIsSyncing">
			<u-loading-icon text="同步中"></u-loading-icon>
		</view>
	</view>
</template>

<script>
	import {
		mapGetters,
	} from "vuex";
	import ChatHeader from './components/ChatHeader.vue'
	import ConversationItem from './components/ConversationItem.vue'


	export default {
		components: {
			ChatHeader,
			ConversationItem
		},
		data() {
			return {
				refreshing: false
			};
		},
		computed: {
			...mapGetters(['storeConversationList', 'storeIsSyncing']),
		},
		onReady() {
			this.$nextTick(() => plus.navigator.closeSplashscreen())
		},
		watch: {
			storeIsSyncing(newVal) {
				if(!newVal){
					this.$nextTick(()=>this.$refs.paging.complete([...this.storeConversationList]));
				}
			}
		},
		methods: {
			async queryList(pageNo,pageSize){
				const data = await this.$store.dispatch('conversation/getConversationList',pageNo === 1)
				this.$nextTick(()=>this.$refs.paging.complete(data));
			},
			closeAllSwipe() {
				this.$refs.swipeWrapperRef.closeAll();
			},
		}
	}
</script>

<style lang="scss" scoped>
	.conversation_container {
		@include colBox(false);
		height: 100vh;
		overflow-y: hidden;
	}

	.conversation_search {
		padding: 24rpx 44rpx;
	}
	
	.z-paging-content {
		flex: 1;
	}

	.swipe_wrapper {
		@include colBox(false);
		flex: 1;
		width: 100%;
		overflow-y: auto;
	}

	.conversation_list {
		flex: 1;
		width: 100%;
		// position: relative;
		// z-index: -1;
	}

	.loading_wrap {
		    position: absolute;
		    top: 50%;
		    left: 50%;
			transform: translate(-50%,-50%);
	}

	/deep/.u-swipe-action-item__right__button__wrapper__text {
		-webkit-line-clamp: 2 !important;
		max-width: 32px;
	}
</style>
