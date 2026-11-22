# Rust checked arithmetic

Rust arithmetic in high-reliability software never panics, never wraps, never silently truncates, and never invents a
number. Financial, governmental, medical and similar systems need this. When a result does not fit, there is no
result. The operation returns `None` or `Err`. The code handles that as close to the operation as it can. When the
specification states the correct value for that case, the code recovers on the spot with `unwrap_or`. Otherwise it
returns an error value, a refusal or a stated "unknown". It never substitutes a zero, a maximum, a clamp or a wrapped
value that the specification did not declare.

This file is the source of truth for specification-driven development. Specifications, designs, code reviews and
tests cite its requirements by ID (for example `RCA-INT-2`). The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY
follow RFC 2119 and RFC 8174.

## Scope

- **RCA-SCOPE-1.** These rules apply to all production code in every crate of the workspace: libraries, binaries,
  build scripts that compute values, and macros that expand into arithmetic.
- **RCA-SCOPE-2.** The rules apply most strictly to **external figures**. An external figure is any value that comes
  from, or depends on, input, storage, the network, the clock, the environment, configuration, another crate's
  output, or an earlier computation on an external figure. Examples: a count, amount, price, rate, dose, weight,
  duration, date, size, length, offset, page number, identifier or version.
- **RCA-SCOPE-3.** When in doubt, a value is external.
- **RCA-SCOPE-4.** Tests, benchmarks and fuzz harnesses MAY use plain operators. Their *assertions* about production
  code MUST still exercise the checked paths (see [Testing and verification](#testing-and-verification)).

## Principles

- **RCA-P-1. No panic.** Production arithmetic MUST NOT panic. A panic mid-batch is an outage at best and a
  half-applied transaction at worst.
- **RCA-P-2. No silent wrong answer.** Production arithmetic MUST NOT wrap, truncate, saturate, round or lose
  precision unless the specification for that figure names that behavior. A silently wrong figure is worse than a
  panic, because nothing detects it.
- **RCA-P-3. An unknown is never a zero.** A result that cannot be computed, and has no declared fallback, is absent
  (`None`) or an error (`Err`). It MUST NOT become `0`, `T::MAX`, `T::MIN`, `Default::default()` or any other sentinel.
  Zero is a real value with a real meaning: zero balance, zero dose, zero days remaining. A declared fallback
  (RCA-P-7) is not an unknown. It is the answer the specification gives for that case.
- **RCA-P-4. Failure is typed and attributable.** Every arithmetic failure MUST surface as a typed error that names the
  figure and the operation, so an operator can tell *which* number failed and *why*.
- **RCA-P-5. Make invalid values unrepresentable.** Domain types SHOULD enforce range, unit and currency when they are
  constructed. This leaves fewer places where arithmetic can fail.
- **RCA-P-6. Defense in depth.** Types, lints, build profile and tests each enforce these rules.
- **RCA-P-7. Recover at the source.** A failure is recoverable when the specification states the correct value for
  that case. Code SHOULD recover from it right at the checked operation, with `unwrap_or` and the declared fallback.
  This keeps the reason in view where the failure happens, and keeps error types small. It also stops a recoverable
  case from travelling up the stack as an error that a distant caller has to interpret.

## Numeric figure declarations

In specification-driven development, the specification declares each figure before the code computes it.

- **RCA-DECL-1.** Every external figure in a specification MUST declare:

  | Property | Question it answers | Example |
  |---|---|---|
  | Name | What is it? | `invoice_total` |
  | Unit | Measured in what? | Minor currency units (cents), with a currency code |
  | Rust type | Which representation? | `Money { amount: i64, currency: Currency }` |
  | Valid range | What values are legal? | `0 ..= 99_999_999_999` |
  | Unknown | How is "not known" represented? | `Option<Money>`; `None` is shown as "unknown" |
  | Rounding | When precision is lost, which way? | Half-to-even at 2 decimal places, once, at the end |
  | Overflow | What happens out of range? | `Err(ArithmeticError::Overflow { figure: "invoice_total", .. })` |
  | Fallback | Is there a correct value when an operation fails? | None: an invoice total that overflows is an error |

- **RCA-DECL-2.** Code MUST NOT pick a rounding mode or a fallback that the declaration does not name. A declaration
  MUST NOT name wrapping. When a declaration bounds a result, it names the bound as a fallback (RCA-DECL-4). The code
  writes it as `checked_*` plus `unwrap_or`, not as `saturating_*` or `clamp` (RCA-INT-4, RCA-INT-6).
- **RCA-DECL-3.** When the code and the declaration disagree, the declaration is correct and the code is a defect.
  When the declaration is missing or ambiguous, the implementer MUST raise it before writing code. The implementer MUST
  NOT choose a behavior silently.
- **RCA-DECL-4.** A declaration SHOULD name a fallback wherever the domain has a correct answer for a failed
  operation, so that code can recover at the source (RCA-P-7). A fallback MUST be a valid value of the figure, within
  its declared range, and the declaration MUST say why it is correct. Examples:

  | Figure | Failure | Declared fallback | Why it is correct |
  |---|---|---|---|
  | `remaining_quota` | `limit - used` underflows | `0` | Usage above the limit (after the limit is lowered) leaves no quota |
  | `retry_delay_ms` | `base * 2^attempt` overflows | `MAX_DELAY_MS` | The delay is capped at `MAX_DELAY_MS` by design |
  | `digits_minus_one` | `ilog10(0)` is undefined | `0` | Zero has one digit, like 1 to 9 |

- **RCA-DECL-5.** A failure that signals invalid, corrupt or inconsistent data is not recoverable. Its declaration
  MUST NOT name a fallback for it. Such a failure MUST become an error or an "unknown".

## Integer operations

- **RCA-INT-1.** Arithmetic on external figures MUST use the checked methods. They return `Option<T>`, and `None` means
  the true result does not fit `T` or does not exist.

  | Operation | Checked method | Returns `None` when |
  |---|---|---|
  | Add | `checked_add` | Result is out of range |
  | Subtract | `checked_sub` | Result is out of range (for unsigned types, any result below zero) |
  | Multiply | `checked_mul` | Result is out of range |
  | Divide | `checked_div` | Divisor is zero, or signed `MIN / -1` |
  | Remainder | `checked_rem` | Divisor is zero, or signed `MIN % -1` |
  | Euclidean divide and remainder | `checked_div_euclid`, `checked_rem_euclid` | Same as above |
  | Power | `checked_pow` | Result is out of range |
  | Negate | `checked_neg` | Signed `MIN`, or any nonzero unsigned value |
  | Absolute value | `checked_abs` | Signed `MIN` |
  | Shift | `checked_shl`, `checked_shr` | Shift amount is at least the bit width (see [Shifts](#shifts-and-bit-operations)) |
  | Mixed sign | `checked_add_signed` (unsigned + signed); `checked_add_unsigned`, `checked_sub_unsigned` (signed with unsigned) | Result is out of range |
  | Logarithm | `checked_ilog2`, `checked_ilog10`, `checked_ilog` | Argument is not positive, or base is less than 2 |
  | Rounding up | `checked_next_power_of_two`, `checked_next_multiple_of` (unsigned) | Result is out of range, or multiple is zero |

  ```rust
  assert_eq!(255_u8.checked_add(1), None);      // not 0 (wrapped), not 255 (saturated), not a panic
  assert_eq!(250_u8.checked_add(5), Some(255));
  assert_eq!(3_u32.checked_sub(5), None);       // unsigned cannot go below zero
  assert_eq!(i32::MIN.checked_abs(), None);
  assert_eq!(i32::MIN.checked_div(-1), None);   // overflow, not only division by zero
  assert_eq!(7_u32.checked_div(0), None);       // not a panic
  ```

- **RCA-INT-2.** Production code MUST NOT use the plain operators `+`, `-`, `*`, `/`, `%`, `<<`, `>>` or unary `-` on
  external figures. The same applies to their compound forms (`+=`, `-=`, `*=`, `/=`, `%=`, `<<=`, `>>=`) and to the
  panicking methods (`pow`, `abs`, `div_euclid`, `rem_euclid`, `isqrt`, `ilog2`, `ilog10`, `next_power_of_two`,
  `div_ceil`, `next_multiple_of`). These behave differently by build:

  | Expression | Debug build | Release build (default profile) |
  |---|---|---|
  | `a + b` that overflows | Panics | Wraps silently |
  | `a / 0`, `a % 0` | Panics | Panics |
  | `i32::MIN / -1` | Panics | Panics |
  | `a << 40` on `u32` | Panics | Masks the shift amount to `40 % 32` silently |
  | `i32::MIN.abs()` | Panics | Returns `i32::MIN` silently |

- **RCA-INT-3.** Production code MUST NOT use the `strict_*` methods. They panic by design.
- **RCA-INT-4.** Production code MUST NOT use saturating or wrapping arithmetic: any `saturating_*` or `wrapping_*`
  method on any type, `std::num::Saturating<T>` or `std::num::Wrapping<T>`. There is no exception (RCA-EXC-3,
  RCA-EXC-4). A saturated result is an undeclared `MIN` or `MAX`; a wrapped result is an invented anything. Where the
  declaration names a bound, write `checked_*` plus `unwrap_or(BOUND)` (RCA-NONE-2). This keeps the bound visible,
  cited and tested.
- **RCA-INT-5.** `overflowing_*` MUST NOT be used: its first value is the wrapped result. Use `checked_*`.
- **RCA-INT-6.** These methods never overflow and MAY be used freely: `abs_diff` (returns the unsigned type),
  `unsigned_abs`, `min`, `max`, `count_ones`, `leading_zeros`, `trailing_zeros`, `is_positive`, `is_negative` and
  `signum`. `clamp` MUST NOT be applied to an external figure: an out-of-range value is an error, not a bound. When
  `min` or `max` bounds an external figure, the declaration MUST name the bound, and the call MUST cite it.
  `x.max(lo).min(hi)` is a clamp and MUST NOT be used on an external figure.
- **RCA-INT-7.** Arithmetic on `std::num::NonZero<T>` (for example `NonZero<u64>`) MUST also use its checked methods
  (`checked_add`, `checked_mul`, `checked_pow`).

## Handling the absent result

- **RCA-NONE-1.** Production code MUST NOT turn the result of a checked operation into a panic or an undeclared
  value:

  ```rust
  // All forbidden in production code:
  a.checked_add(b).unwrap();
  a.checked_add(b).expect("cannot overflow");
  unsafe { a.checked_add(b).unwrap_unchecked() };
  a.checked_add(b).unwrap_or_default(); // the fallback is implicit, so the reader cannot see it
  a.checked_add(b).unwrap_or(0);        // forbidden when no declaration names 0 as the fallback
  ```

- **RCA-NONE-2.** Production code MUST handle the `None` in one of the ways below. It SHOULD handle it as close to the
  checked operation as possible, and SHOULD prefer the ways in this order:
  1. **Recover** with `unwrap_or(fallback)` when the declaration names a fallback for that failure (RCA-DECL-4). Use
     `unwrap_or_else(|| ...)` when the fallback is costly to compute. Use `map_or(fallback, f)` or `map_or_else` when
     the success value is transformed too.
  2. **Match** on it when the two branches do meaningfully different work, for example refusing a transaction.
  3. **Convert** it with `ok_or(...)` or `ok_or_else(...)` and `?` in a function that returns `Result<T, E>`. The error
     names the figure. Convert at the checked operation, where the figure and the operation are known, not in a distant
     caller.
  4. **Propagate** it with `?` in a function that returns `Option<T>`, when "unknown" is a legitimate answer for the
     caller.

  ```rust
  // Recover at the source with a declared fallback.

  /// Spec QUOTA-2: usage above the limit (after the limit is lowered) leaves no quota.
  fn remaining_quota(limit: u64, used: u64) -> u64 {
      limit.checked_sub(used).unwrap_or(0) // QUOTA-2
  }

  /// Spec RETRY-1: the delay is BASE_DELAY_MS * 2^attempt, capped at MAX_DELAY_MS.
  const BASE_DELAY_MS: u64 = 100;
  const MAX_DELAY_MS: u64 = 60_000;

  fn retry_delay_ms(attempt: u32) -> u64 {
      2_u64
          .checked_pow(attempt)
          .and_then(|factor| BASE_DELAY_MS.checked_mul(factor))
          .unwrap_or(MAX_DELAY_MS) // RETRY-1: overflow means "past the cap"
          .min(MAX_DELAY_MS) // RETRY-1: the cap
  }

  /// Spec FMT-4: zero has one digit, like 1 to 9.
  fn digits_minus_one(n: u64) -> u32 {
      n.checked_ilog10().unwrap_or(0) // FMT-4
  }

  // No declared fallback: convert to a named error at the source.
  fn line_total(unit_price: u64, quantity: u32) -> Result<u64, ArithmeticError> {
      unit_price
          .checked_mul(u64::from(quantity))
          .ok_or(ArithmeticError::Overflow { figure: "line_total", operation: Operation::Mul })
  }
  ```

- **RCA-NONE-3.** A fallback passed to `unwrap_or`, `unwrap_or_else`, `map_or` or `map_or_else` MUST:
  1. be the fallback that the figure's declaration names, with a comment at the call that cites the declaration;
  2. be a valid value of the figure, within its declared range;
  3. not stand in for "unknown" (RCA-P-3) or for invalid data (RCA-DECL-5);
  4. not panic, and, for a closure, not have side effects beyond computing the value.

  The fallback SHOULD be a named constant, unless the declaration names a literal and the comment cites it.
- **RCA-NONE-4.** A `None` from arithmetic MUST NOT be logged and then discarded. It is replaced by its declared
  fallback, reaches the caller, or becomes a user-visible refusal or "unknown".
- **RCA-NONE-5.** When a figure is displayed, "unknown" MUST look different from zero ("—" or "unknown", not "0"). Its
  stored form MUST also differ from zero (`NULL` or an absent field, not `0`).

## Errors

- **RCA-ERR-1.** Each crate that does arithmetic on external figures SHOULD define one error type for arithmetic
  failure, or reuse a workspace-wide one. The error MUST carry the figure's name and the operation. It SHOULD
  distinguish overflow, division by zero, out-of-range conversion and domain violation.

  ```rust
  #[derive(Debug, Clone, Copy, PartialEq, Eq)]
  pub enum Operation { Add, Sub, Mul, Div, Rem, Neg, Abs, Pow, Shl, Shr, Convert }

  #[derive(Debug, Clone, Copy, PartialEq, Eq, thiserror::Error)]
  pub enum ArithmeticError {
      #[error("{figure}: overflow in {operation:?}")]
      Overflow { figure: &'static str, operation: Operation },
      #[error("{figure}: division by zero")]
      DivisionByZero { figure: &'static str },
      #[error("{figure}: value does not fit the target type")]
      OutOfRange { figure: &'static str },
      #[error("{figure}: {reason}")]
      Domain { figure: &'static str, reason: &'static str },
  }
  ```

- **RCA-ERR-2.** Arithmetic errors MUST NOT include the operand values by default. Amounts, doses, ages, incomes and
  identifiers may be personal, financial or health data. Log values only where the data-handling policy allows it, and
  only through the project's redaction mechanism.
- **RCA-ERR-3.** An arithmetic error inside a transaction MUST abort the whole transaction. A partial result MUST NOT
  be persisted, sent or shown as if it were complete.
- **RCA-ERR-4.** A user-facing arithmetic error MUST say the value could not be computed, never show a number.

## Conversions

- **RCA-CONV-1.** Converting between integer types MUST use `From` / `Into` when the conversion is lossless (for
  example `u64::from(u32)` or `i64::from(i32)`), and `TryFrom` / `TryInto` otherwise. The conversion MUST NOT use `as`.

  ```rust
  let n: u32 = u32::try_from(big_u64).map_err(|_| ArithmeticError::OutOfRange { figure: "page_count" })?;
  let m: u64 = u64::from(small_u32); // lossless: From exists, so no error path is needed
  ```

- **RCA-CONV-2.** The `as` operator MUST NOT be used for numeric conversion of external figures. It is silent in every
  case:

  | `as` conversion | Silent behavior |
  |---|---|
  | Wider integer to narrower (`u64 as u32`) | Truncates high bits |
  | Signed to unsigned (`-1_i32 as u32`) | Reinterprets: `4_294_967_295` |
  | Unsigned to signed (`u32::MAX as i32`) | Reinterprets: `-1` |
  | Float to integer (`f64 as i64`) | Saturates out-of-range values, and **`NaN` becomes `0`** |
  | Integer to float (`u64 as f64`) | Rounds values above 2^53 to the nearest representable float |
  | `usize` / `isize` to fixed width | Depends on the target (32-bit and wasm32 targets differ from 64-bit) |

- **RCA-CONV-3.** Float to integer conversion MUST check, before converting, that the value is finite, has the intended
  integrality or rounding, and is within range:

  ```rust
  /// Converts an integral `f64` to `i64` exactly, or returns `None`.
  fn f64_to_i64_exact(x: f64) -> Option<i64> {
      // 2^63 is exactly representable as f64. i64::MAX is not: `i64::MAX as f64` rounds up to 2^63.
      const TWO_POW_63: f64 = 9_223_372_036_854_775_808.0;
      if !(x.is_finite() && x.fract() == 0.0 && x >= -TWO_POW_63 && x < TWO_POW_63) {
          return None;
      }
      #[expect(clippy::as_conversions, clippy::cast_possible_truncation, reason = "RCA-CONV-3: checked above")]
      let n = x as i64;
      Some(n)
  }
  ```

- **RCA-CONV-4.** Integer to float conversion MUST use `f64::from` (which exists only for lossless sources such as
  `u32` and `i32`), or MUST check that the value's magnitude is at most 2^53 before converting.
- **RCA-CONV-5.** Parsing MUST use `str::parse::<T>()` or `T::from_str_radix` and handle the `Err`. Parsing already
  rejects out-of-range input. The `Err` MUST NOT be unwrapped or replaced with a default.
- **RCA-CONV-6.** Platform-sized integers (`usize`, `isize`) MUST NOT be stored, serialized or used in domain
  calculations. Use fixed-width types (`u32`, `u64`, `i64`) for domain figures, and convert at the boundary with
  `usize::try_from` / `u64::try_from`.

## Division, remainder and rounding

- **RCA-DIV-1.** Every division and remainder MUST handle a zero divisor. Prefer a divisor of type `NonZero<T>`, which
  moves the check to the place where the divisor is created. `u64 / NonZero<u64>` and `u64 % NonZero<u64>` never panic.

  ```rust
  use std::num::NonZero;

  fn per_member(total: u64, members: NonZero<u64>) -> u64 {
      total / members // cannot panic: the divisor is nonzero by type, and unsigned division cannot overflow
  }
  ```

  Signed division by `NonZero<i64>` can still overflow (`i64::MIN / -1`), so signed division MUST use `checked_div`.
- **RCA-DIV-2.** Integer division truncates toward zero (`-7 / 2 == -3`). Euclidean division rounds toward negative
  infinity for positive divisors (`(-7).div_euclid(2) == -4`). The figure's declaration MUST state which one is meant.
  For signed figures, code MUST use `checked_div` or `checked_div_euclid` to match the declaration.
- **RCA-DIV-3.** Integer division discards the remainder. When the remainder matters (money, doses, quotas), the code
  MUST either distribute it, as in [Money and decimals](#money-and-decimals), or round it as the declaration says.
- **RCA-DIV-4.** When a division loses precision, it SHOULD happen last. Multiply first, in a wide enough type (see
  [Wide intermediates](#wide-intermediates-ratios-and-percentages)), then divide once.
- **RCA-DIV-5.** Rounding MUST use a named strategy that the declaration specifies, such as half-to-even (banker's),
  half-away-from-zero, toward zero, floor or ceiling. Rounding MUST happen once, at the point the declaration states.
  Rounding at every intermediate step causes cumulative drift.

## Shifts and bit operations

- **RCA-BIT-1.** `checked_shl` and `checked_shr` return `None` **only when the shift amount is at least the bit width**.
  They do **not** detect bits shifted out of the value: `0x8000_0000_u32.checked_shl(1) == Some(0)`. To multiply by a
  power of two, code MUST use `checked_mul` or `checked_pow`, not a shift.
- **RCA-BIT-2.** Shifts and bitwise operators (`&`, `|`, `^`, `!`) MAY be used for bit fields, flags, masks and
  encodings, where the specification defines the operation on bits rather than on quantities. The shift amount MUST be
  a constant, or MUST be checked to be less than the bit width.
- **RCA-BIT-3.** Bitwise operators MUST NOT be used as arithmetic shortcuts on external figures (for example `x >> 1`
  instead of `x / 2` on a signed amount, which rounds toward negative infinity rather than toward zero).

## Aggregation

- **RCA-AGG-1.** `Iterator::sum` and `Iterator::product` MUST NOT be used on external figures. They use the plain
  operators, so they panic in debug builds and wrap in release builds. Summing into `Option<T>` (`.sum::<Option<T>>()`)
  does **not** check overflow either: it only stops at a `None` element. Use `try_fold` with a checked operation:

  ```rust
  fn total(amounts: &[u64]) -> Option<u64> {
      amounts.iter().try_fold(0_u64, |acc, &x| acc.checked_add(x))
  }

  fn invoice_total(lines: &[Line]) -> Result<u64, ArithmeticError> {
      lines.iter().try_fold(0_u64, |acc, line| {
          let line_total = line
              .unit_price
              .checked_mul(u64::from(line.quantity))
              .ok_or(ArithmeticError::Overflow { figure: "line_total", operation: Operation::Mul })?;
          acc.checked_add(line_total)
              .ok_or(ArithmeticError::Overflow { figure: "invoice_total", operation: Operation::Add })
      })
  }
  ```

- **RCA-AGG-2.** An average MUST check both the sum and the division. The count MUST be nonzero: an average of no
  values is unknown, not zero, unless the declaration names a fallback for it (RCA-DECL-4). The midpoint of two values MUST NOT be written as `(a + b) / 2`. Use the `midpoint`
  method where the MSRV supports it for the type, or `a.checked_add(b)?.checked_div(2)`. The declaration MUST state
  the rounding.
- **RCA-AGG-3.** A running total that is kept across calls (a ledger balance, a cumulative dose, a counter in storage)
  MUST be updated with a checked operation. The stored value MUST NOT change when the update fails.
- **RCA-AGG-4.** A signed total that must fit only at the end (credits and debits) SHOULD be added up in `i128`, then
  converted with `try_from`, so term order cannot cause an intermediate overflow.

## Wide intermediates, ratios and percentages

- **RCA-WIDE-1.** A product, ratio or percentage SHOULD be computed in a type wide enough that the intermediate result
  cannot overflow. The result MUST then be converted back with `try_from`. The product of two `i64` values always fits
  `i128`, and the product of two `u64` values always fits `u128`.
- **RCA-WIDE-2.** Rates and percentages SHOULD be integers in a stated fixed unit, such as basis points (1/10,000),
  parts per million, or milli-units. Floats SHOULD NOT be used for them.

  ```rust
  /// Returns `amount * rate_bps / 10_000`, rounded half up. Declared in spec FEE-3.
  fn apply_rate(amount: u64, rate_bps: u32) -> Result<u64, ArithmeticError> {
      const BPS_PER_UNIT: u128 = 10_000;
      const HALF: u128 = BPS_PER_UNIT / 2;
      let overflow = |operation| ArithmeticError::Overflow { figure: "fee", operation };
      let product = u128::from(amount)
          .checked_mul(u128::from(rate_bps))
          .ok_or(overflow(Operation::Mul))?;
      let rounded = product
          .checked_add(HALF)
          .ok_or(overflow(Operation::Add))?
          .checked_div(BPS_PER_UNIT)
          .ok_or(ArithmeticError::DivisionByZero { figure: "fee" })?;
      u64::try_from(rounded).map_err(|_| ArithmeticError::OutOfRange { figure: "fee" })
  }
  ```

- **RCA-WIDE-3.** Widening does not replace checking. Code MUST still use checked operations in the wide type, unless
  the bound is proven in a comment and the code follows [Permitted exceptions](#permitted-exceptions).

## Domain types

- **RCA-DOM-1.** Each external figure with a unit or a range SHOULD have a newtype (`Cents`, `Milligrams`,
  `BasisPoints`, `PageNumber`) rather than a bare integer, so milligrams cannot be added to micrograms.
- **RCA-DOM-2.** A newtype's constructor MUST validate the declared range and return `Result` or `Option`. The inner
  field MUST be private, so no value outside the range can exist.
- **RCA-DOM-3.** Newtypes MUST NOT implement `std::ops::Add`, `Sub`, `Mul`, `Div`, `Rem` or `Neg` with a non-fallible
  `Output`. They SHOULD provide named fallible methods instead (`checked_add`, `checked_sub`, `checked_scale`) that
  return `Option<Self>` or `Result<Self, E>`.
- **RCA-DOM-4.** Newtypes SHOULD NOT implement `Default` when zero is not a meaningful default. They MUST NOT implement
  `Default` as a placeholder for "unknown".
- **RCA-DOM-5.** Operations across units MUST be explicit, typed functions (`Milligrams::from_micrograms`,
  `Dose::per_kilogram(weight)`). Converting between units MUST be checked, like any other arithmetic.

  ```rust
  /// A dose in micrograms. Invariant: 0 ..= MAX_DOSE_MCG (spec DOSE-1).
  #[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
  pub struct Micrograms(u64);

  impl Micrograms {
      pub const MAX_DOSE_MCG: u64 = 10_000_000;

      pub fn new(mcg: u64) -> Result<Self, ArithmeticError> {
          if mcg <= Self::MAX_DOSE_MCG {
              Ok(Self(mcg))
          } else {
              Err(ArithmeticError::Domain { figure: "dose", reason: "above the maximum dose" })
          }
      }

      pub fn get(self) -> u64 {
          self.0
      }

      pub fn checked_add(self, other: Self) -> Result<Self, ArithmeticError> {
          let sum = self
              .0
              .checked_add(other.0)
              .ok_or(ArithmeticError::Overflow { figure: "dose", operation: Operation::Add })?;
          Self::new(sum) // re-validates the domain range, not only the type range
      }
  }
  ```

## Money and decimals

- **RCA-MON-1.** Money MUST NOT be represented as `f32` or `f64`. Represent it as one of these:
  - an integer count of the currency's minor unit (`i64` cents) together with the currency, or
  - a fixed-point decimal type (for example `rust_decimal::Decimal`) together with the currency and a declared scale.
- **RCA-MON-2.** A money value MUST carry its currency. Arithmetic between different currencies MUST return an error.
  It MUST NOT convert implicitly.

  ```rust
  #[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
  pub enum Currency { Eur, Gbp, Jpy, Usd }

  #[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
  pub struct Money { minor: i64, currency: Currency }

  impl Money {
      pub fn checked_add(self, other: Self) -> Result<Self, MoneyError> {
          if self.currency != other.currency {
              return Err(MoneyError::CurrencyMismatch { left: self.currency, right: other.currency });
          }
          let minor = self.minor.checked_add(other.minor).ok_or(MoneyError::Overflow)?;
          Ok(Self { minor, currency: self.currency })
      }
  }
  ```

- **RCA-MON-3.** The number of minor units per major unit depends on the currency: 100 for EUR and USD, 1 for JPY, 1000
  for KWD and others. It MUST come from a currency table, not a hard-coded `100`.
- **RCA-MON-4.** Decimal types are not automatically safe. The `+`, `-`, `*` and `/` operators of
  `rust_decimal::Decimal` **panic** on overflow and on division by zero. They can also round silently when a result
  needs more than 28 decimal places. Code MUST use the decimal type's `checked_add`, `checked_sub`, `checked_mul`,
  `checked_div` and `checked_rem`. It MUST round explicitly with a named strategy (for example
  `round_dp_with_strategy`) at the point the declaration states.
- **RCA-MON-5.** Code MUST NOT construct a decimal from a float (`Decimal::from_f64`, `from_f64_retain`, `try_from(f64)`)
  for external figures. Construct it from a string, or from an integer and a scale.
- **RCA-MON-6.** Splitting an amount (instalments, cost sharing, pro-rata refunds) MUST conserve the total: the parts
  MUST sum exactly to the whole. The leftover minor units MUST be distributed by a declared, deterministic rule.

  ```rust
  use std::num::NonZero;

  /// Splits `total` into `parts` shares. The shares differ by at most one minor unit,
  /// the larger shares come first, and they sum exactly to `total` (spec PAY-7).
  fn split_evenly(total: u64, parts: NonZero<u64>) -> Result<Vec<u64>, ArithmeticError> {
      let base = total / parts;      // u64 / NonZero<u64> cannot panic
      let remainder = total % parts; // remainder < parts
      let len = usize::try_from(parts.get()).map_err(|_| ArithmeticError::OutOfRange { figure: "parts" })?;
      let mut shares = Vec::new();
      shares
          .try_reserve_exact(len)
          .map_err(|_| ArithmeticError::OutOfRange { figure: "parts" })?;
      for i in 0..parts.get() {
          let extra = u64::from(i < remainder);
          let share = base
              .checked_add(extra)
              .ok_or(ArithmeticError::Overflow { figure: "share", operation: Operation::Add })?;
          shares.push(share);
      }
      Ok(shares)
  }
  ```

- **RCA-MON-7.** Currency conversion MUST use a rate with a stated precision, source and timestamp, round once by the
  declared strategy, and SHOULD record the rate.

## Floating point

- **RCA-FLT-1.** Floats (`f32`, `f64`) MUST NOT be used for money, counts, identifiers, legal or regulatory quantities,
  or any figure whose declaration requires exact results.
- **RCA-FLT-2.** Where floats are allowed (measurements, statistics, physical and clinical quantities with a declared
  tolerance), every result that crosses a module boundary, gets stored or gets shown MUST be checked with `is_finite()`.
  Float operations do not panic. They produce `inf`, `-inf` or `NaN`, which then spread through every later operation.
  Division by zero yields `inf` or `NaN`, not an error.

  ```rust
  fn ratio(numerator: f64, denominator: f64) -> Result<f64, ArithmeticError> {
      let r = numerator / denominator;
      if r.is_finite() { Ok(r) } else { Err(ArithmeticError::DivisionByZero { figure: "ratio" }) }
  }
  ```

- **RCA-FLT-3.** Floats MUST NOT be compared with `==` or `!=` for computed results. Compare within a declared
  tolerance. Sorting and ordering MUST use `f64::total_cmp`, or must reject `NaN` first, because `NaN` breaks
  `PartialOrd`.
- **RCA-FLT-4.** `NaN` MUST NOT mark "unknown". Use `Option<f64>`.
- **RCA-FLT-5.** Converting a float to an integer follows [RCA-CONV-3](#conversions).

## Time, dates and durations

- **RCA-TIME-1.** Arithmetic on `std::time::Duration`, `Instant` and `SystemTime` MUST use the checked methods:
  `Duration::checked_add`, `checked_sub`, `checked_mul`, `checked_div`; `Instant::checked_add`, `checked_sub`,
  `checked_duration_since`; `SystemTime::checked_add`, `checked_sub`, and `duration_since` (which returns `Result`).
  The `+` and `-` operators on these types panic on overflow. `Instant - Instant`, `Instant::duration_since`,
  `Instant::saturating_duration_since` and `Duration::saturating_*` saturate to zero, so they MUST NOT be used.
- **RCA-TIME-2.** Durations built from floats MUST use `Duration::try_from_secs_f64` or `try_from_secs_f32`.
  `Duration::from_secs_f64` and `from_secs_f32` panic on negative, non-finite or overflowing input.
- **RCA-TIME-3.** Date and time crates follow the same rule. In `chrono`, use `checked_add_signed`, `checked_sub_signed`,
  `checked_add_days`, `checked_add_months` and `TimeDelta::checked_*`. In `time`, use `checked_add` and `checked_sub`.
  In `jiff`, use `checked_add` and `checked_sub`, which return `Result`. Their operator forms panic.
- **RCA-TIME-4.** Calendar arithmetic is not integer arithmetic. "One month after 31 January", leap days, leap
  seconds, time-zone transitions and business days MUST follow a rule the declaration states. They MUST be computed with
  a date library, never by adding a fixed number of seconds.
- **RCA-TIME-5.** A wall-clock interval (`SystemTime`) can be negative because clocks are adjusted. Measuring elapsed
  time MUST use `Instant`. A negative wall-clock interval MUST be handled as an error, an unknown, or a declared fallback
  (`checked_*` plus `unwrap_or`), never silently clamped to zero.

## Lengths, indexes and allocation

- **RCA-LEN-1.** `len() - 1`, `i - 1` and `end - start` MUST be checked. On an empty collection, or when `start > end`,
  they underflow. Use `checked_sub`, `last()`, `split_last()` or `iter().rev()`, never `saturating_sub`. Where the
  declaration names a fallback, write `checked_sub(1).unwrap_or(FALLBACK)`.
- **RCA-LEN-2.** Indexing (`v[i]`) and slicing (`&s[a..b]`) with external indexes MUST NOT be used. Use `get`,
  `get_mut`, `split_at_checked`, `chunks` and iterators, and handle the `None`. String slicing also panics when an index
  is not on a UTF-8 character boundary.
- **RCA-LEN-3.** Computed offsets and sizes (`offset + len`, `count * size_of::<T>()`, `width * height * channels`) MUST
  use checked operations before they are used to index, slice, seek or allocate.
- **RCA-LEN-4.** An allocation sized by an external figure MUST be bounded by a declared maximum, and SHOULD use
  `try_reserve` or `try_reserve_exact`. `Vec::with_capacity` panics on capacity overflow.

## Permitted exceptions

A plain operator or `as` conversion is permitted only in the cases below, visibly at the place it is used. Wrapping
and saturation never are.

- **RCA-EXC-1. Constant expressions.** Arithmetic in `const` and `static` items is evaluated at compile time, and the
  compiler rejects overflow. Plain operators MAY be used there.
- **RCA-EXC-2. Proven bounds.** A plain operator MAY be used when the operands are bounded by values the code itself
  controls. An example is the index of a loop over a fixed-size array. The proof MUST be written next to the operation,
  and the lint MUST be expected, not allowed:

  ```rust
  const SLOTS: usize = 16;
  let mut ring = [0_u32; SLOTS];
  for i in 0..SLOTS {
      #[expect(clippy::arithmetic_side_effects, reason = "RCA-EXC-2: i < SLOTS = 16, so i + 1 <= 16")]
      let next = (i + 1) % SLOTS;
      // ...
  }
  ```

  A bound that depends on an external figure is not a proof.
- **RCA-EXC-3. No wrapping.** Wrapping has no exception. Hashes, checksums, CRCs and random generators MUST come from
  vetted dependencies (RCA-ENF-6), not workspace code. A sequence number that rolls over MUST be compared and reset
  explicitly with checked operations.
- **RCA-EXC-4. No saturation.** `saturating_*`, `Saturating<T>` and `clamp` on external figures have no exception,
  not even for display values, back-off ceilings or counters. Where a bound is wanted, declare it as a fallback and
  write `checked_*` plus `unwrap_or(BOUND)` (RCA-NONE-2). Or compare the checked result with the bound and refuse or
  report. Either way the bound is visible, cited and tested.
- **RCA-EXC-5. Tests.** Test code MAY use plain operators, `unwrap` and `expect`.
- **RCA-EXC-6. Recording.** Each exception in production code MUST use `#[expect(lint, reason = "RCA-EXC-n: ...")]`
  rather than `#[allow]`, so that the exception is reported when it is no longer needed. Exceptions MUST NOT be applied
  to a whole module or crate.

## Enforcement

- **RCA-ENF-1. Lints.** The workspace MUST configure Clippy lints in `Cargo.toml` and run Clippy in CI with
  `cargo clippy --workspace --all-targets --all-features -- -D warnings`.

  ```toml
  # Workspace Cargo.toml. Each member adds: [lints] workspace = true
  [workspace.lints.clippy]
  # Arithmetic (RCA-INT, RCA-AGG, RCA-WIDE)
  arithmetic_side_effects = "deny"
  integer_division = "deny"
  modulo_arithmetic = "deny"
  default_numeric_fallback = "deny"
  # Conversions (RCA-CONV)
  as_conversions = "deny"
  cast_lossless = "deny"
  cast_possible_truncation = "deny"
  cast_possible_wrap = "deny"
  cast_precision_loss = "deny"
  cast_sign_loss = "deny"
  checked_conversions = "deny"
  # Floats (RCA-FLT); in crates allowed to use floats, set float_arithmetic per crate instead
  float_arithmetic = "deny"
  float_cmp = "deny"
  lossy_float_literal = "deny"
  # Panics (RCA-P-1, RCA-NONE, RCA-LEN)
  unwrap_used = "deny"
  expect_used = "deny"
  panic = "deny"
  indexing_slicing = "deny"
  string_slice = "deny"
  unreachable = "deny"
  todo = "deny"
  unimplemented = "deny"
  # Exceptions must be expected, with a reason (RCA-EXC-6)
  allow_attributes = "deny"
  allow_attributes_without_reason = "deny"
  ```

- **RCA-ENF-2. Disallowed methods and types.** The workspace MUST ban what bypasses checking, in `clippy.toml`:

  ```toml
  allow-unwrap-in-tests = true
  allow-expect-in-tests = true
  allow-panic-in-tests = true
  allow-indexing-slicing-in-tests = true

  disallowed-methods = [
    { path = "core::iter::Iterator::sum", reason = "RCA-AGG-1: use try_fold with checked_add" },
    { path = "core::iter::Iterator::product", reason = "RCA-AGG-1: use try_fold with checked_mul" },
    { path = "core::time::Duration::from_secs_f64", reason = "RCA-TIME-2: use try_from_secs_f64" },
    { path = "core::time::Duration::from_secs_f32", reason = "RCA-TIME-2: use try_from_secs_f32" },
    { path = "core::option::Option::unwrap_or_default", reason = "RCA-P-3: an unknown is never a zero" },
    { path = "std::time::Instant::duration_since", reason = "RCA-TIME-1: use checked_duration_since" },
    { path = "std::time::Instant::saturating_duration_since", reason = "RCA-INT-4: no saturation" },
  ]
  disallowed-types = [
    { path = "core::num::Saturating", reason = "RCA-INT-4: no saturation" },
    { path = "core::num::Wrapping", reason = "RCA-INT-4: no wrapping" },
  ]
  ```

  Add each project-specific decimal or date crate's panicking constructors to the list (for example
  `rust_decimal::Decimal::from_f64`).
- **RCA-ENF-3. Adopting in existing code.** A crate that cannot yet deny `arithmetic_side_effects` MUST enable it one
  module at a time as each is converted. New modules MUST deny it from the start; the crate root, once all are done.
- **RCA-ENF-4. Release overflow checks.** The release profile MUST turn on overflow checks, so that any operator the
  lints miss (in macros, in dependencies or behind an exception) panics, which is detectable, instead of wrapping,
  which is not:

  ```toml
  [profile.release]
  overflow-checks = true
  ```

  This is a backstop only; RCA-P-1 still forbids the panic.
- **RCA-ENF-5. Review grep.** CI MUST fail on `rg -w --type rust '(saturating|wrapping|overflowing)_\w+'` in
  production code (RCA-INT-4, RCA-INT-5). CI SHOULD also fail on unwrapped checked results, which no lint catches:
  `rg -U --type rust 'checked_\w+\([^;]*?\)\s*\.(unwrap|expect|unwrap_unchecked|unwrap_or_default)\b'`. Review SHOULD
  confirm that each fallback cites its declaration (RCA-NONE-3). This search lists them:
  `rg -U --type rust 'checked_\w+\([^;]*?\)\s*\.(unwrap_or|unwrap_or_else|map_or|map_or_else)\b'`.
- **RCA-ENF-6. Dependencies.** A dependency that does arithmetic on external figures (money, decimal, date, units,
  parsing) MUST be audited for panicking operators before adoption, and wrapped behind checked functions if needed.

## Testing and verification

- **RCA-TEST-1. Edge values.** Every function that does checked arithmetic on external figures MUST have tests at the
  edges:

  | Case | Values |
  |---|---|
  | Type limits | `T::MIN`, `T::MIN + 1`, `-1`, `0`, `1`, `T::MAX - 1`, `T::MAX` |
  | First overflow | The smallest input for which the result no longer fits |
  | Last success | The largest input for which the result still fits |
  | Divisors | `0`, `1`, `-1`, and `T::MIN / -1` for signed types |
  | Domain limits | The declared minimum and maximum, and one past each |
  | Empty input | Empty collections for sums, averages, splits and `len() - 1` |
  | Floats | `NaN`, `inf`, `-inf`, `-0.0`, subnormals, 2^53, 2^53 + 1, 2^63 |
  | Currency | Mismatched currencies, zero-decimal and three-decimal currencies |

- **RCA-TEST-2. Error paths.** Each error variant a function can return MUST be produced by at least one test. The test
  MUST assert the exact variant and figure name, not just `is_err()`. Each declared fallback MUST be produced by at
  least one test at the first failing input, and the test MUST assert the exact fallback value.
- **RCA-TEST-3. Property tests.** Core arithmetic SHOULD have property-based tests (for example with `proptest`) that
  compare the checked result against an oracle computed in a wider type (`i128`, `u128`, or an arbitrary-precision
  integer). The test asserts `Some` exactly when the oracle's result fits, and that the values are equal.

  ```rust
  proptest! {
      #[test]
      fn line_total_matches_oracle(price in any::<u64>(), qty in any::<u32>()) {
          let oracle = u128::from(price) * u128::from(qty);
          match line_total(price, qty) {
              Ok(v) => prop_assert_eq!(u128::from(v), oracle),
              Err(_) => prop_assert!(oracle > u128::from(u64::MAX)),
          }
      }
  }
  ```

- **RCA-TEST-4. Conservation properties.** Splitting, allocation, rounding and currency code MUST have property tests
  for its invariants. For example: the parts sum to the whole, every part is within one minor unit of the others, and
  rounding is idempotent.
- **RCA-TEST-5. Proofs.** For figures whose failure has safety, legal or large financial consequences, the core
  functions SHOULD be proved panic-free and overflow-free with a model checker such as Kani (`#[kani::proof]` with
  `kani::any()` inputs). The proof harness MUST run in CI.
- **RCA-TEST-6. Fuzzing.** Parsers and calculators that take untrusted input SHOULD be fuzzed (for example with
  `cargo fuzz`) with overflow checks on. Any panic found MUST become a regression test.
- **RCA-TEST-7. Both profiles.** Tests SHOULD run in debug and release profiles.

## Review checklist

A reviewer or agent verifying a change against this file confirms each item:

1. Every external figure has a declaration (RCA-DECL-1): unit, type, range, unknown, rounding, overflow and fallback.
2. No plain operator or panicking method on an external figure outside a recorded exception (RCA-INT-2, RCA-EXC),
   and no wrapping, saturating, overflowing or clamping anywhere, including `max(lo).min(hi)` (RCA-INT-4 to
   RCA-INT-6). Every `min` or `max` that bounds an external figure cites its declaration (RCA-INT-6).
3. No checked result is unwrapped, defaulted or replaced by an undeclared value. Every `unwrap_or` fallback is
   declared, in range, and cites its declaration (RCA-NONE-1, RCA-NONE-3).
4. Every `None` is handled close to its source: it is replaced by its declared fallback, becomes a named error,
   propagates, or becomes a user-visible unknown (RCA-NONE-2 to RCA-NONE-5).
5. No numeric `as` conversion; `From` or `TryFrom` is used instead (RCA-CONV-1, RCA-CONV-2).
6. Every division handles zero, and signed division handles `MIN / -1` (RCA-DIV-1).
7. Rounding is named, declared and applied once (RCA-DIV-5, RCA-MON-4).
8. No `sum` or `product` on external figures (RCA-AGG-1).
9. Money carries its currency, is never a float, and conserves totals when split (RCA-MON).
10. Floats are checked for finiteness where allowed, and are not used where forbidden (RCA-FLT).
11. Time arithmetic uses checked methods and `try_from_secs_*` (RCA-TIME).
12. No external index is used for indexing or slicing, and allocations are bounded (RCA-LEN).
13. Each exception uses `#[expect(..., reason = "RCA-EXC-n: ...")]` with a proof or a citation (RCA-EXC-6).
14. Edge-value and error-path tests exist for each new arithmetic function (RCA-TEST-1, RCA-TEST-2).

## Quick reference

| Instead of | Use |
|---|---|
| `a + b`, `a - b`, `a * b` | `a.checked_add(b)`, `a.checked_sub(b)`, `a.checked_mul(b)` |
| `a / b`, `a % b` | `a.checked_div(b)`, `a.checked_rem(b)`, or a `NonZero` divisor for unsigned types |
| `-a`, `a.abs()` | `a.checked_neg()`, `a.checked_abs()`, or `a.unsigned_abs()` |
| `a.pow(n)` | `a.checked_pow(n)` |
| `x << n` to multiply | `x.checked_mul(1 << n)` with a constant `n`, or `x.checked_mul(2_u64.checked_pow(n)?)` |
| `(a + b) / 2` | `a.midpoint(b)`, or checked add then checked divide |
| `.checked_add(b).unwrap()` | `.checked_add(b).unwrap_or(FALLBACK)` when declared, else `.ok_or(ArithmeticError::Overflow { .. })?` |
| `.unwrap_or_default()`, or `.unwrap_or(0)` with no declaration | Declare the fallback and cite it, or return an error, or propagate `None` |
| `a.saturating_sub(b)` where the spec names a floor | `a.checked_sub(b).unwrap_or(FLOOR)`, citing the declaration |
| `iter.sum()` | `iter.try_fold(0, \|acc, x\| acc.checked_add(x))` |
| `x as u32` | `u32::try_from(x)` or `u32::from(x)` |
| `f as i64` | A checked helper as in RCA-CONV-3 |
| `f64` money | `i64` minor units with a currency, or a checked decimal |
| `decimal_a + decimal_b` | `decimal_a.checked_add(decimal_b)` |
| `instant + duration` | `instant.checked_add(duration)` |
| `Duration::from_secs_f64(s)` | `Duration::try_from_secs_f64(s)` |
| `v[i]`, `&s[a..b]` | `v.get(i)`, `s.get(a..b)` |
| `v.len() - 1` | `v.len().checked_sub(1)` or `v.last()` |
| `Vec::with_capacity(n)` with external `n` | Bound `n`, then `try_reserve_exact(n)` |
| `wrapping_*`, `saturating_*`, `overflowing_*`, `clamp`, `max(lo).min(hi)` | `checked_*`, then recover with a declared fallback, refuse, or report |
| `#[allow(clippy::arithmetic_side_effects)]` | `#[expect(clippy::arithmetic_side_effects, reason = "RCA-EXC-n: ...")]` on one statement |
