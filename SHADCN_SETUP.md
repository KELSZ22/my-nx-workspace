# shadcn/ui Setup in Nx Monorepo

This guide documents how to set up shadcn/ui components in an Nx monorepo with a shared UI library that can be used across multiple applications (Next.js, React, etc.).

## Architecture Overview

```
my-nx-workspace/
├── shared-ui/                    # Shared UI library with shadcn components
│   ├── src/
│   │   ├── components/ui/        # shadcn/ui components
│   │   │   └── button.tsx
│   │   ├── lib/
│   │   │   └── utils.ts          # cn() utility function
│   │   └── index.ts              # Main export file
│   ├── components.json           # shadcn configuration
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   └── package.json
├── apps/
│   ├── my-next-app/              # Next.js app
│   └── my-react-app/             # React app
└── package.json
```

## Step 1: Create Shared UI Library

### 1.1 Initialize the Library

```bash
# Create a new library (if not already created)
nx g @nx/react:library shared-ui --directory=shared-ui --bundler=vite
```

### 1.2 Install Dependencies

In `shared-ui/package.json`:

```json
{
  "name": "@ts-demo/shared-ui",
  "version": "0.0.1",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": {
      "types": "./src/index.ts",
      "import": "./src/index.ts",
      "default": "./src/index.ts"
    },
    "./package.json": "./package.json"
  },
  "devDependencies": {
    "@types/react": "^19.2.2",
    "@types/react-dom": "^19.2.2",
    "autoprefixer": "^10.4.21",
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "postcss": "^8.5.6",
    "tailwindcss": "^4.1.14"
  },
  "dependencies": {
    "@radix-ui/react-slot": "^1.2.3",
    "lucide-react": "^0.545.0",
    "react": "^19.2.0",
    "react-dom": "^19.2.0",
    "tailwind-merge": "^3.3.1"
  }
}
```

### 1.3 Configure Tailwind CSS

