# Decoding Enums

Decoding Structs and Classes is a similar process, but Enums can mix things up.
If your Enum inherits from a Raw Type `String` or `Int`, then making it conform
to `Decodable` is as simple as pie. Let's look at a `Role` type enum:

```swift
enum Role: String {
  case User 
  case Admin
  case SuperAdmin
}
```

To make `Role` conform to `Decodable`, use this one line:

```swift
extension Role: Decodable { }
```

"THAT'S IT?! How?" You ask. Enums with a raw type like `String` or `Int`
conform to `RawRepresentable` and we added a default [implementation] for these
cases.

[implementation]: ../Argo/Extensions/RawRepresentable.swift

So, what if the enum doesn't have a raw type? Conforming to `Decodable` is a
bit more involved but not too hard. Let's look at an example:

```swift
enum FootRace {
  case HalfMarathon
  case Marathon
  case UltraMarathon
}
```

We can write `Decodable` for `FootRace` like so:

```swift
extension FootRace: Decodable {
  static func decode(j: JSON) -> Decoded<FootRace> {
    switch j {
    case let .Number(n):
      switch n {
      case 13.1: return pure(.HalfMarathon)
      case 26.2: return pure(.Marathon)
      case _ where n > 26.2: return pure(.UltraMarathon)
      default: return .typeMismatch("marathon distance", actual: n)
      }
    default: return .typeMismatch("String", actual: j)
    }
  }
}
```

The first thing we do is check to see if the `JSON` is a number. If so, we look
for the matching enum case. Finally, we return the matching case using the
`pure` function which will wrap the enum value in a `Decoded` type.
