# Mews styleguide for Flutter apps

- [Introduction](##introduction)
- [Overview](##overview)
- [Philosophy](##philosophy)
- [Documentation](##user-content-documentation)

## Introduction

This document contains some high-level philosophy and policy decisions for the Flutter project, and a description of specific style issues for some parts of the codebase.

It's heavily based on [Style guide for Flutter repo](https://github.com/flutter/flutter/wiki/Style-guide-for-Flutter-repo).

## Overview

This document describes our approach to designing and programming Flutter app, from high-level architectural principles all the way to indentation rules. These are our norms, written down so that we can easily convey our shared understanding with new team members.

The primary goal of these style guidelines is to improve code readability.
Secondary goals are to design systems that are simple, to increase the likelihood of catching bugs quickly, and avoiding arguments when there are disagreements over subjective matters.

For anything not covered by this document, check the [Dart style guide](https://www.dartlang.org/guides/language/effective-dart/) for more advice. That document is focused primarily on Dart-specific conventions, while this document is more about Flutter conventions.

## Philosophy

### Lazy programming

Write what you need and no more, but when you write it, do it right.

Avoid implementing features you don’t need. You can’t design a feature without knowing what the constraints are. Implementing features "for completeness" results in unused code that is expensive to maintain, learn about, document, test, etc.

When you do implement a feature, implement it the right way. Avoid workarounds. Workarounds merely kick the problem further down the road, but at a higher cost: someone will have to relearn the problem, figure out the workaround and how to dismantle it (and all the places that now use it), and implement the feature. It’s much better to take longer to fix a problem properly, than to be the one who fixes everything quickly but in a way that will require cleaning up later.

### Write Test, Find Bug

When you fix a bug, first write a test that fails, then fix the bug and verify the test passes.

When you implement a new feature, think if it makes sense to cover it with tests.

Avoid submitting code with the promise to "write tests later". Just take the time to write the tests properly and completely in the first place.

### Getters feel faster than methods

Property getters should be efficient (e.g. just returning a cached value, or an O(1) table lookup). If an operation is inefficient, it should be a method instead.

Similarly, a getter that returns a Future should not kick-off the work represented by the future, since getters appear idempotent and side-effect free. Instead, the work should be started from a method or constructor, and the getter should just return the preexisting Future.

### Avoid secret (or global) state

A function should operate only on its arguments and, if it is an instance method, data stored on its object. This makes the code significantly easier to understand.

For example, when reading this code:

```dart
// ... imports something that defines foo and bar ...

void main() {
  foo(1);
  bar(2);
}
```

…​the reader should be confident that nothing in the call to foo could affect anything in the call to bar.

This usually means structuring APIs so that they either take all relevant inputs as arguments, or so that they are based on objects that are created with the relevant input, and can then be called to operate on those inputs.

This significantly aids in making code testable and in making code understandable and debuggable. When code operates on secret global state, it’s much harder to reason about.

The exception to this rule is using providers for passing dependencies down the tree. Be reasonable about using them and tend to pass the dependencies explicitly if it makes sense.

## Documentation

### Avoid useless documentation

If someone could have written the same documentation without knowing anything about the class other than its name, then it’s useless.

Example:

```dart
// BAD:

/// The background color.
final Color backgroundColor;

/// Half the diameter of the circle.
final double radius;


// GOOD:

/// The color with which to fill the circle.
///
/// Changing the background color will cause the avatar to animate to the new color.
final Color backgroundColor;

/// The size of the avatar.
///
/// Changing the radius will cause the avatar to animate to the new size.
final double radius;
```

### Writing prompts for good documentation

If you are having trouble coming up with useful documentation, here are some prompts that might help you write more detailed prose:

- If someone is looking at this documentation, it means that they have a question which they couldn’t answer by guesswork or by looking at the code. What could that question be? Try to answer all questions you can come up with.
- If you were telling someone about this property, what might they want to know that they couldn’t guess? For example, are there edge cases that aren’t intuitive?
- Consider the type of the property or arguments. Are there cases that are outside the normal range that should be discussed? e.g. negative numbers, non-integer values, transparent colors, empty arrays, infinities, NaN, null? Discuss any that are non-trivial.
- Does this member interact with any others? For example, can it only be non-null if another is null? Will this member only have any effect if another has a particular range of values? Will this member affect whether another member has any effect, or what effect another member has?
- Does this member have a similar name or purpose to another, such that we should point to that one, and from that one to this one? Use the See also: pattern.
- Are there timing considerations? Any potential race conditions?
- Are there lifecycle considerations? For example, who owns the object that this property is set to? Who should dispose() it, if that’s relevant?
- What is the contract for this property/method? Can it be called at any time? Are there limits on what values are valid? If it’s a final property set from a constructor, does the constructor have any limits on what the property can be set to? If this is a constructor, are any of the arguments not nullable?
- If there are Future values involved, what are the guarantees around those? Consider whether they can complete with an error, whether they can never complete at all, what happens if the underlying operation is canceled, and so forth.

### Avoid empty prose

It’s easy to use more words than necessary. Avoid doing so where possible, even if the result is somewhat terse.

```dart
// BAD:

/// Note: It is important to be aware of the fact that in the
/// absence of an explicit value, this property defaults to 2.

// GOOD:

/// Defaults to 2.
```

In particular, avoid saying "Note:". It adds nothing.

### Refactor the code when the documentation would be incomprehensible

If writing the documentation proves to be difficult because the API is convoluted, then rewrite the API rather than trying to document it.

### Canonical terminology

The documentation should use consistent terminology:

- _method_ - a member of a class that is a non-anonymous closure
- _function_ - a callable non-anonymous closure that isn’t a member of a class
- _parameter_ - a variable defined in a closure signature and possibly used in the closure body.
- _argument_ - the value passed to a closure when calling it.

Prefer the term "call" to the term "invoke" when talking about jumping to a closure.

Prefer the term "member variable" to the term "instance variable" when talking about variables associated with a specific object.

Typedef dartdocs should usually start with the phrase "Signature for…​".

### Use correct grammar

Avoid starting a sentence with a lowercase letter.

```dart
// BAD

/// [foo] must not be null.

// GOOD

/// The [foo] argument must not be null.
```

Similarly, end all sentences with a period.

### Dartdoc templates and macros

Dartdoc supports creating templates that can be reused in other parts of the code. They are defined like so:

```dart
/// {@template <id>}
/// ...
/// {@endtemplate}
```

and used via:

```dart
/// {@macro <id>}
```

The `<id>` should be a unique identifier that is of the form `library.Class.member[.optionalDescription]`.

The `optionalDescription` component of the identifier is only necessary if there is more than one template defined in one Dartdoc block. If a symbol is not part of a library, or not part of a class, then just omit those parts from the ID.

### Dartdoc-specific requirements

The first paragraph of any dartdoc section must be a short self-contained sentence that explains the purpose and meaning of the item being documented. Subsequent paragraphs then must elaborate. Avoid having the first paragraph have multiple sentences. (This is because the first paragraph gets extracted and used in tables of contents, etc, and so has to be able to stand alone and not take up a lot of room.)

## Coding patterns and catching bugs early

### Use asserts liberally to detect contract violations and verify invariants

`assert()` allows us to be diligent about correctness without paying a performance penalty in release mode, because Dart only evaluates asserts in debug mode.

It should be used to verify contracts and invariants are being met as we expect. Asserts do not enforce contracts, since they do not run at all in release builds. They should be used in cases where it should be impossible for the condition to be false without there being a bug somewhere in the code.

### Prefer specialized functions, methods and constructors

Use the most relevant constructor or method, when there are multiple options.

Example:

```dart
// BAD:
const EdgeInsets.TRBL(0.0, 8.0, 0.0, 8.0);

// GOOD:
const EdgeInsets.symmetric(horizontal: 8.0);
```

### Minimize the visibility scope of constants

Prefer using a local const or a static const in a relevant class than using a global constant.

As a general rule, when you have a lot of constants, wrap them in a class.

### Avoid using if chains or ?: or == with enum values

Use `switch` with no `default` case if you are examining an enum, since the analyzer will warn you if you missed any of the values when you use `switch`. The `default` case should be avoided so that the analyzer will complain if a value is missing. Unused values can be grouped together with a single break or return as appropriate.

Avoid using `if` chains, `? …​ : …`​, or, in general, any expressions involving enums.

### Avoid mysterious and magical numbers that lack a clear derivation

Numbers in tests and elsewhere should be clearly understandable. When the provenance of a number is not obvious, consider either leaving the expression or adding a clear comment (bonus points for leaving a diagram).

```dart
// BAD
expect(rect.left, 4.24264068712);

// GOOD
expect(rect.left, 3.0 * math.sqrt(2));
```

### Be explicit about `dispose()` and the object lifecycle

Even though Dart is garbage collected, having a defined object lifecycle and explicit ownership model (describing in the API documentation who is allowed to mutate the object, for instance) is important to avoid subtle bugs and confusing designs.

If your class has a clear "end of life", for example, provide a `dispose()` method to clean up references such as listeners that would otherwise prevent some objects from getting garbage collected. For example, consider a widget that has a subscription on a global broadcast stream (that might have other listeners). That subscription will keep the widget from getting garbage collected until the stream itself goes away (which, for a global stream, might never happen).

In general, pretending that Dart does not have garbage collection is likely to lead to less confusing and buggy code, because it forces you to think about the implications of object ownership and lifecycles.

### Immutable classes should not have hidden state

Immutable classes (those with const constructors) should not have hidden state. For example, they should not use private statics. If they are stateful, then they should not be const.

### Don't overuse `late` keyword

Don't overuse `late` keyword, it's not needed in situations like this:

```dart
late final String x; // late not needed

if (someCond) {
  x = '1';
else {
  x = '2';
}
```

Apart from having unnecessary keyword, it can lead to some subtle errors. You can read more about it [here](https://dev.to/ookamikb/dont-be-always-late-51jm).

### Extract widgets

Flutter is all about composition of widgets. It's better to have a lot of small widgets than a few big ones. Try to avoid introducing `_build*` methods inside `Widget` class, as they can lead to some subtle errors with wrong context.

### Use `typedef` for better readability

Use typedef for better readability and especially when reusing function types, e.g.:

```dart
typedef DateTimeFormatter = String Function(DateTime);
```

## Naming

### Naming rules for typedefs and function variables

When naming callbacks, use `FooCallback` for the typedef, `onFoo` for the callback argument or property, and `handleFoo` for the method that is called.

Never call a method `onFoo`. If a property is called `onFoo` it must be a function type. (For all values of "Foo".)

Prefer using `typedef`s to declare callbacks. Typedefs benefit from having documentation on the type itself and make it easier to read and find common callsites for the signature.

### Avoid double negatives in APIs

Name your boolean variables in positive ways, such as "enabled" or "visible", even if the default value is true.

This is because, when you have a property or argument named "disabled" or "hidden", it leads to code such as `input.disabled = false` or `widget.hidden = false` when you’re trying to enable or show the widget, which is very confusing.

### Prefer naming the argument to a setter value

Unless this would cause other problems, use value for the name of a setter’s argument. This makes it easier to copy/paste the setter later.

### Naming rules for BLoCs

Base event class should be named as `[BaseName]Event` (e.g. `TaskDetailsEvent`). Event case constructors should be named as verbs in past tense (as in _something happened_), e.g.: `fetchRequested`, `accountUpdated` etc. Event case class name should be either simple form (`FetchRequested`) or prefixed with base class name without `Event` suffix (`TaskDetailsFetchRequested` for `TaskDetailsEvent` class).

```dart
const factory TaskDetailsEvent.fetchRequested(String? reservationId) = FetchRequested;
```

Base state class should be named as `[BaseName]State` (e.g. `TaskDetailsState`). If state is a sealed class, its cases should be nouns or gerunds (e.g. `loading`, `failure`). State case class name should be either simple form (`Loading`) or prefixed with base class name without `State` suffix (`TaskDetailsLoading` for `TaskDetailsState` class).

BLoC class should be named as `[BaseName]Bloc` (e.g. `TaskDetailsBloc`).

### Naming rules for DTOs

Top-level DTOs (passed as request object and received as response object) should be named by the operation they are used in. For example, for the API endpoint `getKiosks` define `GetKiosksRequestDto` and `GetKiosksResponseDto`.

## Comments

### Comment all `// ignores`

| :information_source: | Enforced by [prefer-commenting-analyzer-ignores] rule. |
| :------------------: | :----------------------------------------------------- |

[prefer-commenting-analyzer-ignores]: https://dartcodemetrics.dev/docs/rules/common/prefer-commenting-analyzer-ignores

Sometimes, it is necessary to write code that the analyzer is unhappy with.

If you find yourself in this situation, consider how you got there. Is the analyzer actually correct but you don’t want to admit it? Think about how you could refactor your code so that the analyzer is happy. If such a refactor would make the code better, do it. (It might be a lot of work…​ embrace the yak shave.)

If you are really really sure that you have no choice but to silence the analyzer, use `// ignore:`. The ignore directive should be on the same line as the analyzer warning.

If the ignore is temporary (e.g. a workaround for a bug in the compiler or analyzer, or a workaround for some known problem in Flutter that you cannot fix), then add a link to the relevant bug, as follows:

```dart
foo(); // ignore: lint_code, https://link.to.bug/goes/here
```

If the ignore directive is permanent, e.g. because one of our lints has some unavoidable false positives and in this case violating the lint is definitely better than all other options, then add a comment explaining why:

```dart
foo(); // ignore: lint_code, sadly there is no choice but to do
// this because we need to twiddle the quux and the bar is zorgle.
```

### Comment all test skips

On very rare occasions it may be necessary to skip a test. To do that, use the skip argument. Any time you use the skip argument, file an issue describing why it is skipped and include a link to that issue in the code.

### Comment empty closures to setState

Generally, the closure passed to setState should include all the code that changes the state. Sometimes this is not possible because the state changed elsewhere and the setState is called in response. In those cases, include a comment in the setState closure that explains what the state is that changed.

```dart
setState(() { /* The animation ticked. We use the animation's value in the build method. */ });
```
