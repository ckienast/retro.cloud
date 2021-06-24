+++
title = "Simply integrating Svelte into Rails"
date = 2021-06-30
draft = true
+++

At my dayjob I mostly work with Applications built on Ruby on Rails. On a recent Project I wanted to integrate Svelte into the frontend code to replace jQuery. I believe the thing that most people would try in this sort of situation is to find a gem that integrates the frontend framework and provides a view helper to render components into regular views. Since the only gem I could find was not suitable for the project I was working on, I thought about writing the integration myself. This turned out to be a very simple task.

I started with adding the svelte-loader to the project:

```javascript
module.exports = {
  rules: [
    {
      test: /.svelte$/,
      use: [
        {
          loader: 'svelte-loader',
          options: {
            compilerOptions: {
              css: false
            }
          }
        }
      ]
    }
  ]
}
```

With that out of the way, I started planning what I wanted my integration to do. I concluded that It should give me an easy way to render Svelte components into regular (in our case haml) views and handle passing through props to the component.

The first thing I wrote was the view helper:

```ruby
module SvelteHelper
  def svelte_component(name, props: nil)
    if props
      tag.div('data-svelte-component': name, 'data-svelte-props': props.to_json)
    else
      tag.div('data-svelte-component': name)
    end
  end
end
```

This is of course a very rough version and you can add as many features as you want, but I wanted to know if this could work at all. The next thing I wrote was the JavaScript integration, which only consists of a simple `register` function that takes an opject of components:

```typescript
export type ComponentsMap = { [key: string]: any }

export function register(components: ComponentsMap) {
  for (const key in components) {
    const target = document.querySelector(`[data-svelte-component='${key}']`)

    if (!target) {
      continue
    }

    const rawProps = target.getAttribute('data-svelte-props')

    const props = rawProps ? JSON.parse(rawProps) : {}

    const comp = components[key]

    const inst = new comp({
      target,
      props
    })
  }
}
```

The `ComponentsMap` is not typed perfectly but I couldn't find a type declaration for Svelte components that included the constructor. With that done, I tried to use it:

In the view:

```haml
= svelte_component("MySuperCoolComponent", props: { count: 4 })
```

And also in the JavaScript bundle:

```typescript
import { register } from './svelte_manager';
import MySuperCoolComponent from './MySuperCoolComponent.svelte';


register({ MySuperCoolComponent });
```

And it worked! I was very happy.

This integration is of course extremely simplified. It doesn't for example support SSR and hydration. But it works as a stepping stone for getting Svelte components into a Rails project even in a brownfield situation. It also shows, that you don't always need a gem to do the work for you.
