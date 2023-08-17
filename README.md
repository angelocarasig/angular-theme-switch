# Angular Theme Switching
This repo is just an approach to implementing a theme switcher. 
Maybe it's easier to just use an npm package, but this is more simple and can satisfy use-cases much easier.

## Use Cases
* You want to rely on css variables over $variable .scss notation
* Similarly, you don't want to use @import at the top of each .scss file
* Code easily changeable and configurable
* Similarly, you don't want to use an npm package
* You want to use the same variable names and let the code figure out which colour to use based on the current theme


----
*Assuming that you already have an existing Angular project, and you are using sass for the stylesheets*:

1. Create a themes folder in the same level as the app
```
   src/
   ├── app
   ├── assets
   ├── environments
   ├── ...
   ├── styles
   ├── ...
   ├── index.html
   ├── main.ts
   └── etc.
```

2. In your `angular.json`, add the following into `styles` (applies what's inside) and `stylePreprocessorOptions` (This makes them available without specifying a common path):

```json
"styles": [
    "src/styles/app.scss"
    "src/styles.scss",
],
"stylePreprocessorOptions": {
  "includePaths": [
    "src/styles/",
    "src"
  ]
}
```

3. Create a `app.scss` file inside the themes folder (You can name it whatever you want, just make sure the name is the same as what's included above)
4. Ideally you should group similar .scss files by folder inside the `styles` folder, thus go create a themes folder
5. Inside the themes folder create .scss files for each theme you want to implement:
```
styles/
├── app.scss
└── themes/
    ├── light.scss
    └── dark.scss
```

Assume we modify the `light.scss` file:
6. Create a map with variables you want to define. Make sure that the variable names stay consistent and generic, ideally don't include theme specifics like adding a --light at 
the end, and to also make sure that the map name is unique instead (*'light-theme'*). Essentially you should be defining this like a colour palette:

```scss
$light-theme: (
    primary-color: red,
    secondary-color: blue,
    ...
)
```

7. Repeat with all other themes you wish to implement.
8. Inside the `app.scss` folder, import the themes you are implementing and create a mixin for each theme. We are then looping through each variable and adding the '--' prefix so 
that components in the Angular app can access through `var(--somePropertyName)`. Make sure that for each mixin you use the defined unique map name.

```scss
@import 'light';
@import 'dark';

@mixin light() {
  @each $key, $value in $light-theme: {
    --#{$key}: #{$value};
  }
}

@mixin dark() {
  @each $key, $value in $dark-theme: {
    --#{$key}: #{$value};
  }
}

...

@mixin some-other-theme() {
  @each $key, $value in $some-other-theme: {
    --#{$key}: #{$value};
  }
}
```

The next part requires you to make sure variable names are similar both in the defined .scss and the Angular .ts file side.

9. In the `main.scss` file, do the following:
```scss
@import app.scss

body.light {
    @include light();
}

body.dark {
@include dark();
}

...

body.some-other-theme {
@include some-other-theme();
}
```

10. You will need to make a `ThemeService`, depending on how the application is structured place it with other shared services, although typically it should be inside a shared 
module's services folder.

Assuming it's part of a shared module, you can use a command like below:
```powershell
ng generate service theme --module=shared
```

11. Inside the `ThemeService` do the following:

```typescript
import {Inject, Injectable} from '@angular/core';
import {DOCUMENT} from '@angular/common';

@Injectable({
  providedIn: 'root'
})
export class ThemeService {
  private activeTheme: 'light' | 'dark' = 'light';

  constructor(@Inject(DOCUMENT) private document: Document) {
    this.document.body.classList.add(this.activeTheme);
  }

  toggleTheme(): void {
    this.activeTheme = this.activeTheme === 'light' ? 'dark' : 'light';

    this.document.body.classList.toggle('light');
    this.document.body.classList.toggle('dark');
  }
}
```

To optimize the above, feel free to do the following:
1. Save current theme to local storage, and whenever the theme is toggled to update the local storage value for it
2. Define a theme enum/type for each theme you want to implement instead of two strings like what I did above
3. For multiple themes, change `toggleTheme()` to pass in a specific theme you want to change to (goes hand in hand with point 1)
4. If there are common colours between multiple themes, you may want to define a common.scss where all of those go, and `@include` that in the body without any special class names
5. You may initialize the current theme based on the [browser theme preference](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)
