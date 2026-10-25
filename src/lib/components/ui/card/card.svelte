<script lang="ts">
  import type { Snippet } from "svelte";
  import type { HTMLAttributes } from "svelte/elements";
  import { cn } from "$lib/utils";

  type Props = HTMLAttributes<HTMLDivElement> & { children?: Snippet };
  let { class: className = "", children, ...rest }: Props = $props();
</script>

<div data-slot="card" class={cn("rounded-2xl border border-border bg-card text-card-foreground shadow-sm", className)} {...rest}>
  {@render children?.()}
</div>
