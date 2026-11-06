# utils

## Variables

### pageWidthClass

```ts
const pageWidthClass: "mx-auto w-full max-w-6xl lg:max-w-7xl xl:max-w-[85rem] 2xl:max-w-[96rem]" = "mx-auto w-full max-w-6xl lg:max-w-7xl xl:max-w-[85rem] 2xl:max-w-[96rem]";
```

Shared content column for SPA headers and mains.

Grows past `max-w-6xl` on large / ultrawide viewports so shelves and tables
are not trapped in a narrow center gutter.

***

### titleDetailDialogClass

```ts
const titleDetailDialogClass: "w-full max-w-3xl lg:max-w-4xl xl:max-w-5xl" = "w-full max-w-3xl lg:max-w-4xl xl:max-w-5xl";
```

Title detail dialog width classes — wider than form modals for commerce + reviews.

## Functions

### cn()

```ts
function cn(...inputs): string;
```

Merges Tailwind class names with conflict resolution via `tailwind-merge`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| ...`inputs` | `ClassValue`[] | Class values accepted by `clsx`. |

#### Returns

`string`

Merged class string.