Create `shared-ui/tailwind.config.js`:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{ts,tsx,js,jsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Create `shared-ui/postcss.config.js`:

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

Create `shared-ui/global.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 1.4 Initialize shadcn/ui

Create `shared-ui/components.json`:

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "global.css",
    "baseColor": "slate",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

### 1.5 Configure TypeScript Path Aliases

Update `shared-ui/tsconfig.json`:

```json
{
  "files": [],
  "include": [],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "references": [
    {
      "path": "./tsconfig.lib.json"
    }
  ],
  "extends": "../tsconfig.base.json"
}
```

Update `shared-ui/tsconfig.lib.json`:

```json
{
  "extends": "../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist",
    "types": [
      "node",
      "@nx/react/typings/cssmodule.d.ts",
      "@nx/react/typings/image.d.ts"
    ],
    "rootDir": "src",
    "jsx": "react-jsx",
    "module": "esnext",
    "moduleResolution": "bundler",
    "tsBuildInfoFile": "dist/tsconfig.lib.tsbuildinfo",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "exclude": [
    "out-tsc",
    "dist",
    "jest.config.ts",
    "src/**/*.spec.ts",
    "src/**/*.test.ts",
    "src/**/*.spec.tsx",
    "src/**/*.test.tsx",
    "src/**/*.spec.js",
    "src/**/*.test.js",
    "src/**/*.spec.jsx",
    "src/**/*.test.jsx",
    "eslint.config.js",
    "eslint.config.cjs",
    "eslint.config.mjs"
  ],
  "include": ["src/**/*.js", "src/**/*.jsx", "src/**/*.ts", "src/**/*.tsx"]
}
```

### 1.6 Create Utility Functions

Create `shared-ui/src/lib/utils.ts`:

```ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### 1.7 Add shadcn Components

Create `shared-ui/src/components/ui/button.tsx`:

```tsx
import * as React from 'react';
import { Slot } from '@radix-ui/react-slot';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '../../lib/utils';

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-all disabled:pointer-events-none disabled:opacity-50 [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4 shrink-0 [&_svg]:shrink-0 outline-none focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px] aria-invalid:ring-destructive/20 dark:aria-invalid:ring-destructive/40 aria-invalid:border-destructive",
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive:
          'bg-destructive text-white hover:bg-destructive/90 focus-visible:ring-destructive/20 dark:focus-visible:ring-destructive/40 dark:bg-destructive/60',
        outline:
          'border bg-background shadow-xs hover:bg-accent hover:text-accent-foreground dark:bg-input/30 dark:border-input dark:hover:bg-input/50',
        secondary:
          'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost:
          'hover:bg-accent hover:text-accent-foreground dark:hover:bg-accent/50',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-9 px-4 py-2 has-[>svg]:px-3',
        sm: 'h-8 rounded-md gap-1.5 px-3 has-[>svg]:px-2.5',
        lg: 'h-10 rounded-md px-6 has-[>svg]:px-4',
        icon: 'size-9',
        'icon-sm': 'size-8',
        'icon-lg': 'size-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

function Button({
  className,
  variant,
  size,
  asChild = false,
  ...props
}: React.ComponentProps<'button'> &
  VariantProps<typeof buttonVariants> & {
    asChild?: boolean;
  }) {
  const Comp = asChild ? Slot : 'button';

  return (
    <Comp
      data-slot="button"
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  );
}

export { Button, buttonVariants };
```

### 1.8 Export Components

Update `shared-ui/src/index.ts`:

```ts
export * from './lib/shared-ui';
export * from './components/ui/button';
```

## Step 2: Setup Next.js App

### 2.1 Add CSS Variables

Update `apps/my-next-app/src/app/global.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 0 0% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 0 0% 3.9%;
    --primary: 0 0% 9%;
    --primary-foreground: 0 0% 98%;
    --secondary: 0 0% 96.1%;
    --secondary-foreground: 0 0% 9%;
    --muted: 0 0% 96.1%;
    --muted-foreground: 0 0% 45.1%;
    --accent: 0 0% 96.1%;
    --accent-foreground: 0 0% 9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 89.8%;
    --input: 0 0% 89.8%;
    --ring: 0 0% 3.9%;
    --chart-1: 12 76% 61%;
    --chart-2: 173 58% 39%;
    --chart-3: 197 37% 24%;
    --chart-4: 43 74% 66%;
    --chart-5: 27 87% 67%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 0 0% 3.9%;
    --foreground: 0 0% 98%;
    --card: 0 0% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 0 0% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 0% 98%;
    --primary-foreground: 0 0% 9%;
    --secondary: 0 0% 14.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 0 0% 14.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 0 0% 14.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 14.9%;
    --input: 0 0% 14.9%;
    --ring: 0 0% 83.1%;
    --chart-1: 220 70% 50%;
    --chart-2: 160 60% 45%;
    --chart-3: 30 80% 55%;
    --chart-4: 280 65% 60%;
    --chart-5: 340 75% 55%;
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

### 2.2 Update Tailwind Config

Update `apps/my-next-app/tailwind.config.js`:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ['class'],
  content: [
    './{src,pages,components,app}/**/*.{ts,tsx,js,jsx,html}',
    '!./{src,pages,components,app}/**/*.{stories,spec}.{ts,tsx,js,jsx,html}',
    '../../shared-ui/src/**/*.{ts,tsx,js,jsx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
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

### 2.3 Use Components

In `apps/my-next-app/src/app/page.tsx`:

```tsx
import { Button } from '@ts-demo/shared-ui';

export default function Page() {
  return (
    <div className="p-4">
      <Button variant="default">Click me</Button>
    </div>
  );
}
```

## Step 3: Setup React App

### 3.1 Add CSS Variables

Update `apps/my-react-app/src/styles.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 0 0% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 0 0% 3.9%;
    --primary: 0 0% 9%;
    --primary-foreground: 0 0% 98%;
    --secondary: 0 0% 96.1%;
    --secondary-foreground: 0 0% 9%;
    --muted: 0 0% 96.1%;
    --muted-foreground: 0 0% 45.1%;
    --accent: 0 0% 96.1%;
    --accent-foreground: 0 0% 9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 89.8%;
    --input: 0 0% 89.8%;
    --ring: 0 0% 3.9%;
    --chart-1: 12 76% 61%;
    --chart-2: 173 58% 39%;
    --chart-3: 197 37% 24%;
    --chart-4: 43 74% 66%;
    --chart-5: 27 87% 67%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 0 0% 3.9%;
    --foreground: 0 0% 98%;
    --card: 0 0% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 0 0% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 0% 98%;
    --primary-foreground: 0 0% 9%;
    --secondary: 0 0% 14.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 0 0% 14.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 0 0% 14.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 14.9%;
    --input: 0 0% 14.9%;
    --ring: 0 0% 83.1%;
    --chart-1: 220 70% 50%;
    --chart-2: 160 60% 45%;
    --chart-3: 30 80% 55%;
    --chart-4: 280 65% 60%;
    --chart-5: 340 75% 55%;
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

### 3.2 Update Tailwind Config

Update `apps/my-react-app/tailwind.config.js`:

```js
const { createGlobPatternsForDependencies } = require('@nx/react/tailwind');
const { join } = require('path');

/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ['class'],
  content: [
    join(
      __dirname,
      '{src,pages,components,app}/**/*!(*.stories|*.spec).{ts,tsx,html}'
    ),
    ...createGlobPatternsForDependencies(__dirname),
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
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

### 3.3 Import Styles

Update `apps/my-react-app/src/main.tsx`:

```tsx
import { StrictMode } from 'react';
import { BrowserRouter } from 'react-router-dom';
import * as ReactDOM from 'react-dom/client';
import App from './app/app';
import './styles.css'; // Add this line

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);
```

### 3.4 Use Components

In `apps/my-react-app/src/app/app.tsx`:

```tsx
import { Button } from '@ts-demo/shared-ui';

