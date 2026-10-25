<script lang="ts">
  import { onMount } from "svelte";
  import { Capacitor } from "@capacitor/core";
  import { Device } from "@capacitor/device";
  import { Haptics, ImpactStyle } from "@capacitor/haptics";
  import {
    Check, CheckCheck, ClipboardList, Cpu, ListTodo, Moon, Plus, Search,
    Settings2, ShieldCheck, Smartphone, Sparkles, Sun, Trash2, X
  } from "@lucide/svelte";
  import { Badge } from "$lib/components/ui/badge";
  import { Button } from "$lib/components/ui/button";
  import { Card } from "$lib/components/ui/card";
  import { Input } from "$lib/components/ui/input";

  type Task = { id: string; title: string; done: boolean; createdAt: number };
  type Filter = "all" | "active" | "done";
  type Screen = "tasks" | "settings";

  const taskKey = "testapk.tasks.v1";
  const themeKey = "testapk.theme.v1";
  const examples: Task[] = [
    { id: "example-1", title: "Установить тестовый APK на телефон", done: true, createdAt: 1 },
    { id: "example-2", title: "Проверить работу интерфейса Svelte 5", done: false, createdAt: 2 },
    { id: "example-3", title: "Попробовать тёмную тему и вибрацию", done: false, createdAt: 3 }
  ];

  let tasks = $state<Task[]>([]);
  let draft = $state("");
  let search = $state("");
  let filter = $state<Filter>("all");
  let screen = $state<Screen>("tasks");
  let darkMode = $state(false);
  let deviceName = $state("Определяем устройство...");
  let androidVersion = $state("—");
  let platform = $state("web");
  let notice = $state("");

  const activeCount = $derived(tasks.filter((task) => !task.done).length);
  const doneCount = $derived(tasks.length - activeCount);
  const visibleTasks = $derived(
    tasks.filter((task) => {
      const matchesSearch = task.title.toLocaleLowerCase().includes(search.trim().toLocaleLowerCase());
      return matchesSearch && (filter === "all" || (filter === "active" && !task.done) || (filter === "done" && task.done));
    })
  );

  onMount(() => {
    try {
      const saved = localStorage.getItem(taskKey);
      const parsed: unknown = saved ? JSON.parse(saved) : null;
      if (Array.isArray(parsed)) {
        tasks = parsed.filter((task): task is Task =>
          task !== null && typeof task === "object" &&
          typeof task.id === "string" && typeof task.title === "string" && typeof task.done === "boolean" &&
          typeof task.createdAt === "number"
        );
      } else {
        tasks = [...examples];
        save();
      }
      darkMode = localStorage.getItem(themeKey) === "dark";
    } catch {
      tasks = [...examples];
    }
    document.documentElement.classList.toggle("dark", darkMode);
    platform = Capacitor.getPlatform();
    void Device.getInfo().then((info) => {
      deviceName = info.model || (info.platform === "web" ? "Веб-браузер" : info.platform);
      androidVersion = info.osVersion || "—";
    }).catch(() => { deviceName = "Устройство не определено"; });
  });

  function save() {
    try { localStorage.setItem(taskKey, JSON.stringify(tasks)); } catch { notice = "Не удалось сохранить данные"; }
  }

  function haptic() {
    if (Capacitor.isNativePlatform()) {
      void Haptics.impact({ style: ImpactStyle.Light }).catch(() => {});
    }
  }

  function addTask() {
    const title = draft.trim();
    if (!title) return;
    tasks = [{ id: crypto.randomUUID(), title, done: false, createdAt: Date.now() }, ...tasks];
    draft = "";
    save();
    haptic();
    notice = "Задача добавлена";
  }

  function toggleTask(id: string) {
    tasks = tasks.map((task) => task.id === id ? { ...task, done: !task.done } : task);
    save();
    haptic();
  }

  function deleteTask(id: string) {
    tasks = tasks.filter((task) => task.id !== id);
    save();
    haptic();
    notice = "Задача удалена";
  }

  function clearDone() {
    tasks = tasks.filter((task) => !task.done);
    save();
    haptic();
    notice = "Выполненные задачи удалены";
  }

  function resetDemo() {
    tasks = [...examples];
    draft = "";
    search = "";
    filter = "all";
    save();
    haptic();
    notice = "Демо-данные восстановлены";
  }

  function setTheme(value: boolean) {
    darkMode = value;
    document.documentElement.classList.toggle("dark", value);
    try { localStorage.setItem(themeKey, value ? "dark" : "light"); } catch { /* optional */ }
    haptic();
  }

  function navigate(next: Screen) {
    screen = next;
    notice = "";
    haptic();
  }
</script>

<svelte:head>
  <title>TestAPK — Svelte 5 + Android</title>
</svelte:head>

