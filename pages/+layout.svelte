<script>
	import '@evidence-dev/tailwind/fonts.css';
	import '../app.css';
	import { EvidenceDefaultLayout } from '@evidence-dev/core-components';
	export let data;
</script>

<EvidenceDefaultLayout {data} hideSidebar hideBreadcrumbs githubRepo="https://github.com/archiewood/evidence-openai/">
	<slot slot="content" />
</EvidenceDefaultLayout>
