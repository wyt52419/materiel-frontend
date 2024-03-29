<template>
    <div style="height: 100%;">
        <!-- 系统 Logo -->
        <el-aside class="header-logo" :width="asideWidth">
            <div @click="$router.push({ name: 'Home' })">
                <a v-if="foldAside" style="font-size:18px;">{{language.adminCenter}}</a>
                <a v-else style="font-size:18px">{{language.admin}}</a>
            </div>
        </el-aside>

        <el-aside class="aside" :width="asideWidth" :class='"icon-size-" + iconSize'>
            <el-scrollbar style="height: 100%; width: 100%;">
                <!--
                        default-active 表示当前选中的菜单，默认为 HomePage。
                        collapse 表示是否折叠菜单，仅 mode 为 vertical（默认）可用。 
                        collapseTransition 表示是否开启折叠动画，默认为 true。
                        background-color 表示背景颜色。
                        text-color 表示字体颜色。
                    -->
                <el-menu :default-active="menuActiveName || 'material'" :collapse="!foldAside" :collapseTransition="false"
                 background-color="#FFFFFF" text-color="#8a979e" style="width: 100%;">

                    <!-- 库存管理 -->
					<el-submenu index="inventory">
						<template slot="title">
                            <i class="el-icon-coin"></i>
                            <span>{{language.inventory}}</span>
                        </template>
						<el-menu-item index="material" @click="$router.push({ name: 'material' })">
                            <i class="el-icon-cpu"></i>
                            <span slot="title">{{language.material}}</span>
                        </el-menu-item>
						<el-menu-item index="NewMaterial" @click="$router.push({ name: 'NewMaterial' })">
                            <i class="el-icon-document-add"></i>
                            <span slot="title">{{language.addMaterial}}</span>
                        </el-menu-item>
					</el-submenu>

					<!-- 入库管理 -->
					<el-submenu index="checkIn">
						<template slot="title">
                            <i class="el-icon-sell"></i>
                            <span>{{language.checkIn}}</span>
                        </template>
						<el-menu-item index="currentListIn" @click="$router.push({ name: 'currentListIn' })">
                            <i class="el-icon-shopping-cart-full"></i>
                            <span slot="title">{{language.currentList}}</span>
                        </el-menu-item>
						<el-menu-item index="listRecordIn" @click="$router.push({ name: 'listRecordIn' })">
                            <i class="el-icon-document"></i>
                            <span slot="title">{{language.listRecord}}</span>
                        </el-menu-item>
					</el-submenu>

					<!-- 出库管理 -->
					<el-submenu index="checkOut">
						<template slot="title">
                            <i class="el-icon-sold-out"></i>
                            <span>{{language.checkOut}}</span>
                        </template>
						<el-menu-item index="currentListOut" @click="$router.push({ name: 'currentListOut' })">
                            <i class="el-icon-shopping-cart-full"></i>
                            <span slot="title">{{language.currentList}}</span>
                        </el-menu-item>
						<el-menu-item index="listRecordOut" @click="$router.push({ name: 'listRecordOut' })">
                            <i class="el-icon-document"></i>
                            <span slot="title">{{language.listRecord}}</span>
                        </el-menu-item>
					</el-submenu>

                    <!-- 分类管理 -->
					<el-menu-item index="classification" @click="$router.push({ name: 'classification' })">
                        <i class="el-icon-s-grid"></i>
                        <span slot="title">{{language.classification}}</span>
                    </el-menu-item>

					<DynamicMenu v-for="menu in dynamicRoutes" :key="menu.meta.menuId" :menu="menu"></DynamicMenu>
                </el-menu>
            </el-scrollbar>
        </el-aside>
    </div>
</template>

<script>
	import {
		mapState,
		mapActions
	} from 'vuex'
	// import {
	// 	isURL
	// } from '@/utils/validate.js'
	import DynamicMenu from '@/views/dynamic/DynamicMenu.vue'
	export default {
		name: 'Aside',
		props: ['foldAside'],
		data() {
			return {
				// 保存当前选中的菜单
				// menuActiveName: 'home',
				// 保存当前侧边栏的宽度
				asideWidth: '200px',
				// 用于拼接当前图标的 class 样式
				iconSize: 'true'
			}
		},
		components: {
			DynamicMenu
		},
		computed: {
			...mapState('common', ['menuActiveName', 'mainTabs', 'dynamicRoutes']),
			// 国际化
			language() {
				return {
					adminCenter: this.$t("aside.adminCenter"),
					admin: this.$t("aside.admin"),
					homePage: this.$t("aside.homePage"),
					inventory: this.$t("aside.inventory"), 
					material: this.$t("aside.material"), 
					addMaterial: this.$t("aside.addMaterial"), 
					checkIn: this.$t("aside.checkIn"), 
					checkOut: this.$t("aside.checkOut"), 
					currentList: this.$t("aside.currentList"), 
					listRecord: this.$t("aside.listRecord"), 
					classification: this.$t("aside.classification")
				}
			}
		},
		methods: {
			...mapActions('common', ['updateMenuActiveName', 'updateMainTabs', 'updateMainTabsActiveName'])
		},
		watch: {
			// 监视是否折叠侧边栏，折叠则宽度为 64px。
			foldAside(val) {
				this.asideWidth = val ? '200px' : '64px'
				this.iconSize = val
			},
			// 监视路由的变化，每次点击菜单项时会触发
			$route(route) {
				// 路由变化时，修改当前选中的菜单项
				this.updateMenuActiveName(route.name)
				// 是否显示标签页
				if (route.meta.isTab) {
					// 判断当前标签页数组中是否存在当前选中的标签，根据标签名匹配
					let tab = this.mainTabs.filter(item => item.name === route.name)[0]
					// 若当前标签页数组不存在该标签，则向数组中添加标签
					if (!tab) {
						// 设置标签页数据
						tab = {
							name: route.name,
							params: route.params,
							query: route.query,
							type: isURL(route.meta.iframeUrl) ? 'iframe' : 'module',
							iframeUrl: route.meta.iframeUrl || ''
						}
						// 将数据保存到标签页数组中
						this.updateMainTabs(this.mainTabs.concat(tab))
					}
					// 保存标签页中当前选中的标签名
					this.updateMainTabsActiveName(route.name)
				}
			}
		}
	}
</script>

<style>
	.aside {
		margin-bottom: 0;
		height: 100%;
		max-height: calc(100% - 50px);
		width: 100%;
		max-width: 200px;
		/* background-color: #263238; */
		background-color: #F3F6FB;
		border-right: 1px solid	#E6E6FA;
		text-align: left;
		right: 0;
	}

	.header-logo {
		/* background-color: #17b3a3; */
		background-color: #3B77E3;
		text-align: center;
		height: 50px;
		line-height: 50px;
		width: 200px;
		font-size: 24px;
		color: #fff;
		font-weight: bold;
		margin-bottom: 0;
		cursor: pointer;
		overflow: hidden !important;
	}

	.el-submenu .el-menu-item {
		max-width: 200px !important;
	}

	.el-menu-item:hover {
		background-color: #E4F1FB !important;
	}

	.el-submenu__title:hover {
		background-color: #FFF !important;
	}

	.el-scrollbar__wrap {
		overflow-x: hidden !important;
	}

	.icon-size-false i {
		font-size: 30px !important;
	}

	.icon-size-true i {
		font-size: 18px !important;
	}
</style>
