<template>
	<NcBreadcrumbs 
		data-cy-files-content-breadcrumbs
		:aria-label="t('files', 'Current directory path')">
		<!-- Current path sections -->
		<NcBreadcrumb v-for="(section, index) in sections"
			:key="section.dir"
			v-bind="section"
			dir="auto"
			:to="section.to"
			:title="titleForSection(section)"
			@click.native="onClick(section.to)">
			<template v-if="index === 0" #icon>
				<Home :size="20"/>
			</template>
		</NcBreadcrumb>

		<!-- Forward the actions slot -->
		<template #actions>
			<slot name="actions" />
		</template>
	</NcBreadcrumbs>
</template>

<script>
import { translate as t} from '@nextcloud/l10n'
import { basename } from 'path'
import Home from 'vue-material-design-icons/Home.vue'
import NcBreadcrumb from '@nextcloud/vue/dist/Components/NcBreadcrumb.js'
import NcBreadcrumbs from '@nextcloud/vue/dist/Components/NcBreadcrumbs.js'
import Vue from 'vue'

import { useFilesStore } from '../store/files.ts'
import { usePathsStore } from '../store/paths.ts'

export default Vue.extend({
	name: 'BreadCrumbs',

	components: {
		Home,
		NcBreadcrumbs,
		NcBreadcrumb,
	},

	props: {
		path: {
			type: String,
			default: '/',
		},
	},

	setup() {
		const filesStore = useFilesStore()
		const pathsStore = usePathsStore()
		return {
			filesStore,
			pathsStore,
		}
	},

	computed: {
		currentView() {
			return this.$navigation.active
		},

		dirs() {
			const cumulativePath = (acc) => (value) => (acc += `${value}/`)
			// Generate a cumulative path for each path segment: ['/', '/foo', '/foo/bar', ...] etc
			const paths = this.path.split('/').filter(Boolean).map(cumulativePath('/'))
			// Strip away trailing slash
			return ['/', ...paths.map(path => path.replace(/^(.+)\/$/, '$1'))]
		},

		sections() {
			return this.dirs.map(dir => {
				const fileid = this.getFileIdFromPath(dir)
				const to = { ...this.$route, params: { fileid }, query: { dir } }
				return {
					dir,
					exact: true,
					name: this.getDirDisplayName(dir),
					to,
				}
			})
		},
	},

	methods: {
		getNodeFromId(id) {
			return this.filesStore.getNode(id)
		},
		getFileIdFromPath(path) {
			return this.pathsStore.getPath(this.currentView?.id, path)
		},
		getDirDisplayName(path) {
			if (path === '/') {
				return t('files', 'Home')
			}

			const fileId = this.getFileIdFromPath(path)
			const node = this.getNodeFromId(fileId)
			return node?.attributes?.displayName || basename(path)
		},

		onClick(to) {
			if (to?.query?.dir === this.$route.query.dir) {
				this.$emit('reload')
			}
		},

		titleForSection(section) {
			if (section?.to?.query?.dir === this.$route.query.dir) {
				return t('files', 'Reload current directory')
			}
			return t('files', 'Go to the "{dir}" directory', section)
		},

		t,
	},
})
</script>

<style lang="scss" scoped>
.breadcrumb {
	// Take as much space as possible
	flex: 1 1 100% !important;
	width: 100%;

	::v-deep a {
		cursor: pointer !important;
	}
}

</style>
