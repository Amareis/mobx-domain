# mobx-domain

Currently I'm developing app, which is complex enough to reach mobx-state-tree limits, so I trying to design library, more applicable for complex realtime client applications.

## References
* https://github.com/mobxjs/mobx-state-tree
* https://github.com/mobxjs/serializr
* https://github.com/mmlpxjs/mmlpx
* https://github.com/tommikaikkonen/redux-orm
* https://github.com/charto/classy-mst
* https://mobx.js.org/best/store.html
* https://docs.djangoproject.com/en/2.1/topics/db/models/

## Prerequisites

mobx-state-tree have really good approaches for many use cases, but it isn't very convenient for classic OOP (eg, non-classes approach raises horrible cyclic types issues in typescript). But it's not some significant problem, unlike this three flaws:

1. There is no any automatic normalization, which is just needed sometimes.
For example, in our application there is `File` model, which can be loaded from backend in two different places of tree, so I need to manually destroy model at first place every time as I load it at second place (to avoid duplicate ids).
Some sort of generic denormalization can solve this problem easily, something like an ORM (like redux-orm, but for mobx), which automatically put every model to its store at hydration phase and manage references between it.
2. Smart components (containers, in React terminology) often need to store direct references to models.
Bad side here - model can be destroyed and all references will be broken, so we need to some reference wrapper, which will handle model destroying (or even prevent it).
And it's not just about components, ui stores also are external to domain stores, so every reference in it also will be broken after domain model destroying (mst currently solves this via `safeReference`, but components references still in danger).
3. Previous issue becomes even more complicated if we trying to hot replace domain stores.
Simple example - in realtime app after offline period we need to completely reload state from backend, to get actual data.
If we creates new stores and populate it with data and then atomically replaces old stores, everything will be ok - excludes the external references, which points to old stores!
We can set references to undefined when model destroyed, but currently there is simply no way to automatically redirect they to new domain store.


So, basically, my vision of domain layer of mobx application is:
1. `Model`. Just like mst `types.model` with identifier, but with ES classes (classy-mst is good library which shows how it can be made).  Just like in mst, Model fields should be divided on main, serializable fields and volatile ones. mst `types.model` and django-orm syntax are good how-to examples.
```ts
class Todo extends Model {
    title = fields.string()
    checked = fields.boolean()
    
    loading = false
}
```