export function App() {
  return (
    <div className="p-4">
      <Button variant="default">Click me</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
    </div>
  );
}
```

## Adding More shadcn Components

To add more shadcn components:

1. Navigate to the shared-ui directory:

   ```bash
   cd shared-ui
   ```

2. Install the component using shadcn CLI:

   ```bash
   npx shadcn@latest add <component-name>
   ```

3. Export the new component in `shared-ui/src/index.ts`:

   ```ts
   export * from './components/ui/<component-name>';
   ```

4. Use it in any app:
   ```tsx
   import { ComponentName } from '@ts-demo/shared-ui';
   ```

## Troubleshooting

### Issue: "Cannot find module '@/lib/utils'"

**Solution:** Ensure path aliases are configured in both `tsconfig.json` and `tsconfig.lib.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Issue: Button has no styling

**Solution:** Make sure:

1. CSS variables are added to your app's CSS file
2. Tailwind config includes the color mappings
3. The shared-ui path is included in Tailwind's `content` array
4. Styles are imported in your app's entry point

### Issue: Dark mode not working

**Solution:** Add dark mode toggle to your app:

```tsx
export function ThemeToggle() {
  const [theme, setTheme] = React.useState('light');

  React.useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
  }, [theme]);

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle theme
    </button>
  );
}
```

## Key Points

1. **Shared Library**: All shadcn components live in the `shared-ui` library
2. **Path Aliases**: Use `@/*` in the shared library, regular imports in apps
3. **CSS Variables**: Each consuming app needs the CSS variables
4. **Tailwind Config**: Each app needs the color mappings and shared-ui in content paths
5. **Export Pattern**: Always export new components from `shared-ui/src/index.ts`
6. **Import Pattern**: Import from `@ts-demo/shared-ui` in all apps

## Benefits of This Setup

✅ **Single Source of Truth**: All UI components in one place  
✅ **Consistency**: Same design system across all apps  
✅ **Maintainability**: Update once, use everywhere  
✅ **Type Safety**: Full TypeScript support  
✅ **Tree Shaking**: Only bundle components you use  
✅ **Dark Mode**: Built-in support across all apps

---

**Last Updated:** October 14, 2025
