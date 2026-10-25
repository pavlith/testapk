<script lang="ts">
  import type { Snippet } from "svelte";
  import type { HTMLAttributes } from "svelte/elements";
  import { cn } from "$lib/utils";

  type Props = HTMLAttributes<HTMLSpanElement> & {
    variant?: "default" | "secondary" | "outline" | "success";
    children?: Snippet;
  };
  let { variant = "secondary", class: className = "", children, ...rest }: Props = $props();
  const styles = {
    default: "bg-primary text-primary-foreground",
    secondary: "bg-secondary text-secondary-foreground",
    outline: "border border-border bg-transparent text-foreground",
    success: "bg-emerald-100 text-emerald-700 dark:bg-emerald-950 dark:text-emerald-300"
  };
</script>

<span data-slot="badge" class={cn("inline-flex items-center rounded-full px-2.5 py-1 text-xs font-medium", styles[variant], className)} {...rest}>
  {@render children?.()}
</span>
