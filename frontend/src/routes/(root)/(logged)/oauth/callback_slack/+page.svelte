<script lang="ts">
	import { goto } from '$app/navigation'
	import { page } from '$app/stores'
	import { sendUserToast } from '$lib/utils'
	import { onMount } from 'svelte'
	import { OauthService } from '$lib/gen'
	import { workspaceStore, oauthStore } from '$lib/stores'
	import CenteredPage from '$lib/components/CenteredPage.svelte'
	import PageHeader from '$lib/components/PageHeader.svelte'
	import WindmillIcon from '$lib/components/icons/WindmillIcon.svelte'

	let error = $page.url.searchParams.get('error')
	let code = $page.url.searchParams.get('code') ?? undefined
	let state = $page.url.searchParams.get('state') ?? undefined

	onMount(async () => {
		if (error) {
			sendUserToast(`Error trying to add slack connection: ${error}`, true)
		} else if (code && state) {
			const token = await OauthService.connectSlackCallback({
				workspace: $workspaceStore!,
				requestBody: { code, state }
			})
			oauthStore.set({ access_token: token })
			sendUserToast(
				'Slack workspace connected to your Windmill workspace and slack token saved in the folder `slack_bot` at `f/slack_bot/bot_token`.'
			)
		} else {
			sendUserToast('Missing code or state as query params', true)
		}
		goto('/workspace_settings?tab=slack')
	})
</script>

<CenteredPage>
	<PageHeader title="Connection to slack in progress" />
	<div class="mx-auto w-0">
		<WindmillIcon height="80px" width="80px" spin="fast" />
	</div>
</CenteredPage>