<div class="min-h-dvh bg-background text-foreground">
  <div class="mx-auto flex min-h-dvh w-full max-w-3xl flex-col">
    <header class="safe-top sticky top-0 z-10 border-b border-border/70 bg-background/90 px-5 pb-3 backdrop-blur-xl sm:px-8">
      <div class="flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="flex size-10 items-center justify-center rounded-xl bg-primary text-primary-foreground shadow-sm"><Sparkles class="size-5" /></div>
          <div>
            <div class="text-base font-bold tracking-tight">TestAPK</div>
            <div class="text-xs text-muted-foreground">Svelte 5 · shadcn-svelte</div>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <Badge variant="success" class="gap-1.5"><span class="size-1.5 rounded-full bg-emerald-500"></span>{platform === "android" ? "Android" : "Web"}</Badge>
          <Button variant="ghost" size="icon" aria-label="Переключить тему" onclick={() => setTheme(!darkMode)}>
            {#if darkMode}<Sun class="size-5" />{:else}<Moon class="size-5" />{/if}
          </Button>
        </div>
      </div>
    </header>

    <main class="flex-1 space-y-6 px-5 pb-28 pt-7 sm:px-8">
      {#if screen === "tasks"}
        <section class="space-y-2">
          <Badge variant="outline" class="gap-1.5"><Sparkles class="size-3" /> Demo application</Badge>
          <h1 class="text-3xl font-bold tracking-tight sm:text-4xl">Мои задачи <span class="text-primary">.</span></h1>
          <p class="max-w-lg text-sm leading-6 text-muted-foreground">Небольшой тест Android-приложения. Всё работает локально, без сервера и регистрации.</p>
        </section>

        <div class="grid grid-cols-2 gap-3">
          <Card class="p-4 sm:p-5">
            <div class="mb-3 flex size-9 items-center justify-center rounded-xl bg-indigo-100 text-indigo-600 dark:bg-indigo-950 dark:text-indigo-300"><ListTodo class="size-4.5" /></div>
            <div class="text-3xl font-bold tabular-nums">{activeCount}</div>
            <div class="mt-1 text-xs text-muted-foreground">Осталось выполнить</div>
          </Card>
          <Card class="p-4 sm:p-5">
            <div class="mb-3 flex size-9 items-center justify-center rounded-xl bg-emerald-100 text-emerald-600 dark:bg-emerald-950 dark:text-emerald-300"><CheckCheck class="size-4.5" /></div>
            <div class="text-3xl font-bold tabular-nums">{doneCount}</div>
            <div class="mt-1 text-xs text-muted-foreground">Уже готово</div>
          </Card>
        </div>

        <Card class="p-4 sm:p-5">
          <form class="flex gap-2" onsubmit={(event) => { event.preventDefault(); addTask(); }}>
            <Input bind:value={draft} maxlength={120} aria-label="Новая задача" placeholder="Что нужно сделать?" class="flex-1" />
            <Button size="icon" type="submit" aria-label="Добавить задачу" disabled={!draft.trim()}><Plus class="size-5" /></Button>
          </form>
          <p class="mt-2 text-xs text-muted-foreground">Нажми +, чтобы добавить новую задачу</p>
        </Card>

        <section class="space-y-4" aria-label="Список задач">
          <div class="flex items-center justify-between gap-3">
            <h2 class="text-lg font-semibold">Список задач</h2>
            <Badge variant="secondary">{tasks.length} всего</Badge>
          </div>
          <div class="relative">
            <Search class="pointer-events-none absolute top-3.5 left-3.5 size-4 text-muted-foreground" />
            <Input bind:value={search} placeholder="Поиск задач" aria-label="Поиск задач" class="pl-10 pr-10" />
            {#if search}
              <button class="absolute top-2.5 right-2.5 rounded-lg p-1 text-muted-foreground" aria-label="Очистить поиск" onclick={() => search = ""}><X class="size-4" /></button>
            {/if}
          </div>
          <div class="flex gap-2" aria-label="Фильтр задач">
            {#each [{ id: "all", label: "Все" }, { id: "active", label: "Активные" }, { id: "done", label: "Готово" }] as item}
              <Button variant={filter === item.id ? "default" : "outline"} size="sm" onclick={() => filter = item.id as Filter}>{item.label}</Button>
            {/each}
          </div>

          {#if visibleTasks.length > 0}
            <div class="space-y-2">
              {#each visibleTasks as task (task.id)}
                <Card class="flex items-center gap-3 p-3.5 transition-colors sm:p-4">
                  <button aria-label={task.done ? "Вернуть задачу" : "Выполнить задачу"} aria-pressed={task.done} class:completed={task.done} class="flex size-9 shrink-0 items-center justify-center rounded-xl border border-input transition-colors hover:border-primary completed:border-primary completed:bg-primary completed:text-primary-foreground" onclick={() => toggleTask(task.id)}>
                    {#if task.done}<Check class="size-5" />{/if}
                  </button>
                  <span class:line-through={task.done} class:text-muted-foreground={task.done} class="min-w-0 flex-1 break-words text-sm font-medium">{task.title}</span>
                  <Button variant="ghost" size="icon" class="shrink-0 text-muted-foreground hover:text-destructive" aria-label="Удалить задачу" onclick={() => deleteTask(task.id)}><Trash2 class="size-4" /></Button>
                </Card>
              {/each}
            </div>
          {:else}
            <Card class="flex flex-col items-center justify-center gap-3 px-6 py-12 text-center">
              <ClipboardList class="size-9 text-muted-foreground" />
              <div class="text-sm font-semibold">Задач не найдено</div>
              <p class="text-xs text-muted-foreground">Измени поиск или фильтр, либо добавь новую задачу.</p>
            </Card>
          {/if}
          {#if doneCount > 0}
            <div class="flex justify-end"><Button variant="ghost" size="sm" class="text-muted-foreground" onclick={clearDone}>Очистить выполненные</Button></div>
          {/if}
        </section>
      {:else}
        <section class="space-y-2">
          <Badge variant="outline" class="gap-1.5"><Settings2 class="size-3" /> Настройки</Badge>
          <h1 class="text-3xl font-bold tracking-tight">О приложении <span class="text-primary">.</span></h1>
          <p class="text-sm leading-6 text-muted-foreground">Нативная оболочка Capacitor и интерфейс Svelte 5.</p>
        </section>
        <Card class="space-y-4 p-5">
          <div class="flex items-center gap-3">
            <div class="flex size-10 items-center justify-center rounded-xl bg-secondary"><Smartphone class="size-5" /></div>
            <div><h2 class="font-semibold">Устройство</h2><p class="text-xs text-muted-foreground">Информация через Capacitor Device</p></div>
          </div>
          <div class="space-y-3 rounded-xl bg-muted/70 p-4 text-sm">
            <div class="flex justify-between gap-3"><span class="text-muted-foreground">Платформа</span><strong class="capitalize">{platform}</strong></div>
            <div class="flex justify-between gap-3"><span class="text-muted-foreground">Модель</span><strong class="text-right">{deviceName}</strong></div>
            <div class="flex justify-between gap-3"><span class="text-muted-foreground">Версия ОС</span><strong>{androidVersion}</strong></div>
          </div>
        </Card>
        <Card class="space-y-4 p-5">
          <div class="flex items-center gap-3">
            <div class="flex size-10 items-center justify-center rounded-xl bg-secondary"><Moon class="size-5" /></div>
            <div class="flex-1"><h2 class="font-semibold">Оформление</h2><p class="text-xs text-muted-foreground">Тема сохраняется на устройстве</p></div>
          </div>
          <Button variant="outline" class="w-full justify-start" onclick={() => setTheme(!darkMode)}>{#if darkMode}<Sun /> Светлая тема{:else}<Moon /> Тёмная тема{/if}</Button>
        </Card>
        <Card class="space-y-4 p-5">
          <div class="flex items-center gap-3"><div class="flex size-10 items-center justify-center rounded-xl bg-secondary"><Cpu class="size-5" /></div><div><h2 class="font-semibold">Нативные функции</h2><p class="text-xs text-muted-foreground">Нажми, чтобы проверить вибрацию</p></div></div>
          <Button class="w-full" onclick={() => { haptic(); notice = Capacitor.isNativePlatform() ? "Вибрация отправлена" : "Вибрация доступна на Android"; }}>Проверить Haptics</Button>
        </Card>
        <Card class="space-y-3 p-5">
          <div class="flex items-center gap-2"><ShieldCheck class="size-5 text-primary" /><h2 class="font-semibold">Данные</h2></div>
          <p class="text-sm leading-6 text-muted-foreground">Задачи сохраняются локально в WebView. При удалении приложения они могут быть потеряны.</p>
          <Button variant="outline" class="w-full" onclick={resetDemo}>Восстановить демо-задачи</Button>
        </Card>
        <p class="text-center text-xs text-muted-foreground">TestAPK v0.1.0 · dev.pavlith.testapk</p>
      {/if}
      {#if notice}<div class="rounded-xl bg-secondary px-4 py-3 text-center text-sm text-secondary-foreground" role="status">{notice}</div>{/if}
    </main>

    <nav class="safe-bottom fixed inset-x-0 bottom-0 z-20 border-t border-border bg-card/95 px-5 pt-2 backdrop-blur-xl" aria-label="Навигация">
      <div class="mx-auto flex max-w-3xl items-center justify-around">
        <button class:active={screen === "tasks"} class="flex min-w-28 flex-col items-center gap-1 rounded-xl px-5 py-2 text-xs font-medium text-muted-foreground transition-colors active:text-primary" onclick={() => navigate("tasks")} aria-current={screen === "tasks" ? "page" : undefined}>
          <ListTodo class="size-5" /> Задачи
        </button>
        <button class:active={screen === "settings"} class="flex min-w-28 flex-col items-center gap-1 rounded-xl px-5 py-2 text-xs font-medium text-muted-foreground transition-colors active:text-primary" onclick={() => navigate("settings")} aria-current={screen === "settings" ? "page" : undefined}>
          <Settings2 class="size-5" /> Настройки
        </button>
      </div>
    </nav>
  </div>
</div>
