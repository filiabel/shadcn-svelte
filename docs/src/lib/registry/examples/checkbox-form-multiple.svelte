<script lang="ts" module>
	import { z } from "zod";

	const items = [
		{
			id: "recents",
			label: "Recents",
		},
		{
			id: "home",
			label: "Home",
		},
		{
			id: "applications",
			label: "Applications",
		},
		{
			id: "desktop",
			label: "Desktop",
		},
		{
			id: "downloads",
			label: "Downloads",
		},
		{
			id: "documents",
			label: "Documents",
		},
	] as const;

	export const formSchema = z.object({
		items: z.array(z.string()).refine((value) => value.some((item) => item), {
			message: "You have to select at least one item.",
		}),
	});
	export type FormSchema = typeof formSchema;
</script>

<script lang="ts">
	import SuperDebug, { type Infer, type SuperValidated, superForm } from "sveltekit-superforms";
	import { zodClient } from "sveltekit-superforms/adapters";
	import { toast } from "svelte-sonner";
	import { browser } from "$app/environment";
	import { page } from "$app/state";
	import * as Form from "$lib/registry/ui/form/index.js";
	import { Checkbox } from "$lib/registry/ui/checkbox/index.js";

	let { form: data = page.data.checkboxMultiple }: { form: SuperValidated<Infer<FormSchema>> } =
		$props();

	const form = superForm(data, {
		validators: zodClient(formSchema),
		onUpdated: ({ form: f }) => {
			if (f.valid) {
				toast.success(`You submitted ${JSON.stringify(f.data, null, 2)}`);
			} else {
				toast.error("Please fix the errors in the form.");
			}
		},
	});

	const { form: formData, enhance } = form;

	function addItem(id: string) {
		$formData.items = [...$formData.items, id];
	}

	function removeItem(id: string) {
		$formData.items = $formData.items.filter((i) => i !== id);
	}
</script>

<form action="/?/checkboxMultiple" method="POST" class="space-y-8" use:enhance>
	<Form.Fieldset {form} name="items" class="space-y-0">
		<div class="mb-4">
			<Form.Legend class="text-base">Sidebar</Form.Legend>
			<Form.Description>
				Select the items you want to display in the sidebar.
			</Form.Description>
		</div>
		<div class="space-y-2">
			{#each items as item (item.id)}
				{@const checked = $formData.items.includes(item.id)}
				<div class="flex flex-row items-start space-x-3">
					<Form.Control>
						{#snippet children({ props })}
							<Checkbox
								{...props}
								{checked}
								value={item.id}
								onCheckedChange={(v) => {
									if (v) {
										addItem(item.id);
									} else {
										removeItem(item.id);
									}
								}}
							/>
							<Form.Label class="font-normal">
								{item.label}
							</Form.Label>
						{/snippet}
					</Form.Control>
				</div>
			{/each}
			<Form.FieldErrors />
		</div>
	</Form.Fieldset>
	<Form.Button>Update display</Form.Button>
	{#if browser}
		<SuperDebug data={$formData} />
	{/if}
</form>
