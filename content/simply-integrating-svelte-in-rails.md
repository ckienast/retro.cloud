+++
title = "Introduction"
date = 2021-06-30
draft = true
+++

I mostly work with Applications build on Ruby on Rails. On a recent Project I wanted to integrate Svelte into the Frontend code to replace jQuery. The thing most people would try is to find a gem that integrates the frontend Framework and provides a view helper to render components into regular views. Since the only gem I could find was not that great, I thought about writing the integration myself. This turned out to be a very simple task.

I started with adding the svelte-loader to the project.

With that out of the way, I thought about what I wanted my integration to do. It should give me an easy way to render Svelte components into regular (in our case haml) views and handle passing through props to the component.

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

This is of course a very rough version and you can add as many features as you want, but I first wanted to know if this could work. The next thing I wrote was the JavaScript integration, which only consists of a simple `register` function that takes an opject of components:

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

The `ComponentsMap` is not typed perfectly but I couldn't find a type declaration for Svelte components that included the constructor. With that down, I tried to use it:

```haml
= svelte_component("MySuperCoolComponent", props: { count: 4 })
```

```typescript
import { register } from './svelte_manager';
import MySuperCoolComponent from './MySuperCoolComponent.svelte';


register({ MySuperCoolComponent });
```

And it worked! I was very happy.

This integration is of course extremely simplified. It doesn't for example support SSR and hydration. But it works as a stepping stone for getting Svelte components into a Rails project even in a brownfield situation.
