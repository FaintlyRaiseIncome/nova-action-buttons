<template>
    <panel-item :field="field">
        <template v-slot:value>
            <a href="#" :style="finalStyles" :class="finalClasses" :title="name" @click.stop.prevent="fireAction">
                <span v-if="text" v-text="text"/>
                <span v-if="icon && !iconIsUrl" v-html="icon"/>
                <img v-if="icon && iconIsUrl" :src="icon" class="w-6 h-6 inline"  />
            </a>

            <!-- Action Confirmation Modal -->
            <portal to="modals" transition="fade-transition">
                <component
                    v-if="confirmActionModalOpened"
                    v-bind="options"
                    class="text-left"
                    :is="selectedAction.component"
                    @close="closeConfirmationModal"
                    @confirm="executeAction">
                </component>
            </portal>
        </template>
    </panel-item>
</template>

<script setup>

    // Vue
    import {computed} from 'vue';

    // Props
    const props = defineProps({
        field: {type: Object, default: null},
        queryString: {type: Object, default: null},
        resourceName: {type: String, default: null},
    });

    // Composables
    import {useHandleAction} from '../mixins/HandlesActions'

    // Computed
    const text = computed(() => props?.field?.text || null);
    const icon = computed(() => props?.field?.icon || null);
    const iconIsUrl = computed(() => icon?.value?.startsWith('http') === true);
    const name = computed(() => props?.field?.name || null);
    const customStyles = computed(() => props?.field?.styles || []);
    const customClasses = computed(() => props?.field?.classes || []);
    const asToolbarButton = computed(() => props?.field?.asToolbarButton === true);

    const actionButtonClasses = computed(() => [
        'flex-shrink-0', 'shadow', 'rounded', 'focus:outline-none', 'ring-primary-200', 'dark:ring-gray-600',
        'focus:ring', 'bg-primary-500', 'hover:bg-primary-400', 'active:bg-primary-600',
        'text-white', 'dark:text-gray-800', 'inline-flex', 'items-center', 'font-bold', 'px-2', 'h-9', 'text-sm', 'flex-shrink-0',
    ])
    const toolbarButtonClasses = computed(() => [
        'toolbar-button', 'hover:text-primary-500', 'px-2', 'v-popper--has-tooltip', 'w-10'
    ])

    const finalStyles = computed(() => ({...(customStyles.value || {})}))
    const finalClasses = computed(() => [...(asToolbarButton?.value === true ? toolbarButtonClasses.value : actionButtonClasses.value), ...(customClasses.value || [])])

    const queryString = computed(() => ({
        action: selectedAction?.value?.uriKey,
        search: props?.queryString?.currentSearch,
        filters: props?.queryString?.encodedFilters,
        trashed: props?.queryString?.currentTrashed,
        viaResource: props?.queryString?.viaResource,
        viaResourceId: props?.queryString?.viaResourceId,
        viaRelationship: props?.queryString?.viaRelationship,
    }));

    const selectedAction = computed(() => props?.field?.action);

    const selectedResources = computed(() => [props?.field?.resourceId]);

    // Bindings
    const {
        errors,
        working,
        fireAction,
        executeAction,
        closeConfirmationModal,
        confirmActionModalOpened
    } = useHandleAction(
        {
            queryString: queryString.value,
            resourceName: props?.resourceName,
            selectedAction: selectedAction.value,
            selectedResources: selectedResources.value,
        }
    )

    // Computed
    const options = computed(() => ({
        show: true,
        errors: errors?.value,
        action: selectedAction?.value,
        working: working?.value === true,
        resourceName: props?.resourceName,
        selectedResources: selectedResources?.value,
    }))

</script>
