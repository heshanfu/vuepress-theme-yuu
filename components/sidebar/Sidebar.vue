<template>
	<div class="sidebar">
		<NavLinks />
		<slot name="top"></slot>
		<ul v-if="items.length" class="sidebar-links">
			<li v-for="(item, i) in items" :key="i">
				<SidebarGroup
					v-if="item.type === 'group'"
					:item="item"
					:first="i === 0"
					:open="i === openGroupIndex"
					:collapsable="item.collapsable || item.collapsible"
					@toggle="toggleGroup(i)"
				/>
				<SidebarLink v-else :item="item" />
			</li>
		</ul>
		<slot name="bottom"></slot>
	</div>
</template>

<script>
import SidebarGroup from './SidebarGroup.vue';
import SidebarLink from './SidebarLink.vue';
import NavLinks from '../nav/NavLinks.vue';
import { isActive } from '../../util.js';

function resolveOpenGroupIndex(route, items) {
	for (let i = 0; i < items.length; i++) {
		const item = items[i];
		if (item.type === 'group' && item.children.some(c => isActive(route, c.path))) {
			return i;
		}
	}
	return -1;
}

export default {
	components: { SidebarGroup, SidebarLink, NavLinks },

	props: ['items'],

	data() {
		return {
			openGroupIndex: 0,
		};
	},

	watch: {
		'$route'() {
			this.refreshIndex();
		},
	},

	created() {
		this.refreshIndex();
	},

	methods: {
		refreshIndex() {
			const index = resolveOpenGroupIndex(this.$route, this.items);

			if (index > -1) {
				this.openGroupIndex = index;
			}
		},

		toggleGroup(index) {
			this.openGroupIndex = index === this.openGroupIndex ? -1 : index;
		},

		isActive(page) {
			return isActive(this.$route, page.path);
		},
	},
};
</script>

<style lang="stylus">
@import '../../styles/config.styl';

.sidebar {
	ul {
		padding: 0;
		margin: 0;
		list-style-type: none;
	}

	a {
		display: inline-block;
	}

	.nav-links {
		display: none;
		border-bottom: 1px solid $borderColor;
		padding: 0.5rem 0 0.75rem 0;

		a {
			font-weight: 600;
		}

		.nav-item, .repo-link {
			display: block;
			line-height: 1.25rem;
			font-size: 1.1em;
			padding: 0.5rem 0 0.5rem 1.5rem;
		}
	}

	.sidebar-links {
		padding: 1.5rem 0;
	}
}

.yuu-theme-dark {
	.sidebar {
		background-color: $darkPrimaryBg;
		border-right-color: $darkBorderColor;

		.nav-links {
			border-bottom-color: $darkBorderColor;
		}
	}
}

@media (max-width: $MQMobile) {
	.sidebar {
		.nav-links {
			display: block;

			.dropdown-wrapper .nav-dropdown .dropdown-item a.router-link-active::after {
				top: calc(1rem - 2px);
			}
		}

		.sidebar-links {
			padding: 1rem 0;
		}
	}
}
</style>
