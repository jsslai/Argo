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

    // First, make sure JSON is a number.
    case let .Number(n):

      // Next, match the number to the enum case.
      switch n {

      // When a case matches, use pure to wrap the enum in Decoded.
      case 13.1: return pure(.HalfMarathon) // 13.1 miles is a half marathon distance.
      case 26.2: return pure(.Marathon) // 26.2 miles is a marathon distance.
      case _ where n > 26.2: return pure(.UltraMarathon) // Any distance over a marathon is considered an ultra marathon.

      // Return an error if no case matched
      default: return .typeMismatch("marathon distance", actual: n)
      }

    // Return an error if JSON is not a number.
    default: return .typeMismatch("String", actual: j)
    }
  }
}
```
