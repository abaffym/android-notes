## Memory Leaks on Android

These are some of the common leaks to be careful about.
Enabling [LeakCanary](https://square.github.io/leakcanary/) on the project is strongly recommended!

### 1. View references in Fragment

View references must be cleared in fragment’s `onDestoryView`!

The view gets destroyed but fragment itself lives past that, so any references to the view will cause leak.

This also includes any `DataBinding` or `ViewBinding` references!

```kotlin
var _binding: Binding?
val binding get() = requireNotNull(_binding)

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
  super.onViewCreated(view, savedInstanceState)
  _binding = Binding.inflate()
}

override fun onDestroyView() {
  _binding = null
  super.onDestroyView()
}
```

### 2. RecyclerView Adapter references in Fragment
`RecyclerView` adapters must be cleared if fragment’s `onDestoryView`! 

Adapters internally holds reference to the recycler via `AdapterDataObservable` @see [here](https://charlesmuchene.com/a-subtle-memory-leak-fragment-recyclerview-and-its-adapter-ck805s7jd03frzns17uapi3vh?guid=none&deviceId=e49c4ae3-a07d-48d4-a127-5b41ffb41cfb).

You can clean adapter either by:

```kotlin
var _adapter: MyAdapter?
val adapter get() = requireNotNull(_adapter)

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
  super.onViewCreated(view, savedInstanceState)
  _adapter = MyAdapter()
}

override fun onDestroyView() {
  _adapter = null
  super.onDestroyView()
}
```

or

```kotlin
val recyclerView: RecyclerView
val adapter = MyAdapter()

override fun onDestroyView() {
  recyclerView.adapter = null
  super.onDestroyView()
}
```

Note: The second approach does not clean the `AdapterDataObserver` so you have to do it manually!

### 3. ViewPager2 Adapter references in Fragment
`ViewPager2` is internally implemented as a `RecyclerView` so the same rules as in point 2 apply.

## Resources

Live tracking memory leaks (pretty useful if you want to improve your LeakCanary skills!): [Ask the Expert #2: LeakCanary 2.x is out, track your memory leaks](https://www.youtube.com/watch?v=Sx0k4ipqwBs&feature=emb_title)
