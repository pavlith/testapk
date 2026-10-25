<script lang="ts">
  import type { HTMLInputAttributes } from "svelte/elements";
  import { cn } from "$lib/utils";

  type Props = Omit<HTMLInputAttributes, "value"> & { value?: string };
  let { value = $bindable(""), class: className = "", type = "text", ...rest }: Props = $props();
</script>

<input data-slot="input" {type} bind:value class={cn("h-11 w-full min-w-0 rounded-xl border border-input bg-background px-3.5 text-sm shadow-xs outline-none placeholder:text-muted-foreground focus-visible:border-ring focus-visible:ring-2 focus-visible:ring-ring/30 disabled:opacity-50", className)} {...rest} />
