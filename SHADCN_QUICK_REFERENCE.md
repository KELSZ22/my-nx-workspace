# shadcn/ui Quick Reference

## 🚀 Quick Start Commands

### Add a New Component

```bash
cd shared-ui
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add dialog
# etc...
```

### Export New Component

Add to `shared-ui/src/index.ts`:

```ts
export * from './components/ui/button';
```

### Use in Any App

```tsx
import { Button } from '@ts-demo/shared-ui';

function MyComponent() {
  return <Button>Click me</Button>;
}
```

## 📁 File Checklist for New Apps

When adding a new app that needs shadcn components:

### ✅ CSS File (styles.css or global.css)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --primary: 0 0% 9%;
    --primary-foreground: 0 0% 98%;
    /* ... all other variables ... */
  }
  .dark {
    /* dark mode variables */
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### ✅ Tailwind Config

```js
module.exports = {
  darkMode: ['class'],
  content: [
    './src/**/*.{ts,tsx,js,jsx}',
    '../../shared-ui/src/**/*.{ts,tsx,js,jsx}', // Include shared-ui!
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        // ... all color mappings
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [],
};
```

### ✅ Import Styles

**Next.js:** Already imported in `layout.tsx`
**React/Vite:** Add to `main.tsx`:

```tsx
import './styles.css';
```

## 🎨 Available Button Variants

```tsx
<Button variant="default">Default</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="destructive">Destructive</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>

<Button size="default">Default Size</Button>
<Button size="sm">Small</Button>
<Button size="lg">Large</Button>
<Button size="icon">🔥</Button>
```

## 🌓 Dark Mode Implementation

### Option 1: Simple Toggle

```tsx
function ThemeToggle() {
  const [isDark, setIsDark] = React.useState(false);

  React.useEffect(() => {
    document.documentElement.classList.toggle('dark', isDark);
  }, [isDark]);

  return (
    <Button onClick={() => setIsDark(!isDark)}>{isDark ? '🌙' : '☀️'}</Button>
  );
}
```

### Option 2: With localStorage

```tsx
function ThemeToggle() {
  const [theme, setTheme] = React.useState(
    () => localStorage.getItem('theme') || 'light'
  );

  React.useEffect(() => {
    localStorage.setItem('theme', theme);
    document.documentElement.classList.toggle('dark', theme === 'dark');
  }, [theme]);

  return (
    <Button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </Button>
  );
}
```

## 🔧 Common Customizations

### Custom Colors

Update CSS variables in your app's CSS file:

```css
:root {
  --primary: 220 90% 56%; /* Blue instead of black */
  --destructive: 0 100% 50%; /* Bright red */
}
```

### Custom Border Radius

```css
:root {
  --radius: 1rem; /* More rounded */
}
```

### Component-Specific Override

```tsx
<Button className="bg-purple-500 hover:bg-purple-600">Custom Color</Button>
```

## 📦 Popular Component Combinations

### Card with Button

```tsx
import {
  Card,
  CardHeader,
  CardTitle,
  CardContent,
  Button,
} from '@ts-demo/shared-ui';

<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>
    <p>Content here</p>
    <Button className="mt-4">Action</Button>
  </CardContent>
</Card>;
```

### Dialog with Form

```tsx
import {
  Dialog,
  DialogContent,
  DialogHeader,
  Button,
} from '@ts-demo/shared-ui';

<Dialog>
  <DialogTrigger asChild>
    <Button>Open</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Title</DialogTitle>
    </DialogHeader>
    {/* Form content */}
  </DialogContent>
</Dialog>;
```

## 🐛 Troubleshooting Checklist

### Component has no styling?

- [ ] CSS variables added to app's CSS file
- [ ] Tailwind config has color mappings
- [ ] `shared-ui/src/**/*.{ts,tsx}` in Tailwind content array
- [ ] Styles imported in app entry point (main.tsx/layout.tsx)

### Import error "@/lib/utils"?

- [ ] Path alias in `shared-ui/tsconfig.json`
- [ ] Path alias in `shared-ui/tsconfig.lib.json`
- [ ] Both have `"baseUrl": "."` and `"paths": {"@/*": ["./src/*"]}`

### TypeScript errors?

- [ ] Component exported from `shared-ui/src/index.ts`
- [ ] Importing from `@ts-demo/shared-ui` (not relative path)
- [ ] Run `nx reset` to clear cache

### Tailwind classes not working?

- [ ] Content paths include shared-ui components
- [ ] Restart dev server after Tailwind config changes
- [ ] Check for conflicting global styles

## 📚 Resources

- [shadcn/ui Documentation](https://ui.shadcn.com)
- [Tailwind CSS Documentation](https://tailwindcss.com)
- [Nx Documentation](https://nx.dev)
- [Radix UI Primitives](https://www.radix-ui.com)

## 💡 Pro Tips

1. **Component Preview**: Create a `shared-ui/src/preview.tsx` to preview all components
2. **Storybook**: Add Storybook to document components visually
3. **Color Generator**: Use [ui.shadcn.com/themes](https://ui.shadcn.com/themes) for color palettes
4. **VSCode Extension**: Install "Tailwind CSS IntelliSense" for autocomplete
5. **Pre-commit Hook**: Use Prettier to format Tailwind classes consistently

---

**Quick Command Reference:**

```bash
# Add component
cd shared-ui && npx shadcn@latest add <component>

# Run Next.js app
nx serve my-next-app

# Run React app
nx serve my-react-app

# Build shared-ui
nx build shared-ui

# Run all apps
nx run-many --target=serve --all
```
