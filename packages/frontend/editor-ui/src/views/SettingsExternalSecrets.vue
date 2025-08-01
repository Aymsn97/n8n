<script lang="ts" setup>
import { useI18n } from '@n8n/i18n';
import { useToast } from '@/composables/useToast';
import { useDocumentTitle } from '@/composables/useDocumentTitle';
import { useExternalSecretsStore } from '@/stores/externalSecrets.ee.store';
import { computed, onMounted } from 'vue';
import ExternalSecretsProviderCard from '@/components/ExternalSecretsProviderCard.ee.vue';
import type { ExternalSecretsProvider } from '@/Interface';
import { usePageRedirectionHelper } from '@/composables/usePageRedirectionHelper';
import { I18nT } from 'vue-i18n';

const i18n = useI18n();
const externalSecretsStore = useExternalSecretsStore();
const toast = useToast();
const documentTitle = useDocumentTitle();
const pageRedirectionHelper = usePageRedirectionHelper();

const sortedProviders = computed(() => {
	return ([...externalSecretsStore.providers] as ExternalSecretsProvider[]).sort((a, b) => {
		return b.name.localeCompare(a.name);
	});
});

onMounted(() => {
	documentTitle.set(i18n.baseText('settings.externalSecrets.title'));
	if (!externalSecretsStore.isEnterpriseExternalSecretsEnabled) return;
	try {
		void externalSecretsStore.fetchAllSecrets();
		void externalSecretsStore.getProviders();
	} catch (error) {
		toast.showError(error, i18n.baseText('error'));
	}
});

function goToUpgrade() {
	void pageRedirectionHelper.goToUpgrade('external-secrets', 'upgrade-external-secrets');
}
</script>

<template>
	<div class="pb-3xl">
		<n8n-heading size="2xlarge">{{ i18n.baseText('settings.externalSecrets.title') }}</n8n-heading>
		<div
			v-if="externalSecretsStore.isEnterpriseExternalSecretsEnabled"
			data-test-id="external-secrets-content-licensed"
		>
			<n8n-callout theme="secondary" class="mt-2xl mb-l">
				{{ i18n.baseText('settings.externalSecrets.info') }}
				<a :href="i18n.baseText('settings.externalSecrets.docs')" target="_blank">
					{{ i18n.baseText('settings.externalSecrets.info.link') }}
				</a>
			</n8n-callout>
			<ExternalSecretsProviderCard
				v-for="provider in sortedProviders"
				:key="provider.name"
				:provider="provider"
			/>
		</div>
		<n8n-action-box
			v-else
			class="mt-2xl mb-l"
			data-test-id="external-secrets-content-unlicensed"
			:button-text="i18n.baseText('settings.externalSecrets.actionBox.buttonText')"
			@click="goToUpgrade"
		>
			<template #heading>
				<span>{{ i18n.baseText('settings.externalSecrets.actionBox.title') }}</span>
			</template>
			<template #description>
				<I18nT keypath="settings.externalSecrets.actionBox.description" scope="global">
					<template #link>
						<a :href="i18n.baseText('settings.externalSecrets.docs')" target="_blank">
							{{ i18n.baseText('settings.externalSecrets.actionBox.description.link') }}
						</a>
					</template>
				</I18nT>
			</template>
		</n8n-action-box>
	</div>
</template>
